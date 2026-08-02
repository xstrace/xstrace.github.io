# 检测与防护服务

## 提示注入检测工具对比（2026）

| 工具 | 托管/自托管 | 许可 | 声称检测率 | p50 延迟 | 直接注入 | 间接/RAG 注入 |
|---|---|---|---|---|---|---|
| **Lakera Guard** | 托管 API | 商业（免费层） | 98%+ | <50ms | ✔ | ✔（100+ 语言） |
| **LLM Guard** | 自托管 | MIT | DeBERTa-v3 | 10-50ms | ✔ | ✔（扫任意文本） |
| **StackOne Defender** | 自托管（npm） | Apache 2.0 | ~88.7% acc / 90.8% F1 | <10ms | 部分 | ✔（专用） |
| **Rebuff** | 自托管/SaaS | Apache 2.0 | 多层 | 10-50ms+ | ✔ | 有限 |
| **Prompt Guard 2** | 自托管模型 | MIT | 二元分类 | 10-50ms | ✔ | ✔（上下文感知） |
| **Vigil** | 自托管 | Apache 2.0 | 集成（YARA+DeBERTa+向量） | 10-50ms+ | ✔ | 有限 |
| **NeMo Guardrails** | 自托管 | Apache 2.0 | 可编程 | LLM rails 200-1000ms | ✔ | ✔（retrieval rail） |
| **Azure Prompt Shields** | 托管 API | 商业（免费层） | 直接+间接检测 | 网络往返 | ✔ | ✔ |
| **OpenAI moderation** | 托管 API | API 用户免费 | 内容分类 | 网络往返 | 内容类 | 内容类 |

## 1. Lakera Guard

- 实时检测：提示注入、越狱、系统提示提取、PII、恶意内容
- 一行集成；免费自服务层
- 权衡：提示文本出网 + 每调用成本
- 适用：想零维护托管防火墙、可接受文本外发的团队

## 2. StackOne Defender

- 专攻**间接注入**：包裹工具调用，结果进 LLM 前检查
- 22MB、CPU-only、<10ms
- 适用：Agent/工具调用场景的间接注入闭环（用户输入分类器看不到的通道）

## 3. Rebuff

- 分层检测 + **canary 机制**：提示中注入唯一 token，若模型在输出中泄露它 → 检测到操纵
- 在线学习：用户上报的成功攻击回馈检测模型
- 局限：窄范围（注入专用）、API 依赖（可自托管）

## 4. Vigil

- 集成检测：YARA 规则 + DeBERTa 分类 + 向量相似度
- 支持自定义 YARA 签名
- alpha 阶段

## 5. 云侧托管（企业）

- **Azure Prompt Shields**：直接（越狱）+ 间接（文档注入）检测；Microsoft 明确建议用于 MCP 工具响应与描述扫描
- **OpenAI moderation**：免费、内容安全分类（不是注入检测主力）
- **Defender for Cloud AI Protection**：运行时告警（可疑提示/工具输出）
- 数据主权注意：提示文本离开环境

## 选择矩阵

```
延迟敏感热路径（每请求）    → StackOne / Prompt Guard 2 / LLM Guard 分类器级
间接注入（RAG/工具结果）    → StackOne Defender / LLM Guard / Prompt Guard 2 / NeMo retrieval rail
多语言                    → Lakera / Prompt Guard 2（mDeBERTa）
完全本地/数据主权           → LLM Guard / StackOne / Prompt Guard 2 / Vigil
多轮对话级                 → NeMo（对话 rails）
零维护托管                  → Lakera / Azure Prompt Shields
```
