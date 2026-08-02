# MCP 生态安全

> MCP 是 2025-2026 增长最快的攻击面（工具投毒、供应链、协议漏洞）。此处聚焦防御侧落地。

## 1. 服务器治理（Supply Chain Governance）

- **租户级白名单**：批准发布的 MCP 服务器清单；禁用 "Allow all"
- **发布者审查**：服务负责人 + 安全审查双签字；第三方服务器必须文档化负责人
- **版本锁定**：生产环境禁止自动更新；更新 diff 当代码审查
- **最小工具启用**：只开 Agent 真正需要的工具
- **来源追溯（AIBOM）**：记录每个服务器的来源、版本、负责人
- **签名验证**：支持签名时校验工具定义数字签名

## 2. 元数据防护（Tool Metadata Protection）

- **描述指纹（Fingerprinting）**：首次连接对 description/schema 做哈希；变更 → Drift Detected → 阻止调用 → 人工审查（五态：Pending / Drift Detected / Trusted / Blocked / Removed）
- **描述扫描**：指令性语言检测、隐藏 Unicode、注入载荷（Markdown/HTML）
- **语义意图验证**：分类器区分"能力说明"与"动作指令"（SHIELDMCP τ=0.72 隔离）
- **变更审查**：工具描述变更 = 系统提示变更级别的审查（Microsoft 强调）

## 3. 运行时拦截（Runtime Interception）

网关/代理模型（SHIELDMCP、Waxell、pipelock 等）：
- **Stage 1**：工具注册时描述完整性检查
- **Stage 2**：出站参数 schema 校验 + 字符串参数注入扫描（SQL/命令/注入载荷）
- **Stage 3**：响应分析——指令性 token 检测、上下文边界包裹、跨调用相关性（依赖图）
- 实证效果（SHIELDMCP）：工具投毒攻击成功率 74%→<9%；间接注入 47%→<6%；中位延迟 +<120ms

## 4. 决策级防护（Decision-Level Guardrails）

- **MindGuard**：注意力 → 决策依赖图（DDG），检测"未被调用工具对最终调用的异常影响"
  - 检测精度 94%-99%，归因准确率 95%-100%，处理 <1s，零额外 token
- 原理：成功投毒会在注意力中留下足迹（未调用工具的强激活 + 用户查询的弱激活）
- 零信任策略形式化："被调用的工具只能受自身描述与用户查询影响"

## 5. 客户端安全要求

- 静态校验工具描述（多数客户端缺失：Cursor 100% 攻击成功率 vs Claude Desktop 0%）
- 参数可见性：用户应能看到传给工具的全部参数
- 响应展示工具描述（安装时 + 变更时）
- 安全敏感工作选择经审计的客户端（Claude Desktop、Cline 推荐）

## 6. 出站控制（Egress）

- 阻断 Agent 运行时任意出站 HTTP（pipelock 等代理模式）
- 出站白名单 + DLP（敏感数据在出站载荷中被拦截，如 Purview DLP 检查工具调用参数）
- 新端点检测（Defender for Cloud Apps 等）

## 7. 行动清单（审计现有部署）

```
□ 盘点所有 MCP 服务器（含废弃）
□ 导出所有工具描述，与已知良好版本 diff
□ 移除未使用工具
□ 锁定版本
□ 阻断任意出站 HTTP
□ 读/写工具分离
□ 跨信任域数据移动审批
□ 记录 prompts、元数据、工具调用、输出
□ 为使用过不可信/近期变更工具的 Agent 轮换凭证
```
