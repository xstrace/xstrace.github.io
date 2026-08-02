# Agent 架构加固

> 核心：**最小自主（Least Agency）**——比最小权限更进一步：即使权限最小的 Agent，若可无检查行动也能造成危害。

## 1. Agent 权限策略文件示例（YAML）

```yaml
# agent-policies.yaml
agents:
  invoice_agent:
    identity: "entra-id:invoice-agent@corp"     # 每 Agent 独立身份（NHI）
    credentials: short_lived_tokens(ttl=15min)   # 短时凭证
    allowed_tools:
      - read: [invoice_db.query, outlook.read]   # 读工具
      - write: []                                 # 无写权限
    requires_approval:                           # HITL 清单
      - any_outbound_http
      - send_email
      - delete_*
      - external_share
    egress: deny_by_default                      # 默认拒绝出网
    rate_limit: { calls_per_min: 30 }
    sandbox: { type: docker, net: none }         # 沙箱 + 无网络
    max_steps: 20                                # 会话步骤上限
    memory: { persistent: false }                # 临时上下文

  coding_agent:
    identity: "entra-id:code-agent@corp"
    allowed_tools:
      - read: [repo.read, search]
      - write: [repo.commit_with_review]
    requires_approval:
      - exec_shell
      - apply_patch_to_prod
    sandbox: { type: container, net: allowlist[registry.corp.com] }
```

## 2. HITL 审批实现要点

```python
# 强制交互式确认（不可被 Agent API 绕过）
def approve(action: dict) -> bool:
    ticket = create_approval_ticket(action)   # 每次动作独立票据
    # ❌ 错误做法：一次审批授权整个会话
    # ✅ 正确做法：每动作独立 + 强制用户交互（扫码/双因素）
    return user_confirms_interactively(ticket.id, action.summary)

# 分级审批
LEVELS = {
    "low":    lambda a: auto_allow(a),                 # 只读/无害
    "medium": lambda a: single_approval(a),            # 一人确认
    "high":   lambda a: dual_approval(a),              # 双人确认
    "critical": lambda a: manual_execution(a),         # 人工亲自执行
}
```

## 3. 身份与授权外置（Complete Mediation）

```python
# ❌ 错误：让模型判断权限（模型可被注入说服）
# ✅ 正确：确定性代码根据认证用户判定
def can_execute(user: User, agent: Agent, tool: str, args: dict) -> bool:
    if not authenticate(user):                      # 认证
        return False
    if tool not in agent.allowlist:                 # Agent 范围
        return False
    if not user_has_permission(user, tool, args):   # 用户权限（RBAC）
        return False
    return True
# 模型只能"请求"，决定权在代码
```

## 4. 沙箱执行示例

```bash
# Docker 沙箱：无网络 + 只读挂载
docker run --rm \
  --network none \
  -v /workspace:/work:ro \
  -v /tmp/agent-cache:/cache \
  --memory 512m --cpus 1 \
  --read-only \
  agent-runtime python run_task.py

# 编码 Agent 代码执行沙箱
# 禁止直接执行 LLM 生成的 shell 命令；强制走代码审查 + 沙箱 CI
```

## 5. 工具调用治理配置

```python
# 参数强校验（类型+范围+枚举），防参数注入
SCHEMAS = {
    "send_email": {
        "recipient": {"type": "email", "allowlist": ["@corp.com"]},
        "body":      {"type": "str", "max_len": 5000},
        "attachments": {"type": "list", "max_items": 5,
                        "allowed_ext": [".pdf", ".csv"]},
    }
}

def validate_params(tool, args):
    schema = SCHEMAS.get(tool)
    if not schema: return False          # 未登记工具 → 拒绝
    return jsonschema.validate(args, schema)  # 严格校验

# 读/写分离：读工具与写工具分开注册、分开授权
# 跨信任域移动（Drive→Slack、CRM→邮件、密钥库→任何）→ 强制审批
```

## 6. 部署前检查（Microsoft 2026 提炼）

```
□ 治理工具供应链：发布者/服务器租户级白名单，关闭 Allow-all
□ 工具描述当系统提示管理：变更审查 + 指令性语言扫描
□ 高影响动作人工审批（执行前）
□ Agent 独立身份 + 条件访问（Entra Agent ID 等）
□ 行为基线 + 日志关联（新端点/扩参数/异常查询告警）
□ 部署前红队演练（promptfoo/PyRIT 跑目标劫持与工具滥用场景）
```

## 7. 参考链接

| 内容 | 链接 |
|---|---|
| 微软 Agent 安全策略（2026 指南） | microsoft.com/security/blog（agentic AI 安全系列） |
| OWASP Agentic Top 10（最小自主 ASI05 等） | genai.owasp.org/agentic-top-10 |
| MITRE ATLAS（Agent 威胁矩阵） | atlas.mitre.org |
| 最小自主原则（Least Agency） | 微软安全博客（agentic security） |
| MAESTRO 威胁建模（CSA） | cloudsecurityalliance.org |
