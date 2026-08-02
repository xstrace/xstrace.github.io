# 07 参考资源

## 标准与框架

| 资源 | 链接 |
|---|---|
| OWASP Top 10 for LLM Applications 2025 | https://genai.owasp.org/llm-top-10/ |
| OWASP Top 10 for Agentic Applications 2026 | https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/ |
| OWASP MCP Top 10 | https://genai.owasp.org/ |
| MITRE ATLAS | https://atlas.mitre.org/ |
| SAFE-MCP（OpenSSF） | https://openssf.org/ |
| NIST AI RMF | https://www.nist.gov/itl/ai-risk-management-framework |
| ISO/IEC 42001 | https://www.iso.org/standard/81230.html |

## 研究论文

- EchoLeak 零点击注入（2025-09 arXiv）
- MCPTox：工具投毒基准（2025-08，arxiv.org/abs/2508.14925）
- MindGuard：决策依赖图工具投毒防护（arxiv.org/abs/2508.20412）
- SHIELDMCP：MCP 运行时防御（ACL 2026 Industry，aclanthology.org/2026.acl-industry.58）
- AgentDojo：Agent 注入攻防基准（arxiv.org/abs/2406.13352）
- PoisonedRAG：知识破坏攻击（USENIX Security 2025）
- Crescendo / TAP：多轮越狱（Microsoft 2024）
- Tool Emu / ToolPoison（2025）

## GitHub 项目

- NVIDIA/garak — LLM 漏洞扫描器（8.6k stars）
- microsoft/PyRIT — AI 红队框架
- promptfoo/promptfoo — LLM/Agent 评估与红队（18k+ stars）
- Tencent/AI-Infra-Guard — 全栈 AI 红队平台（4.4k stars）
- getagentseal/agentseal — Agent 安全套件
- luckyPipewrench/pipelock — Agent 出口防火墙
- six2dez/burp-ai-agent — Burp AI 扩展
- Goochbeater/Spiritual-Spell-Red-Teaming — Claude 越狱集合
- protectai/rebuff — 提示注入检测器
- protectai/llm-guard — LLM 输入输出扫描器
- NVIDIA/NeMo-Guardrails — 可编程护栏
- guardrails-ai/guardrails — 输出验证框架
- meta-llama / llama-recipes — Llama Guard 等

## 社区与信息源

- OWASP GenAI Security Project（Slack 按风险类别分工区）
- MITRE ATLAS 知识库
- The Hacker News / Bleeping Computer AI 安全栏目
- Microsoft Security Blog（AI 安全系列）
- 各厂商安全博客（Anthropic、OpenAI、Google、NVIDIA）

## 更新说明

本手册为持续更新文档，新增攻击/防御技术、CVE、工具会逐步补充到对应章节。建议以 GitHub 仓库为基线跟踪变更。
