# AI/Agent 安全攻防速查手册

> AI Security & Agent Security Offensive/Defensive Cheat Sheet — 面向安全研究人员与工程师的手册型速查文档（GitBook 格式）。

## 这是什么

本手册系统收集 **GitHub 与互联网上最新的 AI / Agent 安全攻防技术**，覆盖：

- **攻击侧**：Prompt Injection（提示注入）、Jailbreak（越狱）、Tool Poisoning（工具投毒）、MCP 协议攻击、RAG 投毒、记忆投毒、供应链攻击、成本滥用等
- **防御侧**：输入输出防护、上下文隔离、Guardrails（护栏）、最小权限 Agent 架构、MCP 安全、红队评估、行为监控等
- **工具与案例**：开源红队/防御工具速查、真实攻击事件复盘、攻击/防御快速检查清单

## 内容更新

- 持续跟踪 OWASP（LLM / Agentic / MCP Top 10）、MITRE ATLAS、SAFE-MCP 等框架的更新
- 收录 2025–2026 年真实攻防事件（EchoLeak、Amazon Q、ClawHavoc、postmark-mcp 等）
- 收录 GitHub 活跃的攻防项目（garak、PyRIT、Promptfoo、AI-Infra-Guard 等）

## 目录导航

见 [SUMMARY.md](SUMMARY.md)，建议按顺序阅读：

1. [总体概览与威胁模型](01-overview/README.md)
2. [攻击技术详解](02-attacks/README.md)
3. [防御技术详解](03-defenses/README.md)
4. [工具速查](04-tools/README.md)
5. [真实攻击案例](05-incidents/README.md)
6. [实战速查清单](06-playbook/README.md)
7. [参考资源](07-resources/README.md)

## 在线访问

公开页面：<https://xstrace.github.io/>

## 如何更新

1. 编辑本仓库 `main` 分支中的 Markdown 源文件（章节位于 `01-overview` ~ `07-resources`，目录结构在 `SUMMARY.md`）
2. 推送 `main` 分支
3. 部署：
   - **自动（已启用）**：推送到 `main` 后，GitHub Actions（`.github/workflows/gitbook-pages.yml`）自动用 HonKit 构建并发布到 GitHub Pages，约 1 分钟生效
   - 手动预览：本地 `npx honkit build` 后打开 `_book/` 目录

## 免责声明

本手册仅用于安全研究与防御教育目的。未经授权对他人系统进行攻击测试可能违反法律法规，请务必在授权范围内使用。
