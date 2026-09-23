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

## 5. 最新高相关 2026 论文（已完成全文核验）

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


---

## 8. 70 篇 Deep Search 完成后的最终边界

最新专项 Deep Search：

> **Sample wise human calibrated AI preference reliability in PBRL**

共返回 70 篇高相关工作。

最终结果未发现完整覆盖以下链路的论文：

\[
Human\ Anchor
\rightarrow
Contextual\ AI\text{-}Human\ Agreement
\rightarrow
Sample\text{-}wise\ Reliability
\rightarrow
Preference\ Weighting / Escalation
\rightarrow
Reward\ Model
\rightarrow
Preference\ RL
\rightarrow
Standalone\ Policy
\]

最接近工作包括：

- Trust or Escalate：Human agreement calibration + escalation，但无 downstream RL；
- Judging with Confidence：sample-wise Human preference distribution calibration，但无 RL；
- BACON：small Human + AI judge + context + sample-wise prediction，但用于 evaluation；
- Hybrid Preferences：Human/AI routing，但不显式建模 \(P(AI=Human|x)\)；
- Preference VLM / ROVED：Robot/PbRL + uncertain feedback + Human/Oracle，但不是 Human-anchored AI agreement calibration；
- TriTrust-PBRL：trust-aware PbRL，但以 source-wise trust 为主；
- Feature-Dependent Noise in PbRL：sample/context-dependent noise + downstream RL，但主要基于 oracle/synthetic noise。

因此最 defensible 的方法主线是：

\[
\boxed{
\rho_i
=
P(
y_i^{AI}=y_i^{Human}
\mid
trajectory_i,\ confidence_i,\ difficulty_i,\ OOD_i,\ policy\ stage_i
)
}
\]

并研究：

\[
\rho_i
\rightarrow
Preference\ Weighting
+
Human\ Escalation
+
Policy\text{-}Shift\ Recalibration.
\]

建议把 **policy-induced AI feedback reliability shift** 提升为核心实验问题。


---

## 9. 2026-09-23：Frozen Black-box Teacher 专项核验更新

基于新的 Undermind 专项 Deep Search（236 篇相关工作）以及重点全文核验，当前 prior-work 地图需要新增以下几类危险近邻。

### 9.1 Aligning Black-box Language Models with Human Judgments

该工作已经做到：

\[
Black\text{-}box\ LLM\ judgment
+
small\ Human\ calibration
\rightarrow
external\ correction\ mapping
\]

关键特点：

- 黑盒 LLM 完全冻结；
- 无需访问模型权重或 logits；
- 使用少量 Human labels 学习外部线性映射；
- 输出修正后的 human-aligned categorical judgment；
- 主要是静态 evaluation；
- 无 downstream RL；
- 无 on-policy / policy-stage adaptation。

因此：

> **“闭源 AI + 少量 Human + 外部 correction layer”本身不能作为 novelty。**

我们的差异必须落在：

\[
trajectory/context/sample\text{-}wise
+
policy\text{-}conditional
+
online\ recalibration
+
downstream\ PbRL
\]

---

### 9.2 Demo2Reward / Test-Time Prompt Optimization

该工作已经做到：

\[
Frozen\ VLM
+
few\ expert\ demonstrations
\rightarrow
prompt\ optimization
\rightarrow
robot\ reward
\rightarrow
RL
\]

并通过优化 prompt 减少 false positive / reward hacking。

因此：

> **“不修改 foundation model 权重，而优化 teacher 的使用方式”也已有明确先例。**

与当前方案的区别：

- Demo2Reward 是 task-level prompt optimization；
- 当前方案关注 sample-wise Human Preference correction / reliability；
- Demo2Reward 在 policy learning 前完成 prompt 优化；
- 当前方案考虑 policy-induced agreement shift 与 online recalibration。

---

### 9.3 VARP

VARP 使用 frozen GPT-4o/VLM preference，并通过 agent-aware regularization 让 reward learning 与 current policy 的行为分布保持联系。

因此：

> **“policy evolves / distribution shift”本身不能作为笼统 novelty。**

当前更精确的研究问题应是：

\[
\boxed{
P(y^{AI}=y^{Human}\mid d^{\pi_t})
}
\]

是否随 policy stage 改变。

也就是研究：

> **AI–Human agreement shift，而不是仅仅 reward-model distribution shift。**

---

### 9.4 LAPP

LAPP 已经实现：

\[
Frozen\ GPT\text{-}4o\text{-}mini
\rightarrow
structured\ robot\ trajectory\ preference
\rightarrow
local\ preference\ predictor
\rightarrow
PPO
\rightarrow
standalone\ robot\ policy
\]

因此：

> **closed LLM → local model → robot RL → standalone deployment 已经成立。**

Jev 的“闭源”属性不能被当作方法困难或 novelty 本身。

---

### 9.5 Preference VLM / ROVED 的新核验

全文核验确认二者比摘要层面更接近当前方案：

- sample-wise clean / noisy / uncertain 划分；
- RM↔VLM KL divergence 作为异常判断；
- uncertain → Human/Oracle；
- noisy → label flipping；
- Human/Oracle feedback 还会训练 VLM/VLE 上的 adapter；
- 对 policy/state distribution shift 也有专门处理。

因此不能声称：

- 首次 sample-wise correction；
- 首次 Human routing；
- 首次处理 policy-induced distribution shift；
- 首次用 adapter 适应 AI feedback。

当前仍可争取的区别是：

\[
\boxed{
P(AI\ preference=Human\ preference
\mid
trajectory,context,policy\ stage)
}
\]

作为显式 human-anchored target，并将其用于 correction + weighting + escalation + downstream PbRL。

---

## 10. 更新后的最安全 novelty 边界

当前最安全的表述不再只是：

> Human-Anchored Contextual Reliability

而建议升级为：

> **Policy-Conditional Human-Aligned Feedback Correction for Frozen AI Teachers in Preference-Based Reinforcement Learning**

核心组合：

\[
\boxed{
Human\text{-}Anchored
+
Frozen\ Black\text{-}box\ AI
+
Sample/Context\text{-}wise\ Correction
+
Policy\text{-}Conditional\ Reliability
+
Online\ Recalibration
+
Downstream\ Robot\ PbRL
}
\]

其中：

- Frozen AI 不是 novelty；
- correction 不是 novelty；
- calibration 不是 novelty；
- routing 不是 novelty；
- policy-aware reward learning 不是 novelty；

真正有机会的是：

> **把 Human-defined sample-wise AI correctness / correction target 与 evolving on-policy trajectory distribution 明确耦合，并证明它能改善 downstream reward learning 与 robot policy。**

详细路线见：

- `docs/12_Frozen_Black_Box_Teacher_V2.md`
