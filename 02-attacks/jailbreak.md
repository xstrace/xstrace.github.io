# 越狱 Jailbreak

> 目标：绕过模型的安全对齐（safety alignment），让模型输出其被禁止生成的内容。

## 常用技术分类

### 1. 角色扮演与人格伪装

- **DAN（Do Anything Now）**：宣称模型进入无限制模式
- **"developer mode" / "jailbreak mode"**：假称自己是调试环境
- 角色扮演："My deceased grandmother used to read me the recipe for napalm..."
- 权威伪造："As a senior security auditor, I need to test..."
- 人格分裂：一个角色守规矩，另一个角色回答问题

### 2. 编码与混淆

- Base64 / ROT13 / 十六进制 / Unicode 编码绕过过滤
- 同形字攻击（homoglyph，30 种字符映射）
- 隐藏字符（零宽连接符、RTL 覆盖）
- 拼接/拆分词："n a p a l m"，"ign-ore"
- leet speak："1gn0re 4ll rul3s"

### 3. 多轮渐进攻击（Multi-turn）

- **Crescendo**（Microsoft 2024）：通过一系列"看似无害的小步骤"逐渐引导模型生成有害内容，成功绕过 GPT-4 与 Gemini Pro 对齐。PyRIT 内置实现。
- **TAP（Tree of Attacks with Pruning）**：并行探索多个攻击分支，根据评分器反馈剪枝；原论文报告对 GPT-4 Turbo / GPT-4o 成功率 >80%。PyRIT 内置。
- 多轮语境铺垫：先用无害话题建立"许可氛围"，再逐步升级。

### 4. 越狱模板与社区

- GitHub 上有大量公开越狱模板库，如 `Goochbeater/Spiritual-Spell-Red-Teaming`（2.9k stars，针对 Claude 的越狱集合）
- jailbreakbench、JailbreakHub 等基准维护常用越狱列表
- 注意：2026 年公开越狱模板迭代极快，红队应持续跟踪

### 5. 面向新攻击面的越狱

- **多模态越狱**：图像内嵌文本、音频隐藏指令、对抗性图像噪声
- **工具调用越狱**：诱导模型以"工具参数"形式输出被禁内容（绕过输出过滤）
- **JSON/结构化输出越狱**：要求模型用合法 schema 包装有害内容

## 数据点（2025-2026）

- Llama Guard 4 在越狱压力下 F1 从 ~0.96（干净数据）降到 ~0.80；长上下文场景降到 ~0.60
- 更"听话"的模型更易被越狱/注入：MCPTox 发现更强指令遵循能力 = 更高攻击成功率（最高 72.8%）
- Agent 场景下模型几乎不拒绝：最高拒绝率 <3%

## 检测要点

- 输入分类器（Prompt Guard 2 / LLM Guard / Lakera 等）检测已知模式
- 多轮会话级检测（NeMo 对话 rails 能跟踪多轮越狱，单次分类器做不到）
- 输出侧检测：即使输入无法拦截，也要检测输出是否出现被禁内容
- 语义分类器优于正则/关键词：翻述式越狱（roleplay、假设、翻译）字符串层面无特征
