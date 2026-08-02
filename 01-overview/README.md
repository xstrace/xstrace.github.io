# 01 总体概览与威胁模型

本章建立对 AI / Agent 安全威胁的全局认知：

- [威胁模型与攻击面](threat-model.md)：LLM 应用、Agent、MCP 生态的分层威胁模型
- [安全框架速查](frameworks.md)：OWASP LLM Top 10、OWASP Agentic Top 10、OWASP MCP Top 10、MITRE ATLAS、SAFE-MCP 等

## 核心认知（先记住这三条）

1. **LLM 无法可靠区分"指令"与"数据"**——这是提示注入等一切攻击的根源（OWASP 将其列为 LLM01，架构性缺陷而非实现缺陷）。
2. **Agent 把"读"变成"做"**——传统 LLM 攻击改变输出，Agent 攻击改变行为。同样的注入，对聊天机器人只是说错话，对 Agent 就是数据外泄、删库、转账。
3. **MCP 把信任边界合并为单一提示**——工具描述、系统提示、用户输入、检索文档、工具输出全部混在同一个上下文里，工具元数据已被视为"可写指令"而非"可信配置"。
