# Guardrails 护栏体系

> Guardrails = 运行在 LLM 之外的确定性/半确定性检查层。提示词约束是"建议"，Guardrails 是"执行"。

## 1. 五阶段护栏模型（NeMo 模型）

| 阶段 | 检查点 | 防护 |
|---|---|---|
| Input rail | 用户输入 | 越狱/注入/违规主题/PII |
| Dialog rail | 对话流转 | 主题边界、多轮越狱 |
| Retrieval rail | 检索内容 | RAG 注入 |
| Execution rail | 工具调用 | 工具使用策略（NeMo 独有） |
| Output rail | 模型输出 | 有害内容、外泄、格式 |

## 2. 输入护栏 vs 输出护栏 vs 动作授权

- **输入护栏**：模型看到前拦截。拦截成功 = 零推理成本
- **输出护栏**：返回用户前检查。最终防线，无论攻击如何构造都生效
- **动作授权（Action Authorization）**：工具执行前策略判定（allow/deny/escalate）——Agent 时代最关键的护栏层
- **注意**：输出护栏只拦"响应"，不拦"已发生的动作"（邮件已发、文件已删）。动作层必须单独防护

## 3. 护栏类型速查

| 类型 | 机制 | 成本/延迟 |
|---|---|---|
| 确定性规则 | 正则、关键词、schema 校验 | 亚毫秒 |
| 分类器 | 专用小模型（DeBERTa、Prompt Guard 2、Llama Guard 1B/8B） | 10-90ms |
| LLM-as-judge | 大模型判断 | 200-1000ms+（用于抽样/异步） |
| 框架 | NeMo Colang、Guardrails AI 验证器组合 | 按组件 |

## 4. 常用护栏库（详见 04-tools/guardrail-tools.md）

- **NeMo Guardrails**（NVIDIA，Apache 2.0）：Colang 可编程对话规则
- **Guardrails AI**（Apache 2.0）：结构化输出验证（pydantic 式）
- **LLM Guard**（Protect AI，MIT）：模块化扫描器（输入 15 类 + 输出 20 类）
- **Llama Guard / Prompt Guard**（Meta）：开源权重安全分类器
- **Lakera Guard**（商业，有免费层）：托管提示注入防火墙
- **Azure Prompt Shields / OpenAI moderation**：云侧托管检测

## 5. 生产落地模式（2026 推荐组合）

```
LLM Guard (每请求廉价扫描) 
  → NeMo Guardrails (对话+工具执行策略)
    → Guardrails AI (结构化输出校验)
      → Llama Guard 4 作为内容安全分类器（抽样/风险路由，因其 ~0.46s 延迟）
```

- 延迟预算：热路径内联 <50ms；LLM-judge 层放异步/抽样
- 模糊类别 log-and-alert；精确类别 block
- 每条规则明确"谁负责、何时触发、误报代价"
