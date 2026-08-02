# MCP / Agent 安全工具

## 开源工具

| 工具 | 定位 | 关键能力 | 许可/状态 |
|---|---|---|---|
| **AI-Infra-Guard**（腾讯） | 全栈 AI 红队平台 | OpenClaw Security Scan、Agent Scan、Skills Scan、MCP Scan、AI Infra Scan、LLM 越狱评估 | Apache 2.0，4.4k+ stars |
| **agentseal** | Agent 本机安全套件 | 扫描危险 skills/MCP 配置、供应链攻击监控、提示注入抗性测试、MCP 服务器工具投毒审计 | Python CLI |
| **pipelock** | Agent 出口防火墙 | 中介代理扫描 HTTP/MCP/A2A/WebSocket 流量：外泄、SSRF、提示注入；生成可验证审计凭证 | Go，786 stars |
| **burp-ai-agent** | Burp 扩展 | MCP 工具接入、AI 辅助分析、被动/主动扫描（AI 应用渗透进传统流程） | Kotlin，1.4k stars |
| **SHIELDMCP** | MCP 运行时代理（研究） | 三段拦截：描述完整性、参数消毒、响应分析；工具投毒 74%→<9%，间接注入 47%→<6%，+<120ms | ACL 2026 Industry 论文 |
| **MindGuard** | 决策级护栏（研究） | 注意力→决策依赖图（DDG）；检测+归因工具投毒；精度 94-99%，<1s，零 token 成本 | arXiv 2508.20412 |
| **CASCADE** | 本地三层级联检测（研究） | 正则预过滤 → BGE embedding 语义分析（Llama3 兜底）→ 输出模式过滤；全本地运行；精度 95.85%，FPR 6.06% | arXiv 2604.17125 |
| **VulnerableMCP** | 漏洞靶场 | 故意有漏洞的 MCP 服务器，用于练习与验证扫描器 | 开源 |
| **MCP-Vulnerability-Database** | 漏洞库 | MCP 已知漏洞与 PoC 收集 | 开源 |

## 商业/平台侧能力（参考）

- **Waxell MCP Gateway**：描述指纹五态管理（Pending/Drift/Trusted/Blocked/Removed）、注入扫描、HITL 强制执行
- **Microsoft 安全栈**：Prompt Shields（Azure AI Content Safety）、Defender for Cloud AI Protection、Entra Agent ID、Purview DLP、Sentinel 关联、Cloud Apps 端点发现
- **StackOne Defender**：npm 包，包裹工具调用检查结果（间接注入专用，<10ms，Apache 2.0，~88.7% 准确率）

## 使用建议

```
存量审计：AI-Infra-Guard / agentseal（扫描已部署配置与技能）
运行时防护：pipelock（出口代理）或 SHIELDMCP 类拦截层
工具元数据治理：指纹 + 漂移检测（自建或网关）
决策监控：MindGuard 类决策级检测（研究前沿）
验证：VulnerableMCP 靶场 + MCPTox 思路的投毒测试
```
