# 红队评估与测试

> 红队是唯一能验证"防御是否真的有效"的手段。2026 年的定位：持续安全测试（CI 回归）+ 周期性深度评估。

## 1. 红队流程

```
范围定义（资产/模型/工具/租户）
  → 威胁建模（OWASP LLM/Agentic/MCP + MITRE ATLAS）
    → 攻击面枚举（输入通道、工具、检索源、记忆）
      → 攻击执行（自动扫描 + 人工深度）
        → 证据收集（工具调用轨迹，非仅最终回答）
          → 报告映射（OWASP/ATLAS/EU AI Act）
            → 修复 → CI 回归（RAMPART/promptfoo 断言）
```

## 2. 分层测试覆盖

| 层 | 测试内容 | 工具 |
|---|---|---|
| 模型层 | 越狱、注入、输出异常 | garak（120+ probes） |
| 应用层 | 直接/间接注入、RAG、提示泄露、RBAC | promptfoo |
| Agent 层 | 工具滥用、目标劫持、记忆投毒、跨域注入 | PyRIT、promptfoo agent 模式 |
| 基础设施 | API、认证、SSRF、主机 | 传统 pentest 工具 |

## 3. 关键测试思路

- **间接注入要打真实通道**：检索文档、网页、邮件、CRM 备注、MCP 工具描述
- **证据在动作层**：工具调用、出站请求、副作用、trace spans —— 比"礼貌的最终回答"重要
- **跨域注入**（cross-domain）：目标 A 存储毒内容，目标 B 处理它（PyRIT 文档明确支持）
- **多轮攻击**：Crescendo、TAP（PyRIT 内置）
- **代码 Agent 场景**：仓库提示注入、终端输出注入、密钥处理、网络出口、验证器破坏
- **防御泛化验证**：自己的测试套件过了 ≠ 防御有效 → AgentDojo 基准对照

## 4. 工具选择速查（详见 04-tools）

| 场景 | 工具 |
|---|---|
| 默认生产 Agent 测试 | promptfoo |
| 自定义攻击路径编排 | PyRIT |
| 模型快速基线 | garak |
| 漏洞转 CI 回归 | RAMPART |
| 防御泛化基准 | AgentDojo |

## 5. 持续化：从红队到 CI

- 每次发布在 CI 中跑回归（promptfoo GitHub Action / RAMPART pytest）
- 漏洞修复必须有对应回归测试
- 定期（季度/重大变更后）深度红队
- 攻击面随功能变化（新工具、新检索源、新记忆）→ 重新评估

## 6. 合规映射

- 报告映射到 OWASP LLM Top 10 / Agentic Top 10 / MITRE ATLAS / NIST AI RMF / EU AI Act（promptfoo 内置映射）
- AIBOM 更新：每次红队发现 → 组件记录更新
- 审计证据：红队报告 + 修复记录 + 回归测试
