# 11. Final Deep Search Result — Sample-wise Human-Calibrated AI Preference Reliability

更新时间：2026-09-22

## 1. Deep Search 目标

本轮 Undermind Deep Search 专门检索以下精确问题：

> 是否已有工作完整实现：由固定 AI evaluator / judge 对 trajectory / response pairs 产生 pairwise preference；使用少量可信 Human Preference 学习或校准 context-dependent、sample-wise reliability
>
> \[
> \rho_i
> =
> P(
> y_i^{AI}=y_i^{Human}
> \mid
> x_i,c_i^{AI},difficulty_i,OOD_i,stage_i
> )
> \]
>
> 并让 \(\rho_i\) 用于：
>
> 1. AI preference weighting / correction；
> 2. low-reliability sample escalation；
> 3. reward-model / policy learning；
> 4. 最终部署 policy 不再依赖 evaluator。

Deep Search 共返回 **70 篇高相关工作**。

---

## 2. 最终检索结论

Undermind 的最终总结是：

> **No paper in the searched set appears to implement the full method.**

更准确地说：

> **截至 2026-09-22，没有在本轮检索范围内发现一篇论文完整实现“Human-calibrated、context/sample-wise AI-to-human agreement probability → preference weighting / escalation → downstream PbRL / robot RL → evaluator-free policy deployment”这一整条链路。**

因此，当前仍存在一个可以辩护的 novelty gap，但必须把 claim 限制在**完整组合与 end-to-end coupling**，而不能把任何单一模块宣称为首次提出。

---

## 3. 当前不能再声称“新”的模块

本轮 70 篇文献进一步确认，下面每一部分都已有明确先例。

### 3.1 AI Preference → RL

已有：

- RLAIF
- Direct RLAIF / OAIF
- RL-VLM-F
- LAPP
- Preference VLM
- PrefCLM

因此：

> AI evaluator / LLM / VLM 产生 preference，再训练 RL policy，不新。

---

### 3.2 AI teacher only during training → standalone policy

已有：

- RLAIF
- RL-VLM-F
- LAPP
- Preference VLM
- ROVED

因此：

> “训练时用 Jev，部署时不调用 Jev”不是 novelty。

---

### 3.3 Human / AI routing

已有：

- Hybrid Preferences
- RLTHF
- Preference VLM
- ROVED
- Trust or Escalate

因此：

> “AI 不确定就问 Human”本身不新。

---

### 3.4 Confidence / uncertainty weighting

已有：

- Confidence-Weighted Preference Optimization
- Conformal Feedback Alignment
- Uncertainty-Aware Reward Modeling
- Calibrating the Evaluator
- CA-PbRL

因此：

> raw confidence 乘 loss / reward / policy update 本身不新。

---

### 3.5 Sample-wise / instance-dependent preference reliability

已有：

- Evaluating Feature Dependent Noise in PbRL
- When Human Preferences Flip: FA-DPO
- Conformal Feedback Alignment
- instance-level annotator reliability literature

因此：

> “sample-wise reliability”本身也不能作为首次贡献。

---

### 3.6 Human-calibrated AI Judge

已有：

- Trust or Escalate
- Judging with Confidence
- BACON
- Calibrate, Don't Curate
- selective/conformal judge calibration

因此：

> “用 Human 校准 AI evaluator”本身也不新。

---

## 4. 当前最接近的工作

### 4.1 Trust or Escalate: LLM Judges with Provable Guarantees for Human Agreement

这是当前最接近我们**Human–AI agreement calibration**部分的工作之一。

它已经研究：

\[
AI\ Judge
\rightarrow
Human\ Agreement
\rightarrow
Trust / Escalate
\]

并提供 human-agreement guarantee。

但是它停在：

\[
AI\ Evaluation
\]

没有继续：

\[
calibrated\ preference
\rightarrow
Reward\ Model
\rightarrow
RL
\rightarrow
Standalone\ Policy.
\]

因此它覆盖了我们的“上游 calibration / escalation”部分，但没有覆盖 end-to-end Preference RL。

---

### 4.2 Judging with Confidence

目标：

\[
p^*(x)
=
P(Y=1\mid X=x)
\]

训练 autorater 去拟合人类 preference distribution。

它已经做到：

- Human preference target；
- sample-wise probability；
- calibrated autorater。

但没有：

- preference-weighted RL；
- downstream reward-model learning；
- robot policy training；
- selective escalation into RL loop。

---

### 4.3 BACON

BACON 使用：

- small Human calibration subset；
- multiple AI judges；
- contextual embeddings；
- uncertainty features；
- sample-wise prediction。

因此它已经非常接近：

\[
Human
+
AI
+
Context
\rightarrow
sample\text{-}wise\ prediction.
\]

但目标是：

- evaluation；
- scoring；
- ranking；
- population statistics。

而不是：

\[
Preference\ Learning
\rightarrow
Reward\ Model
\rightarrow
RL\ Policy.
\]

---

### 4.4 Hybrid Preferences

Hybrid Preferences 已经研究：

> 哪些 instance 给 Human，哪些给 AI。

但是它并不显式学习：

\[
P(
AI=Human
\mid
x
)
\]

并把这个概率作为 preference training weight。

---

### 4.5 RLTHF

RLTHF：

- 找出可能的 AI feedback mistake；
- targeted Human correction；
- downstream alignment。

非常接近“Human correction of AI feedback”。

但是：

- 没有显式 sample-wise AI–Human agreement probability；
- 没有把该 probability 作为 calibration target；
- 主要是 targeted correction，而不是 reliability-aware preference weighting。

---

### 4.6 Preference VLM / ROVED

二者是机器人/PbRL 中最危险的近邻。

已经做：

\[
VLM / VLE
\rightarrow
Preference
\rightarrow
uncertainty
\rightarrow
Oracle / Human
\rightarrow
Reward\ Model
\rightarrow
RL.
\]

但 trust 主要来自：

- RM ↔ VLM disagreement；
- uncertainty；
- filtering；
- oracle query。

而不是：

\[
\boxed{
P(
AI\ preference
=
Human\ preference
\mid
trajectory/context
)
}
\]

这仍然是关键差别。

---

### 4.7 TriTrust-PBRL

TriTrust 已经：

\[
Multi\text{-}Expert
\rightarrow
Trust
\rightarrow
Reward\ Model
\rightarrow
SAC.
\]

但它主要学：

\[
\alpha_k
=
Trust(Source_k)
\]

即 source-wise reliability。

而我们要研究：

\[
\rho_{ik}
=
Trust(Source_k,Sample_i,Context_i).
\]

同一个 Jev evaluator：

\[
\rho_{i,Jev}
\]

应随着：

- trajectory difficulty；
- pair similarity；
- task；
- OOD；
- policy stage

动态改变。

---

### 4.8 Feature-Dependent Noise in PbRL

该工作已经非常明确地指出：

\[
N(\tau_1,\tau_2)
=
P(
y\neq y^*
\mid
\phi(\tau_1),\phi(\tau_2)
)
\]

即 preference noise 是 trajectory-dependent 的。

并且 Language Model noise 也具有 feature-dependent characteristics。

这说明：

> AI evaluator reliability 不能仅使用全局 source trust 表示。

但其 ground truth 主要来自 environment oracle / synthetic construction，而不是：

\[
small\ Human\ Preference
\rightarrow
AI\text{-}Human\ agreement\ calibration.
\]

---

## 5. 当前最 defensible 的 novelty

当前最安全的 novelty 不是任何单个组件，而是：

\[
\boxed{
Human\text{-}Anchored
+
AI\text{-}Specific
+
Contextual
+
Sample\text{-}wise
+
Policy\text{-}Aware
+
Preference\ RL
}
\]

更具体：

> **使用少量 Human Preference 显式学习同一个 AI evaluator 在不同 trajectory/context 下与 Human Preference 一致的概率，并让该 sample-wise Human-Aligned Reliability 同时控制 preference weighting、Human escalation 和 downstream RL policy learning。**

---

## 6. 建议加入 Policy Stage

目前最值得进一步强化的方法变量：

\[
stage_i
\]

因为 RL 训练过程中：

\[
\pi_0
\rightarrow
\pi_1
\rightarrow
\cdots
\rightarrow
\pi_T
\]

trajectory distribution 会变化：

\[
d^{\pi_0}(s,a)
\neq
d^{\pi_T}(s,a).
\]

因此可能：

\[
P(
AI=Human
\mid
d^{\pi_0}
)
\neq
P(
AI=Human
\mid
d^{\pi_T}
).
\]

定义：

\[
\boxed{
Policy\text{-}Induced\ AI\ Feedback\ Reliability\ Shift
}
\]

这是当前非常值得验证的现象。

---

## 7. 当前推荐 reliability model

不再建议只做：

\[
\rho_i=g(c_i^{Jev}).
\]

建议：

\[
\boxed{
\rho_i
=
g_\phi(
c_i^J,
z(\tau_i^A,\tau_i^B),
d_i,
o_i,
t_i
)
}
\]

其中：

- \(c_i^J\)：Jev raw confidence；
- \(z(\tau_i^A,\tau_i^B)\)：trajectory/context representation；
- \(d_i\)：difficulty / similarity；
- \(o_i\)：OOD score；
- \(t_i\)：policy-training stage。

Human supervision：

\[
a_i
=
\mathbb{1}
[
y_i^J=y_i^H
].
\]

训练目标：

\[
\rho_i
=
P(a_i=1\mid x_i).
\]

---

## 8. \(\rho_i\) 应该控制三个环节

### 8.1 Preference Weighting

\[
\mathcal L_{RM}
=
\mathcal L_H
+
\lambda
\sum_i
\rho_i
\mathcal L_i^{AI}.
\]

### 8.2 Human Escalation

\[
\rho_i<\tau
\Rightarrow
Human / Strong\ Judge.
\]

### 8.3 Policy-Shift Recalibration

Static：

\[
D_H^0
\rightarrow
g_{\phi_0}
\]

Online：

\[
D_H^0
\rightarrow
g_{\phi_0}
\rightarrow
\pi_1
\rightarrow
D_H^1
\rightarrow
g_{\phi_1}
\rightarrow
\cdots
\]

目标是验证：

\[
ECE_{static}
\uparrow
\]

随着 policy shift 恶化，而：

\[
ECE_{online}
<
ECE_{static}
\]

且：

\[
PolicySuccess_{online}
>
PolicySuccess_{static}.
\]

---

## 9. 当前推荐研究问题

最终可收缩为：

> **When an RL policy continuously changes its trajectory distribution, does a fixed AI preference evaluator exhibit context-dependent reliability shift relative to human preferences, and can a very small amount of human feedback calibrate this sample-wise reliability sufficiently well to improve downstream preference-based RL under a fixed human budget?**

中文：

> **当 RL policy 持续改变 trajectory distribution 时，同一个 AI evaluator 对不同 trajectory pair 的 preference reliability 是否会产生 context-dependent / policy-induced shift？能否用极少量 Human Preference 在线校准这种 sample-wise Human-Aligned Reliability，并在固定 Human budget 下提高最终 Preference RL policy？**

---

## 10. 第一阶段 PoC

先不要继续无限扩展文献。

第一阶段只验证现象：

收集：

\[
200\sim500
\]

个 MetaWorld trajectory pairs。

同时获得：

\[
y_i^{Human},
\quad
y_i^{Jev},
\quad
c_i^{Jev}.
\]

至少覆盖：

- random policy；
- early-stage policy；
- middle-stage policy；
- near-converged policy。

先看：

### Metric A

\[
Accuracy(Jev,Human)
\]

### Metric B

\[
ECE(c_{Jev},Jev=Human)
\]

### Metric C

\[
Brier
\]

### Metric D

\[
P(
Jev=Human
\mid
policy\ stage
)
\]

### Metric E

\[
P(
Jev=Human
\mid
trajectory\ difficulty
)
\]

如果发现：

1. Jev preference 明显优于随机；
2. raw confidence 不是完美 calibrated；
3. Human calibration 可以改善 agreement prediction；
4. agreement/reliability 随 policy stage 或 context 系统变化；

则这个课题的核心现象成立。

---

## 11. 当前研究判断

| 维度 | 当前判断 |
|---|---:|
| 大方向新颖性 | 6/10 |
| 精确问题新颖性 | 8–8.5/10 |
| 撞题风险 | 8/10 |
| PoC 实现难度 | 6/10 |
| 完整论文难度 | 8/10 |
| 真机扩展难度 | 8.5–9/10 |
| 是否需要训练大模型 | 否 |
| 前期算力需求 | 中低 |
| Preference RL 结合度 | 很高 |
| Robot RL 结合度 | 很高 |
| 当前是否值得做 PoC | 是 |

注意：

> **8–8.5/10 不是“确定首次”，而是本轮 70 篇 Deep Search 没有发现完整重合工作后的研究空间判断。**

---

## 12. 当前最安全的论文表述

不建议：

> We introduce sample-wise preference reliability.

不建议：

> We calibrate AI judges with human feedback.

不建议：

> We use confidence to weight AI preferences.

不建议：

> We query humans when AI is uncertain.

建议：

> **We study human-anchored, context-dependent reliability of AI-generated trajectory preferences under policy-induced distribution shift, and use calibrated sample-wise AI–human agreement to jointly control preference weighting, selective human escalation, and downstream preference-based policy learning.**

进一步：

> **Unlike source-wise trust models, disagreement-based filtering, and evaluation-only judge calibration, our method estimates the probability that the same AI evaluator agrees with human trajectory preferences on each individual comparison and operationalizes that probability throughout reward learning and policy optimization.**

---

## 13. 结论

当前调研阶段已经足够支持进入 PoC。

领域并不是空白，而是：

> **2025–2026 正快速从“AI 能否提供 Preference/Reward”转向“AI feedback 在什么条件下值得相信”。**

目前最值得推进的课题主线：

> **Human-Anchored Contextual Reliability for AI-Assisted Preference-Based Reinforcement Learning**

Jev 在其中作为：

> **fast probabilistic AI preference evaluator backend**

而不是论文方法本身。

下一阶段重点不是继续无限搜文献，而是实验验证：

\[
\boxed{
Jev\ confidence
\neq
Human\text{-}Aligned\ Reliability
}
\]

以及：

\[
\boxed{
Human\text{-}Anchored\ Recalibration
\Rightarrow
Better\ Preference\ Learning
\Rightarrow
Better\ Policy
}
\]
