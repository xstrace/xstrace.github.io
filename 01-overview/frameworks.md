# 安全框架速查

## 1. OWASP Top 10 for LLM Applications 2025（LLM 应用）

当前事实标准威胁清单，覆盖 LLM 应用全生命周期。

| ID | 风险 | 关键缓解 |
|---|---|---|
| LLM01 | 提示注入 Prompt Injection | 最小权限、HITL、上下文隔离、输入过滤 |
| LLM02 | 敏感信息泄露 | 数据脱敏、输出过滤、检索层权限控制 |
| LLM03 | 供应链漏洞 | 依赖锁定、SBOM、可信模型注册表 |
| LLM04 | 数据与模型投毒 | 数据源验证、对抗评估、反馈限流 |
| LLM05 | 不当输出处理 | 输出视为不可信输入、结构化输出、参数化查询 |
| LLM06 | 过度自主 Excessive Agency | 最小权限、下游完整授权、HITL、工具调用审计 |
| LLM07 | 系统提示泄露 | 提示中不存密钥、安全逻辑外置、Guardrails |
| LLM08 | 向量与 Embedding 弱点 | 向量库租户隔离、检索层权限、文档准入校验 |
| LLM09 | 错误信息 | RAG 接地、事实核查、高影响场景人工确认 |
| LLM10 | 无界消耗 | 速率限制、成本上限、输出长度限制、循环检测 |

> 注意：编号不是严重度排序；单类目风险通常与其他类目组合出现（注入→过度自主→数据泄露）。

## 2. OWASP Top 10 for Agentic Applications 2026（Agent 应用）

2025-12-09 发布，面向自主 Agent（计划、记忆、工具调用、委托授权）。

| ID | 风险 | 主防御 | 对应 LLM Top 10 |
|---|---|---|---|
| ASI01 | Agent 目标劫持 | 检索内容视为不可信；目标约束 | LLM01 |
| ASI02 | 工具滥用与利用 | 最小工具范围；参数校验；运行时策略 | LLM06 |
| ASI03 | 身份与权限滥用 | 每 Agent 独立身份；短时凭证 | 无对应 |
| ASI04 | Agent 供应链漏洞 | 签名组件；AIBOM；来源追溯 | LLM03 |
| ASI05 | 意外代码执行（RCE） | 沙箱执行；默认拒绝出网 | LLM05 |
| ASI06 | 记忆与上下文投毒 | 记忆写入校验；临时上下文；租户隔离 | LLM04/LLM08 |
| ASI07 | 不安全的 Agent 间通信 | 双向认证；消息签名 | 无对应 |
| ASI08 | 级联故障 | 爆炸半径隔离；熔断器 | 无对应 |
| ASI09 | 人-Agent 信任利用 | 高敏操作强制确认；防自动化偏见 | LLM09 |
| ASI10 | 失控 Agent | 行为基线监控；终止开关；审计 | LLM06 |

## 3. OWASP MCP Top 10（MCP 生态，2025 发布）

面向 MCP（Model Context Protocol）的专属清单。典型条目（以 2025 版为例）：

| ID | 风险 | 说明 |
|---|---|---|
| MCP01 | Prompt Injection | 经工具输出/描述注入 |
| MCP02 | 数据外泄 Data Exfiltration | 经工具调用的敏感数据外发 |
| MCP03 | 工具投毒 Tool Poisoning（DREAD 46.5/50，Critical） | 子技术：Rug Pull、Schema 投毒、工具影身 Tool Shadowing |
| MCP04 | 越权访问 Unauthorized Access | 未认证的暴露服务器 |
| MCP05 | 输入验证缺失 | 工具参数校验不足（SSRF、命令注入等） |
| MCP06 | 认证/授权缺陷 | Token 传递与重放 |
| MCP07 | 拒绝服务 | 资源耗尽 |
| MCP08 | 供应链投毒 | 恶意服务器分发 |
| MCP09 | 服务器认证缺失 | 无签名/无校验 |
| MCP10 | 客户端安全缺陷 | 客户端静态校验与参数可见性不足 |

> 实证：2026 年对 7 款主流 MCP 客户端的评估显示攻击成功率 0%（Claude Desktop）到 100%（Cursor）不等。

## 4. MITRE ATLAS

攻击者视角的知识库，基于 ATT&CK 方法论的 AI 系统对抗威胁图谱：https://atlas.mitre.org

常用战术：侦察 → 资源开发 → 初始访问 → ML 模型访问 → 开发/执行 → 持久化 → 提权 → 规避 → 凭据访问 → 数据收集 → 数据外泄 → 影响

典型技术示例：
- `AML.T0040` 提示注入（直接/间接）
- `AML.T0051` 训练数据投毒
- `AML.T0020` 模型逆向
- `AML.T0018` 模型窃取/提取

## 5. SAFE-MCP（OpenSSF，Linux 基金会）

MCP 攻击战术目录，14 个战术类别、80+ 技术，基于 MITRE ATT&CK 方法论。防御项目 SHIELDMCP 基于该分类法构建运行时防御（工具投毒攻击成功率从 74% 降至 <9%）。

## 6. 合规与治理框架

| 框架 | 关注点 |
|---|---|
| NIST AI RMF 1.0 | AI 风险管理（治理、映射、测量、管理） |
| ISO/IEC 42001 | AI 管理体系（AIMS） |
| EU AI Act | 分层监管；禁止性实践 2025-02 生效；GPAI 义务 2025-08 生效 |
| AI BOM（AIBOM） | Agent 软件物料清单（组件、技能、模型、数据来源） |

## 7. 框架选择速查

```
传统 LLM 应用评估   → OWASP LLM Top 10 2025 + MITRE ATLAS
Agent 应用评估      → OWASP Agentic Top 10 2026 + AIBOM
MCP 生态评估        → OWASP MCP Top 10 + SAFE-MCP 战术目录
合规/治理           → NIST AI RMF + ISO 42001 + EU AI Act
红队报告映射        → MITRE ATLAS（+ ATT&CK 用于传统层）
```
