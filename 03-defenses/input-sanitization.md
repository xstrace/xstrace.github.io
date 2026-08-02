# 输入防护与净化

## 1. 输入验证流程

```
用户输入 → 规范化 → 模式过滤 → 语义分类 → 决策(BLOCK/REVIEW/ALLOW)
```

### 规范化（归一化）

绕过的第一步是编码混淆，先做归一化再检测：

- Unicode NFKD 规范化
- 同形字转换（Cyrillic→Latin、leet speak 等 30+ 映射）
- 混淆解码：Base64、ROT13、百分号编码、八进制、HTML 实体、十六进制、Unicode 转义
- 压缩重复字符（squash）
- 大小写归一化

### 模式过滤（低延迟第一层）

多视图检测（原文/规范化/压缩/小写 4 个视图），覆盖类别：
- 直接注入模式（"ignore instructions" 等）
- 高风险间接注入（"follow its footer" 等）
- 低风险间接注入（"summarize this email" 等）
- 提示泄露（"system prompt"）
- 工具滥用（exec()、/etc/passwd）
- 数据外泄（reveal api_key）
- 越狱（developer mode）
- 语义信号（replace prior rules）

## 2. 语义检测（第二层）

- embedding 相似度 vs 已知攻击模式（BGE 等本地模型，无需外部 API）
- 专用分类器：DeBERTa-v3 微调的 prompt-injection 检测器（LLM Guard 使用）
- 上下文感知分类器（Prompt Guard 2：86M 多语言 / 22M 英文）
- 指令/信息 token 分类：标记指令性 token（Das et al. 2025 方法）
- 三级决策：BLOCK / REVIEW（人工队列）/ ALLOW

## 3. 输入净化的注意点

- **不过度依赖关键词**：翻述式攻击无字符串特征
- **抑制误报**：出现良性工作流信号（debug/fix/explain）时降低弱信号权重
- 误报的代价 = 阻断合法用户，也是事故；模糊类别上"log-and-alert"优于"block"
- 净化（sanitize）而非仅检测：PII 匿名化、秘密遮蔽

## 4. 边界场景

- 多模态输入同样需要扫描（图像 OCR 文本、音频转录）
- 检索文档/工具输出在**进入上下文前**扫描（间接注入通道）
- 提示词中的指令性注入无法根治——输入过滤是纵深防御第一层，不是终点
