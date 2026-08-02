# 02 攻击技术

本章按攻击类别详解当前已知的 AI / Agent 攻击技术，每条包含**原理、攻击手法、真实案例、检测要点**。

- [提示注入 Prompt Injection](prompt-injection.md) — 直接/间接注入、工具输出注入、绕过技巧
- [越狱 Jailbreak](jailbreak.md) — 角色扮演、编码混淆、多轮渐进（Crescendo/TAP）等
- [工具投毒与 MCP 攻击](tool-poisoning.md) — 工具描述投毒、Rug Pull、Schema 投毒、参数注入、跨工具链
- [RAG 与向量库攻击](rag-vector-attacks.md) — 知识库投毒、embedding 攻击、跨租户泄露
- [模型与数据投毒](model-poisoning.md) — 训练数据投毒、后门、微调攻击、反馈污染
- [Agent 专属攻击](agent-attacks.md) — 目标劫持、记忆投毒、Agent 间攻击、HITL 绕过
- [供应链攻击](supply-chain.md) — 恶意技能/服务器/依赖、Rug Pull、依赖混淆
- [信息泄露与提示词窃取](disclosure-leakage.md) — 系统提示泄露、训练数据记忆、模型提取
- [滥用与成本攻击](abuse-dos.md) — 无界消耗、无限循环、配额耗尽
