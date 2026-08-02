# 供应链攻击（Supply Chain）

> 核心原理一句话：**AI 栈的信任链条极长——模型权重、依赖包、网关、代理、插件、工具市场，任何一环被投毒，下游所有 Agent 都继承该环节的权限。** 攻击者不需要攻破你的应用，只需要攻破你信任的"上游"。

## 0. 为什么供应链是 AI 时代的高价值目标（原理剖析）

```
AI 供应链的特殊性（与传统软件供应链不同）：
1. 凭证聚合：LLM 网关（如 LiteLLM）集中持有所有模型提供商的 API key
   → 攻破一个包 = 拿到 OpenAI/Anthropic/Azure 等全部密钥（"一个包，所有钥匙"）
2. 深度依赖树：DSPy/MLflow/CrewAI/OpenHands 等框架传递依赖
   → LiteLLM 存在于 Wiz 调研的 36% 云环境中
3. 工具即权限：Agent 平台/MCP 服务器本身持有高价值凭证
   → 投毒目标选择规律：选"需要广泛读取权限"的工具
     （扫描器、网关、路由——这些工具按设计就要读凭证和配置）
4. 模型层：模型权重、LoRA、数据集也可被投毒（见 model-poisoning 章节）
```

**供应链攻击链的共同模式**（MITRE ATLAS 供应链类攻击）：
```
攻击者 → 攻破上游一环（凭证/账号/CI） → 向链条注入恶意工件
       → 下游用户无感安装/更新 → 恶意代码在受害者环境执行
       → 窃取凭证 → 横向移动 → 二次投毒
```

---

## 1. 真实案例解剖：TeamPCP 级联攻击（2026.03，史上最复杂的多生态供应链攻击）

**攻击时间线**（CSA 研究报告还原）：

```
2026.03.19  攻破 trivy-action（GitHub Action，安全扫描器）：
            重写 git tag 指向恶意版本 v0.69.4，携带凭证收割载荷
2026.03.20  CanisterWorm：npm 自传播蠕虫
            窃取 1 个 npm 发布 token → 60 秒内枚举该 token 全部包
            → 插入恶意代码 → 发布新版（50+ npm 包受影响）
2026.03.21  用窃取的 aqua-bot 服务账号凭证，篡改 Aqua Security
            GitHub 组织 44 个仓库
2026.03.24  LiteLLM 1.82.7/1.82.8 投毒 PyPI：
            触发链 = LiteLLM 的 CI/CD 跑"投毒的 Trivy"
            → Trivy 窃取 CI runner 中的 PYPI_PUBLISH token
            → 攻击者用合法凭证发布恶意版本（通过所有完整性校验！）
2026.03.25  Telnyx SDK 也被投毒
```

**恶意载荷三阶段**（1.82.7 源码注入 + 1.82.8 `.pth` 文件）：

```python
# 阶段 1：凭证收割（>50 类密钥）
# 扫描：SSH keys / AWS·GCP·Azure IAM 凭证 / K8s 配置 / Docker 令牌
#       npm auth / Vault tokens / 加密货币钱包 / shell 历史
#       ★ 重点：LLM API keys（OpenAI、Anthropic、Azure 全部）
# 阶段 2：加密外泄
# 收割数据 → AES-256-CBC 加密（随机会话密钥）
#        → RSA-4096-OAEP 公钥包裹会话密钥（只有攻击者可解）
#        → HTTPS POST 到 models[.]litellm[.]cloud
#          （2026.03.23 注册——比恶意发布早 1 天，伪装成正常 API 流量）
# 阶段 3：持久化后门 + K8s 横向
# ~/.config/sysmon/sysmon.py 每 50 分钟轮询 checkmarx[.]zone/raw
# → 下载 URL 指向的载荷执行（运营者随时切换投递真实载荷）
# → 发现 K8s service account token → 读取所有 namespace 的全部 secrets
#   → 向每个节点部署特权 pod（挂载宿主文件系统）→ 节点后门
```

**关键技术细节（为什么传统防线失效）：**
- `.pth` 文件在 Python 解释器**每次启动**时执行（无需 import！），`pip install`、`python -c`、IDE 语言服务器都会触发
- `pip uninstall` 移除包后 `.pth` 仍留在 site-packages → 卸载不彻底 = 持续后门
- 恶意版本用**合法凭证**发布 → 通过哈希校验和所有标准完整性检查
- C2 使用 ICP 区块链（Internet Computer Protocol）→ 域名注册商/托管商无法关停
- 攻击者用 AI agent（openclaw）做自动化攻击目标选择——**AI 参与供应链攻击的首批案例**

**影响与应对**：
```
■ 受影响窗口：2026.03.24 10:39-16:00 UTC 安装/升级 litellm 的环境
■ 检查 IOC：site-packages/litellm_init.pth 是否存在；出站流量到
  models[.]litellm[.]cloud / checkmarx[.]zone
■ 处置：假定完全失陷 → 轮换所有 CI/CD secrets、云凭证、
  全部 LLM API keys（无例外）；删除 .pth；升级到 v1.83.0+
■ 教训：官方 Docker 镜像（锁定依赖）不受影响——锁版本是保命手段
```

---

## 2. 恶意 LLM API 路由器（2026 实证研究：arxiv.org/abs/2604.08407）

**原理**：第三方 LLM API 路由器（淘宝/闲鱼/Shopify 购买的付费路由器 + 社区免费路由器）作为应用层代理，**明文访问所有流经的 JSON 载荷**（tool-calling 请求），且提供商与上游模型之间无加密完整性保障。

**攻击类别**：
```text
AC-1  载荷注入（Payload Injection）：路由器在流经请求中注入
      恶意指令（如要求模型输出敏感信息）
AC-1.a 依赖定向注入：检测到特定依赖/框架时触发（自适应规避）
AC-1.b 条件投递：满足条件（特定用户/时段）才投递恶意载荷
AC-2  密钥外泄（Secret Exfiltration）：拦截并窃取请求中的凭证
```

**实测数据**（428 个路由器）：
```
9/428 活跃注入恶意代码
17 个触及研究者部署的 AWS canary 凭证
1 个从研究者私钥中转走 ETH
投毒实验：泄露的 OpenAI key 生成 1 亿+ tokens；
弱配置诱饵 → 20 亿账单 tokens、440 会话中外泄 99 个凭证
```

**客户端侧防御**（论文提出的三种可部署防御）：
```python
# 1. fail-closed 策略门（policy gate）：未获信任的路由器拒绝放行
# 2. 响应侧异常筛查（response-side anomaly screening）：
#    校验响应是否与请求/任务预期一致（防注入的响应）
# 3. 追加式透明日志（append-only transparency logging）：
#    路由器行为全部记录，可审计可溯源
```

---

## 3. 其他重要向量

### 3.1 模型仓库投毒（Model Zoo / HF）

```text
# HuggingFace 模型/LoRA/数据集投毒：
# - 恶意权重后门（触发词激活）
# - 数据集投毒（指令跟随模型训练数据注入恶意样本）
# - 过期模型重命名发布（冒充新版本）
# 2026 年趋势：AI 编码 Agent 的"技能生态"（skill marketplace）
# 成为新投毒面——恶意 skill 在代码生成时植入漏洞
# （Supply-Chain Poisoning Attacks Against LLM Coding Agent Skill Ecosystems, 2026）
```

### 3.2 工具市场/插件投毒

```text
# MCP 服务器市场、IDE 插件市场、Agent 技能市场：
# - 发布者身份冒用（typosquatting：mcp-server-files vs mcp-server-fils）
# - 恶意工具描述诱导 Agent 调用恶意工具
# - 通过更新通道投毒（合法工具 + 恶意新版本）
# 检视：发布者白名单、版本锁定、工具元数据指纹（见 mcp-security 章节）
```

### 3.3 代码生成投毒（LLM 即供应链本身）

```text
# 攻击面：模型生成的代码/依赖建议被投毒
# - 训练数据注入：模型推荐恶意包名（typosquatting 包）
# - 编码 Agent 的上下文注入：仓库 README 中的恶意指令
#   诱导 Agent 引入恶意依赖
# - 2026 实证：编码 Agent 技能生态投毒（新研究领域，ACL/arXiv 2026）
```

---

## 4. 防护清单（CSA/Snyk/LiteLLM 经验综合）

```bash
# 依赖与发布侧
□ 锁定版本（requirements.txt pinning；官方镜像优先）——LiteLLM 官方
  Docker 镜像不受影响即因锁定
□ 禁用自动更新（auto-update 是投毒入口）
□ 发布者白名单（租户级 allowlist）
□ 校验签名（cosign 签名镜像、attestation 验证）
□ SCA 扫描 + 新依赖 fail-build 策略
□ 检查传递依赖深度（AI/ML 框架依赖树异常深）

# 运行时与凭证侧（核心：打破级联链条）
□ CI/CD 凭证最小化：runner 凭证不得同时能发布 PyPI、访问 LLM API、
  修改 Docker Hub（TeamPCP 级联正是多权限并存导致）
□ LLM API keys 与 CI 发布凭证隔离
□ AI 工作负载环境与 CI/CD 环境隔离
□ 轮换策略：高权限工具（网关/扫描器）凭证定期轮换
□ 出站白名单：生产环境默认拒绝出网（模型 API 域名白名单）

# 检测侧
□ 监控新安装包/新版本的行为（异常进程、异常出站）
□ 检查 site-packages 中的 .pth 等持久化机制
□ 供应链 IOC 情报订阅（Snyk/Datadog/Wiz）
□ 透明日志：路由器/网关行为可审计

# 模型侧
□ 模型下载验证：SHA256 + HF 官方账号确认
□ 禁用训练数据自动拉取（训练流程隔离）
```

## 5. 参考链接

| 内容 | 链接 |
|---|---|
| TeamPCP 攻击分析（Snyk） | snyk.io/blog/poisoned-security-scanner-backdooring-litellm |
| TeamPCP 分析（Datadog Security Labs） | securitylabs.datadoghq.com/articles/litellm-compromised-pypi-teampcp-supply-chain-campaign |
| TeamPCP 分析（Trend Micro） | trendmicro.com/en_gb/research/26/c/inside-litellm-supply-chain-compromise |
| CSA 研究报告（TeamPCP，权威） | labs.cloudsecurityalliance.org（research note 20260330） |
| LiteLLM 官方安全公告 | docs.litellm.ai/blog/security-update-march-2026 |
| 恶意 API 路由器研究（2026） | arxiv.org/abs/2604.08407 |
| 编码 Agent 技能生态投毒（2026） | arxiv.org（Supply-Chain Poisoning Against LLM Coding Agent Skill Ecosystems） |
| Ultralytics AI 投毒（2024 对比案例） | blog.ultralytics.com / snyk.io |
| MAESTRO 威胁建模（CSA） | cloudsecurityalliance.org（MAESTRO framework） |
