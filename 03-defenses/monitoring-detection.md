# 监控与检测

> 注入无法 100% 阻止 → 检测必须在分钟级发现。以下为可直接落地的规则与命令。

## 1. 行为基线 + 偏离告警

```yaml
# agent-behavior-baseline.yaml（每 Agent 一份）
baseline:
  agent: invoice_agent
  learned_from: 30d_traffic
  tools:
    invoice_db.query:  { avg_per_day: 120, max_per_day: 400 }
    outlook.read:      { avg_per_day: 60,  max_per_day: 200 }
  outbound_endpoints: [api.corp.com, gate.corp.com]
  param_patterns:
    invoice_db.query:  "SELECT ... WHERE id = <int>"   # 正常参数形态
  max_data_rows_per_call: 1000
alerts:
  new_outbound_endpoint:          { severity: high }
  tool_call_volume_spike_x5:      { severity: high }
  param_shape_change:             { severity: medium }  # SQL 突然全表
  cross_tool_unexpected_chain:    { severity: high }    # 未预测的级联
  approval_mismatch:              { severity: high }    # 执行≠审批内容
```

```python
# 跨调用相关性检测（会话依赖图简化版）
def check_unexpected_chain(session, next_call):
    expected = predict_next_calls(session.task_plan)   # 模型任务计划预测
    if next_call not in expected:
        alert("Unexpected tool call chain",
              f"{session.id}: {next_call} not in plan")
        return "block"          # 高置信拦截
    return "log"
```

## 2. 外泄检测规则（可直接部署）

```python
# 输出/工具参数侧检测
import re, base64

SECRET_PATTERNS = [
    r"sk-[A-Za-z0-9]{20,}",                      # OpenAI key
    r"AIza[0-9A-Za-z\-_]{35}",                   # Google key
    r"AKIA[0-9A-Z]{16}",                         # AWS access key
    r"-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY", # 私钥
    r"ghp_[0-9A-Za-z]{36}",                      # GitHub PAT
    r"xox[baprs]-[0-9A-Za-z-]{10,}",             # Slack token
    r"\b\d{16}\b",                               # 信用卡
]

def is_base64_blob(s, min_len=80):               # 加密外泄（高熵检测）
    try:
        d = base64.b64decode(s, validate=True)
        return len(d) >= min_len
    except Exception:
        return False

def scan_exfil(text):
    hits = [p for p in SECRET_PATTERNS if re.search(p, text)]
    entropy_hits = [t for t in split_tokens(text) if is_base64_blob(t)]
    return {"secrets": hits, "encoded_blobs": len(entropy_hits)}
```

## 3. 检测四层速查

| 层 | 检测内容 | 工具/方法 |
|---|---|---|
| 输入层 | 越狱/注入 | Prompt Guard 2、LLM Guard、Lakera |
| 行为层 | 工具调用异常 | 基线偏离、依赖图（上文） |
| 数据层 | 外泄模式 | 密钥正则、高熵、DLP |
| 事后层 | 会话取证 | 全量日志（prompts/元数据/调用/输出） |

## 4. 日志与取证要求

```python
# 关键：记录"调用发生时可见的工具元数据版本"（不是当前版本！）
log_entry = {
    "ts": now(),
    "agent": agent_id,
    "user": user_id,
    "task": task_hash,
    "tool_call": {"tool": name, "args": full_args},
    "tool_metadata_fp": fingerprint_at_call_time,   # ← 溯源关键
    "retrieved_docs": [doc_ids],
    "output_hash": sha256(response),
    "decision_path": trace_id,
}
# 日志脱敏在写入时执行（PII/密钥），非读取时
```

## 5. 告警运营建议

```
□ 精确类（密钥/越狱字符串/新端点）→ block 或立即告警
□ 模糊类（语义违规/参数形态变化）→ log-and-alert（误报代价权衡）
□ 阈值按基线而非固定值（正常流量无特征，靠偏离检测）
□ 检测系统自身定期红队验证（检测也有盲区）
□ 事件复盘按"工具调用时间线"，非仅最终输出
□ 关联：MCP 服务器遥测 + Agent 行为 + 网络层（Sentinel 类 SIEM 集成）
```

## 6. 最小可部署监控包

```
1. 密钥/高熵输出检测（上文正则，输出侧 + 工具参数侧）
2. 每 Agent 行为基线（工具/端点/参数形态）
3. 新端点 + 调用量激增 + 跨调用异常链告警
4. 全量调用日志（含元数据指纹）
5. 成本/token 用量看板（防无界消耗）
```
