# 输入防护与净化

> 纵深防御第一层。目标：把明显攻击拦截在模型之外；难点：翻述式攻击无字符串特征，语义检测兜底。

## 1. 处理流水线

```
用户输入 → ①规范化（归一化+解码） → ②模式过滤（正则，亚毫秒）
        → ③语义分类（embedding/分类器，10-90ms） → 决策 BLOCK/REVIEW/ALLOW
```

## 2. 规范化（归一化）代码示例（Python）

```python
import re, base64, html, unicodedata

def normalize(text: str) -> list[str]:
    """返回 4 个视图供后续检测：原文/规范化/压缩/小写"""
    nfkd = unicodedata.normalize("NFKD", text)              # Unicode 规范化
    nfkd = nfkd.translate(_HOMOGLYPH_MAP)                    # 同形字映射（Cyrillic→Latin 等）
    squashed = re.sub(r"(.)\1{2,}", r"\1", nfkd)            # 压缩重复字符
    return [text, nfkd, squashed, nfkd.lower()]

# 混淆解码尝试（部分解码失败不影响其他视图）
def try_decode(s: str) -> str:
    for decoder in (lambda t: base64.b64decode(t).decode("utf-8", "ignore"),
                    lambda t: bytes.fromhex(t).decode("utf-8", "ignore"),
                    lambda t: html.unescape(t)):
        try:
            return decoder(s)
        except Exception:
            continue
    return s
```

## 3. 模式过滤正则库（可直接复制）

```python
# 七大检测类别（CASCADE 分类法，31 攻击类型 / 6 层级）
PATTERNS = {
    "direct_injection": [      # 直接注入（19 类）
        r"\bignore\s+(all\s+)?(previous|prior|above)\s+(instructions?|rules|prompts?)\b",
        r"\boverride\b.{0,30}\b(instructions?|rules)\b",
        r"\bdisregard\b.{0,30}\b(previous|prior)\b",
        r"\byou\s+are\s+(now\s+)?(an?\s+)?(unrestricted|developer|system|admin)\b",
        r"\bdo\s+anything\s+now\b", r"\bdeveloper\s+mode\b",
        r"\bsystem\s+prompt\b.{0,40}\b(output|show|reveal|print)\b",
        r"\bforget\s+(all|everything)\b",
    ],
    "indirect_injection": [    # 间接注入
        r"\bfollow\s+its\s+footer\b", r"\bsummarize\s+this\s+(email|mail|message)\b",
        r"\bwhen\s+summarizing\b.{0,60}\b(output|also|include)\b",
        r"\b(attach|send|exfiltrate|copy)\b.{0,40}\b(attacker|bcc|external|endpoint)\b",
    ],
    "prompt_leakage": [        # 提示泄露（14 类）
        r"\b(show|print|reveal|output|repeat)\b.{0,40}\b(system|initial|first)\s+prompt\b",
        r"\bwhat\s+was\s+(the\s+)?first\s+(sentence|message|instruction)\b",
        r"\b(initial|first)\s+instructions?\s+verbatim\b",
        r"\brepeat\s+(your|the)\s+(initial|first)\s+instructions?\b",
    ],
    "tool_abuse": [            # 工具滥用（11 类）
        r"\bexec\(|eval\(|os\.system|subprocess", r"\b/etc/passwd\b",
        r"\brm\s+-rf\b", r"\bcurl\b.{0,30}\b(attacker|evil|\d+\.\d+\.\d+\.\d+)",
        r"\bsql\s+injection\b|UNION\s+SELECT", r"\bchmod\s+777\b",
    ],
    "data_exfiltration": [     # 数据外泄（11 类）
        r"\b(api_?key|token|password|secret|credential)\b.{0,40}\b(reveal|output|send|include)\b",
        r"\boutput\b.{0,30}\b(all|every|entire)\b.{0,30}\b(conversation|context|emails|contacts)\b",
    ],
    "jailbreak": [             # 越狱（15 类）
        r"\bdan\b|do\s+anything\s+now", r"\bstan\b|stay\s+true", r"\baim\b",
        r"\bdeveloper\s+mode\b", r"\bno\s+restrictions\b", r"\buncensored\b",
        r"\bgrandma\b|deceased\b.{0,30}\b(read|told|recipe)\b",
    ],
    "semantic_signals": [      # 语义信号（43+ 类，用于组合加权）
        r"\breplace\s+prior\s+rules\b", r"\bhigher\s+priority\b",
        r"\bignore\s+all\s+filters\b", r"\bauthenticated\s+by\s+admin\b",
        r"\balready\s+approved\b", r"\bno\s+need\s+to\s+confirm\b",
    ],
}

def match_any(text: str) -> list[str]:
    hits = []
    for cat, pats in PATTERNS.items():
        for p in pats:
            if re.search(p, text, re.I | re.S):
                hits.append((cat, p))
    return hits
```

## 4. 语义分类层（调用示例）

```python
# LLM Guard（Protect AI）—— 自托管分类器
from llm_guard import scan_prompt
from llm_guard.vault import Vault
from llm_guard.input_scanners import PromptInjection, Anonymize, Secrets

scanners = [
    PromptInjection(threshold=0.6),   # DeBERTa-v3 fine-tuned
    Secrets(),                        # 密钥检测
]
sanitized, results = scan_prompt(scanners, "user input", Vault())

# Meta Prompt Guard 2（86M / 22M，本地）
from transformers import AutoModelForSequenceClassification
model = AutoModelForSequenceClassification.from_pretrained("meta-llama/Prompt-Guard-2-86M")
# 输出：benign / jailbreak / injection

# 本地 embedding 相似度（CASCADE 模式：BGE + Llama3 兜底）
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("BAAI/bge-small-zh-v1.5")
sim = model.encode(attack_pattern).dot(model.encode(user_input))
```

## 5. 阈值与决策策略

| 决策 | 条件 | 动作 |
|---|---|---|
| BLOCK | 高置信命中（正则强特征 / 分类器 >0.72） | 直接拒绝（精确类） |
| REVIEW | 模糊命中（弱信号组合 / 分类器 0.5-0.72） | 人工审核队列（CASCADE 三态决策） |
| ALLOW | 无命中 | 放行（可异步抽样复核） |

```
□ 误报代价高（阻断合法用户也是事故）→ 模糊类 log-and-alert
□ 弱信号抑制：出现良性工作流词（debug/fix/explain）时降低权重
□ 多视图检测：同一条规则跑 4 个视图（原文/规范化/压缩/小写）
```

## 6. 边界场景补充

```
□ 检索文档/工具输出在进上下文前同样扫描（间接注入通道）
□ 多模态输入：图像 OCR 文本 + 音频转录也要过扫描
□ 净化而非仅检测：PII 匿名化、密钥遮蔽后再入上下文
□ 输入过滤是"减震器"不是"防火墙"——后续层必须跟上
```
