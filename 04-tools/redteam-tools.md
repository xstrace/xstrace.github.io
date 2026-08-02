# 红队测试工具

## 快速选择

| 工具 | 最佳场景 | 主要能力 | 局限 | 许可 |
|---|---|---|---|---|
| **promptfoo** | 生产 Agent 默认测试 | Agent 轨迹、直接/间接注入、工具滥用、编码 Agent 攻击、trace 证据、CI 集成、OWASP/NIST/ATLAS 报告映射 | 高级浏览器攻击流依赖托管功能 | MIT（18k+ stars） |
| **PyRIT** | 自定义攻击编排 | 多轮攻击（Crescendo/TAP）、跨域注入、Web 目标（Playwright）、scoring 链 | 需 Python 编程、无内置报告 | MIT（Microsoft） |
| **garak** | 模型层快速基线 | 120+ 注入/越狱/护栏绕过探针、结构化命中日志 | 非端到端 Agent 运行时框架 | Apache 2.0（NVIDIA，8.6k+ stars） |
| **RAMPART** | 漏洞转 CI 回归 | pytest 场景、统计试验、跨提示注入回归 | 新（2026-05 发布）、偏回归非发现 | 开源（Microsoft） |
| **AgentDojo** | 防御泛化基准 | 97 任务 + 629 安全用例、动态工具环境 | 基准环境，非即插扫描器 | 开源（研究） |
| **AI-Infra-Guard** | 全栈 AI 红队 | OpenClaw/Agent/Skills/MCP/AI Infra 扫描 + 越狱评估 | 侧重腾讯生态深度 | Apache 2.0（腾讯，4.4k+ stars） |

## 1. promptfoo

- YAML 配置、CLI + Web UI、GitHub Actions/GitLab/Jenkins 原生集成
- 测试覆盖：提示注入（直接/间接）、RAG、RBAC 违规、工具滥用、BOLA/BFLA、SSRF、提示泄露
- Agent 模式：轨迹断言（中途用错工具也会失败，不只检查最终回答）、浏览器 Agent 攻击流
- 报告映射：OWASP Top 10、NIST AI RMF、MITRE ATLAS、EU AI Act
- 2026-03 被 OpenAI 收购（约 $86M），保持 MIT 开源
- 启动：`npx promptfoo redteam` / `promptfoo eval`

## 2. PyRIT（Python Risk Identification Toolkit）

- 三原语：Orchestrator（攻击流程）+ Converter（提示转换）+ Scorer（结果评估）
- 多轮攻击：Crescendo、TAP（>80% 成功率于 GPT-4 Turbo/GPT-4o）
- 跨域提示注入：目标 A 存储毒内容 → 目标 B 处理
- 攻击转换器：Base64、ASCII 艺术、说服技术
- 支撑 100+ Microsoft 内部产品（含 Copilot）红队
- 适合：安全研究员、需要编程控制攻击链的团队

## 3. garak

- 快速探测：prompt injection、jailbreak、guardrail bypass、data leakage、replay
- 结构化输出：run report、hit log、debug log
- 支持本地/远程/API 多种目标
- 用途：发布前模型基线、回归对比、组件压力测试
- 启动：`garak --model_type openai --model_name gpt-4o`

## 4. RAMPART

- 把红队发现固化为 pytest 回归测试，进 CI 防复发
- 最成熟覆盖：跨提示注入（Agent 处理投毒文档/邮件/工单）
- 定位：已知失败的持久化守护，不是发现工具

## 5. AgentDojo

- 动态环境：真实工具调用 + 状态环境 + 形式化效用检查（不靠 LLM 模拟）
- 97 个现实任务（邮件、网银、旅行）+ 629 安全用例
- 研究结论参考：现状模型无攻击下任务解决率 <66%；最佳防御（工具过滤+检测器）把定向攻击成功率压到 ~8%
- 用于：评估防御泛化、模型对比、发表级别证据

## 6. 其他

- **DeepTeam**（Confident AI）：OWASP 映射最清晰、上手最简单
- **VulnerableMCP**：有漏洞的 MCP 服务器靶场（练习/验证扫描器）
- **MCP-Vulnerability-Database**：MCP 漏洞库
- **Spiritual-Spell-Red-Teaming**（Goochbeater）：Claude 越狱模板集合（2.9k+ stars）
- **0xClaw / NodeZero / PentestGPT**：Agent 外围（Web/API/基础设施）渗透

## 组合建议（2026）

```
CI 基线：promptfoo（或 garak 模型层）
深度评估：PyRIT 自定义编排
回归固化：RAMPART
基准对照：AgentDojo
传统层：常规 Web/API 渗透工具
```
