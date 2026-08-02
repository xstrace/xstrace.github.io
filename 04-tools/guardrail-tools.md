# Guardrails 工具

## 快速选择

| 工具 | 类型 | 干什么 | 优势 | 局限 | 许可 |
|---|---|---|---|---|---|
| **NeMo Guardrails** | 运行时框架 | Colang DSL 可编程对话规则；五阶段 rails（input/dialog/retrieval/execution/output） | 多轮状态跟踪、执行 rail 唯一支持工具策略 | Colang 学习曲线、无默认覆盖 | Apache 2.0（NVIDIA） |
| **Guardrails AI** | 输出验证框架 | 声明式 Rail 规范 + 60+ 验证器组合；pydantic 式结构化输出 | 结构输出强制、schema 校验、自动重试 | 非安全护栏（不检测注入）、需 Python | Apache 2.0 |
| **LLM Guard** | 扫描器库 | 15 输入 + 20 输出扫描器：注入、PII 匿名化、密钥、毒性、主题禁用、代码检测 | 模块化、PII 匿名化强、完全本地 | 多模型内存/延迟开销 | MIT（Protect AI，2.5M+ 下载） |
| **Llama Guard 4** | 分类器模型 | 多模态 12B，14 类 MLCommons 危害分类 + S14 代码滥用 | 开源权重、多模态、可自托管 | ~0.46s 延迟、长上下文退化（F1 0.60） | Llama 社区许可 |
| **Prompt Guard 2** | 分类器模型 | 注入/越狱二元分类；86M 多语言 / 22M 英文 | 极小、上下文感知、可扫检索内容 | 仅注入/越狱 | MIT（Meta） |

## 1. NeMo Guardrails（NVIDIA）

- Colang DSL 定义对话流与护栏：topical rails（只讨论 X）、safety rails（不输出 Y）、jailbreak rails（检测覆盖尝试）
- 唯一把工具调用策略纳入五阶段模型的框架
- 状态跟踪能抓多轮越狱（单次分类器做不到）
- 局限：beta、护栏必须显式定义、LLM rails 延迟 200-1000ms（放异步/抽样）

## 2. Guardrails AI

- Guard = 可组合验证器管道；60+ 验证器（ToxicLanguage、DetectPII、正则、SQL 注入检查…）
- 输出强转为 Pydantic 模型，失败可自动重问
- v0.10.0（2026-04）支持 server 模式（REST）
- 诚实定位：质量与格式工具，不是对抗安全工具；生产需配注入检测层

## 3. LLM Guard（Protect AI）

- PromptInjection 扫描器：fine-tuned DeBERTa-v3 分类器
- Anonymize（PII）、Secrets、BanTopics、Toxicity、代码检测…
- 扫描任意文本：检索文档、工具结果都可先过扫描（间接注入闭环）
- 独立 API server 或进程内运行

## 4. Llama Guard 4 / Prompt Guard 2（Meta）

- Llama Guard 4：12B 密集多模态分类器，输出 `safe` / `unsafe` + 类别码
  - 干净数据 F1 ~0.961（HarmBench）；越狱压力下 ~0.80；长上下文 ~0.60（高误报）
  - 生产用法：抽样/风险路由，不放每请求热路径
- Prompt Guard 2：专攻注入/越狱的微型分类器，可与 Llama Guard 组合

## 5. 生产组合模式（2026 共识）

```
LLM Guard（每请求廉价扫描：注入/PII/密钥/毒性）
  → NeMo Guardrails（对话 + 工具执行策略；LLM 判断放异步）
    → Guardrails AI（结构化输出校验）
      → Llama Guard 4（内容安全分类，抽样/高风险路由）
```

配套建议：
- 延迟预算：内联 <50ms（分类器级）；200-1000ms（LLM-judge 级）只用于标志流量/异步
- 模糊类别 log-and-alert；精确类别 block
- 所有基准在对抗压力下都会退化 → 保持固定红队节奏，勿迷信单一工具
