# MCP 生态安全

> 2025-2026 增长最快的攻击面。核心思路：**把工具元数据当代码管理，把 MCP 服务器当供应链治理。**

## 1. 工具描述指纹 + 漂移检测（核心防御，代码示例）

```python
# 工具定义指纹：首次连接哈希，变更即阻断待审
import hashlib, json, sqlite3

def tool_fingerprint(tool_def: dict) -> str:
    # 只对元数据签名（名称+描述+schema），不含代码
    blob = json.dumps(tool_def, sort_keys=True, ensure_ascii=False)
    return hashlib.sha256(blob.encode()).hexdigest()

STATES = ["PENDING", "DRIFT_DETECTED", "TRUSTED", "BLOCKED", "REMOVED"]

def register_or_verify(conn, server, tool_def):
    fp = tool_fingerprint(tool_def)
    row = conn.execute(
        "SELECT fingerprint, state FROM tools WHERE server=? AND name=?",
        (server, tool_def["name"])).fetchone()
    if row is None:
        # 新工具 → PENDING（人工审查后才能被调用）
        conn.execute("INSERT INTO tools VALUES (?,?,?,?)",
                     (server, tool_def["name"], fp, "PENDING"))
        return "PENDING"          # 未审工具不可调用
    if row[0] != fp:
        # 元数据变更 → DRIFT_DETECTED（阻断 + 告警 + 需重新审批）
        conn.execute("UPDATE tools SET fingerprint=?, state=? WHERE server=? AND name=?",
                     (fp, "DRIFT_DETECTED", server, tool_def["name"]))
        alert(f"MCP tool metadata drift: {server}/{tool_def['name']}")
        return "DRIFT_DETECTED"
    return row[1]
```

## 2. 描述指令性语言扫描（正则 + 语义）

```python
# 正则快速筛查（低误报设计：要求动作指令特征）
DESCRIPTION_RED_FLAGS = [
    r"\b(always|must|require(?:d)?|important|urgent)\b.{0,80}\b(attach|send|output|include|copy|retrieve)\b",
    r"\b(ignore|override|bypass|disregard)\b",           # 控制意图
    r"\b(no|without|don'?t)\s+need\s+(user\s+)?(confirmation|approval)\b",  # 授权伪造
    r"\b(already\s+)?approved\b.{0,40}\b(by\s+(admin|management|security|legal))\b",
    r"\b(api[_-]?key|secret|token|password|credential)\b",  # 敏感词（描述中不该出现）
    r"\b(attacker|evil\.com|c2|exfiltrat)\b",             # 恶意端点
]
# 语义层：分类器（SHIELDMCP 方法）区分"能力说明"与"动作指令"
# 阈值 τ≈0.72，超限隔离待人工审查

# 隐藏字符检查
import re
def has_hidden_chars(s: str) -> bool:
    return bool(re.search(r"[\u200b-\u200f\u202a-\u202e\u2060-\u206f\ufeff]", s))
```

## 3. 运行时拦截（网关/代理模式）

```
拦截点 1（注册时）：描述完整性 + 指纹入库
拦截点 2（调用前）：出站参数 schema 校验 + 字符串注入扫描（SQL/命令/注入载荷）
拦截点 3（响应后）：指令性 token 标记 + 上下文边界包裹 + 跨调用相关性

// 响应分析示例：检测"工具输出中出现指令性语言"
def classify_tokens(text):
    # 分类器输出 instructional vs informational token
    # instructional 占比超阈值 → 标记/阻断（Das et al. 方法）
    return token_classifier.predict(text)

// 跨调用相关性：会话依赖图
// 若某响应触发了"未被任务计划预测"的级联调用 → 高危告警
```

实测数据（SHIELDMCP）：工具投毒 74%→<9%，间接注入 47%→<6%，延迟 +<120ms。

## 4. 服务器治理清单（可执行）

```bash
# 盘点已安装的 MCP 服务器配置
cat ~/.config/claude/claude_desktop_config.json   # Claude Desktop
cat .mcp.json                                    # 项目级配置
grep -r "mcpServers" ~/.config/*/config.json 2>/dev/null

# 审查项：
#  □ 发布者是否在白名单（租户级 allowlist）
#  □ 服务器版本是否锁定（禁止 auto-update）
#  □ 工具是否最小集（关闭不用的）
#  □ 工具描述与已批准基线 diff
#  □ 出站白名单（Agent 运行时不可任意出网）
```

## 5. 客户端选择与要求

| 客户端 | 2026 实证攻击成功率（工具投毒） | 说明 |
|---|---|---|
| Claude Desktop | 0% | 静态校验 + 参数可见性较好 |
| Cline | 低 | 推荐安全敏感场景 |
| Cursor | 100% | 严重缺乏静态校验（需补偿控制） |
| 其他多数 | 部分成功 | 需审计静态校验与参数可见性 |

要求清单：
```
□ 安装时展示工具描述，变更时再次展示
□ 用户可见传给工具的完整参数
□ 元数据变更触发重新审批（防"静默重新信任"）
□ 支持签名验证（有签名必验）
```

## 6. 审计存量部署（30 分钟行动）

```
1. 盘点所有 MCP 服务器（含废弃/实验）
2. 导出所有工具描述，与已知良好版本 diff
3. 移除未使用工具；锁定版本
4. 阻断 Agent 运行时任意出站 HTTP
5. 读/写工具分离；跨信任域数据移动审批
6. 记录 prompts、元数据、工具调用（全参数）、输出
7. 为使用过不可信/近期变更工具的 Agent 轮换凭证
8. 部署指纹 + 漂移阻断（上文脚本）
```
