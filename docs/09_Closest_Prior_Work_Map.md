# 09. Closest Prior Work Map

## 1. 目的

这份文档只回答一个问题：

> **我们当前方法的每个模块，最近的 prior work 是谁？还有哪一段没有被完整做掉？**

---

## 2. 研究链路

当前设想：

\[
Human\ Anchor
\rightarrow
AI/Jev\ Preference
\rightarrow
Human\text{-}Aligned\ Reliability
\rightarrow
Preference\ Weighting
\rightarrow
Selective\ Escalation
\rightarrow
Reward\ Model
\rightarrow
RL
\rightarrow
Standalone\ Policy
\]

---

## 3. 模块级对照

| 模块 | 相关工作 | 是否已基本解决 | 对我们的影响 |
|---|---|---:|---|
| Human pairwise preference → RM → RL | Christiano et al. | ✅ | 经典基础，不是创新 |
| AI 替代 Human 产生 preference | RLAIF | ✅ | Jev-as-labeler 本身不新 |
| VLM preference → RM → robot RL | RL-VLM-F | ✅ | robot AI-preference 已成立 |
| Structured trajectory → LLM preference → robot RL | LAPP | ✅ | Jev structured-state 用法高度相近 |
| AI + 少量 Human | Preference VLM | ✅ | hybrid feedback 已存在 |
| uncertain AI sample → Human | Preference VLM / ROVED | ✅ | selective query 本身不新 |
| noisy preference filtering | RIME | ✅ | filtering/flipping 不新 |
| Human vs AI routing | Hybrid Preferences | ✅ | routing 不新 |
| confidence → preference loss weight | CW-PO | ✅ | raw confidence weighting 不新 |
| uncertainty → PPO/DPO weight | Conformal Feedback Alignment / UARM | ✅ | optimization weighting 不新 |
| small Human set → calibrate AI judges | BACON | ✅（evaluation） | calibration 思想已有，但未完整落到 robot PbRL |
| multi-source imperfect preference | RL-MSIP / multi-expert work | ✅（理论/一般） | simple pooling 不可作为贡献 |
| Human-aligned sample-wise AI reliability → robot PbRL | **当前未确认有完整工作** | ⚠️ | 当前最核心 novelty |
| reliability 同时控制 weighting + escalation + downstream policy | **当前未确认有完整 robot-RL 工作** | ⚠️ | 可作为方法主线 |
| policy-induced shift 下持续重校准 AI trust | **当前仍相对开放** | ⚠️ | 可强化贡献 |

---

## 4. 最危险的近邻论文

### Preference VLM: Leveraging VLMs for Scalable Preference-Based Reinforcement Learning

最危险原因：

- VLM 产生 preference；
- 与 Reward Model 的 KL 用于判断 clean/noisy/uncertain；
- uncertain 样本送 Human；
- Human feedback 还可帮助适配 VLM；
- 最后训练独立 RL policy。

我们不能只做：

> AI + Human + uncertainty routing。

需要强调：

> **Human 不是只做 uncertain label，而是用于估计 AI→Human agreement probability。**

---

### ROVED: Reducing Oracle Feedback with Vision-Language Embeddings for Preference-Based RL

最危险原因：

- 自动 preference source；
- uncertainty-aware filtering；
- targeted oracle query；
- 目标就是减少人工 preference cost；
- 最终 policy standalone。

我们的差异必须落在：

\[
\text{sample-wise human-aligned calibration}
\]

而不是只落在 query efficiency。

---

### Confidence-Weighted Preference Optimization

最危险原因：

\[
AI\ confidence
\times
preference\ loss
\]

已经明确出现。

因此 raw Jev confidence 不能直接作为方法贡献。

我们需要：

\[
c^{Jev}
\rightarrow
Human\text{-}Aligned\ Calibration
\rightarrow
\rho
\]

---

### BACON

最危险原因：

- 小量 Human；
- 多 AI judges；
- judge-level reliability；
- sample-level uncertainty；
- Human 是 calibration anchor。

但 BACON 的主要目标是 evaluation / scoring，而不是：

\[
calibrated\ preference
\rightarrow
robot\ reward\ learning
\rightarrow
policy.
\]

这可能是当前最清楚的“跨领域迁移空位”。

---

## 5. 最新高相关 2026 论文（待全文核验）

本轮 Undermind 搜索新出现：

### Preference-Calibrated Human-in-the-Loop Reinforcement Learning for Robotic Manipulation

从标题上看，这是目前最需要优先核验的一篇。

必须回答：

- 它所谓 preference-calibrated 是校准什么？
- 是否存在 AI evaluator？
- Human 是否校准 AI？
- 是否 sample-wise reliability？
- reliability 是否进入 RM / policy？
- 是否 standalone deployment？

### TrustRoboReward

必须回答：

- isotonic calibration 是针对 reward score 还是 human preference agreement？
- 是否涉及 multi-paradigm robot reward？
- 是否训练 downstream RL policy？

### Finding the Signal in the Spam

必须回答：

- worker reliability 是否能直接对应 AI judge reliability？
- 是否 sample-wise + source-wise 联合建模？
- 是否只做 reward inference，还是进入 RL？

### Trust, Don't Trust, or Flip

必须回答：

- multi-expert feedback 如何定义 reliability？
- expert 可以是 AI 吗？
- 是否有 Human anchor？
- 是否有 robot experiments？
- 是否涉及 selective escalation？

---

## 6. 当前最安全的论文表述

### 不建议写

> We introduce Jev-assisted reinforcement learning.

### 不建议写

> We use AI preference feedback to reduce human annotation.

### 不建议写

> We query humans when AI is uncertain.

### 更安全的表述

> We study whether an AI evaluator's self-reported confidence reflects agreement with human trajectory preferences, and introduce a human-anchored calibration mechanism that estimates sample-wise evaluator reliability for preference learning and downstream policy optimization.

进一步：

> Unlike methods that filter AI feedback using model disagreement or raw confidence, we estimate the probability that an AI preference agrees with the human objective and use this calibrated reliability to jointly control preference weighting and selective human escalation.

---

## 7. 当前建议 title

### Version A

**Human-Calibrated AI Preference Feedback for Reinforcement Learning**

### Version B

**Human-Anchored Reliability Calibration for AI-Assisted Preference-Based Reinforcement Learning**

### Version C（机器人版）

**Human-Calibrated AI Preference Feedback for Sample-Efficient Robot Reinforcement Learning**

当前不建议把 Jev 写进正式论文题目。

Jev 更适合作为：

> one evaluator backend / case study.
