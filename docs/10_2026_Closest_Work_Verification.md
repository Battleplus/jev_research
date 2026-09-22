# 10. 2026 Closest-Work Verification

## 1. 结论

在 Undermind 配额恢复后，对此前最危险的 2026 近邻工作进行了全文核验。

当前方向 **尚未被完整覆盖**，但 novelty 必须进一步收缩。

当前最有防御力的核心，不是：

- AI feedback → preference RL
- training-only AI teacher
- hybrid Human + AI feedback
- confidence weighting
- uncertain sample escalation
- generic source reliability

而是：

\[
\rho(x)
=
P(
y_{AI}=y_{Human}
\mid
x,c_{AI}
)
\]

并在 **robot/general preference-based RL** 中，用少量 Human trajectory preferences 对 AI evaluator 的 **sample-wise / context-dependent human agreement probability** 做校准，再将该 reliability 用于：

1. AI preference weighting；
2. selective Human / Strong-Judge escalation；
3. downstream reward / policy learning；
4. policy-induced distribution shift 下的 re-calibration。

---

# 2. Preference-Calibrated Human-in-the-Loop RL for Robotic Manipulation

论文：**PACT: Preference-Calibrated Actor-Critic Training**

## 它的“Preference-Calibrated”到底是什么？

这里的 calibration 不是校准 AI evaluator。

它主要做的是：

> 利用 Human intervention 产生的 implicit preference，修正 Actor-Critic 的 credit assignment。

Human 一旦介入，就隐含表达：

\[
a^{human}
\succ
a^{policy}
\]

PACT 通过这种信号：

- 修正 critic Bellman target；
- 对导致干预的 policy action 降权/惩罚；
- 对 human corrective action 进行 actor preference optimization。

## 是否有 AI evaluator？

没有。

- 不使用 LLM/VLM judge；
- progress model 只是用于定位 suboptimal trajectory segment；
- progress model 不是 preference annotator。

## 部署

最终 policy 独立运行，无 inference-time overhead。

## 和我们的关系

重合度低。

PACT 校准的是：

\[
Actor/Critic
\]

我们想校准的是：

\[
P(
AI\ preference
=
Human\ preference
)
\]

所以标题虽然危险，但实际没有覆盖我们的核心问题。

---

# 3. TrustRoboReward

这是更危险的近邻。

## 核心方法

它研究：

> pointwise score 与 pairwise preference 之间的 cross-paradigm inconsistency。

例如：

\[
Score(A)>Score(B)
\]

但：

\[
B\succ A
\]

这就是 score-pair reversal。

它提出 POISE：

> Preference-Ordered Isotonic Score Editing

使用 pairwise preference order 作为偏序约束，通过 isotonic/PAVA 修正 pointwise score。

## Supervision

包括：

- Score-A：trajectory progress score
- Score-B：video-QA score
- Pair-A：trajectory pair preference
- Pair-B：video-QA pair preference

主要 teacher：

> GPT-5-mini

student reward model：

> Qwen3-VL 4B / 8B

另外，少量 expert human 用于解决 cycle / transitivity conflict。

## 是否做 Human-calibrated AI reliability？

没有。

它真正校准的是：

\[
pointwise\ score
\leftrightarrow
pairwise\ preference\ order
\]

而不是：

\[
P(y_{AI}=y_{Human}\mid x,c_{AI})
\]

它关注的是 **跨监督范式一致性**，不是 AI 对 Human 的 sample-wise agreement probability。

## 是否进入 RL？

会用于 downstream embodied policy optimization。

因此：

> Robot reward calibration 本身已经非常活跃。

我们必须明确区别：

> TrustRoboReward 校准 score consistency；我们校准 Human-aligned evaluator reliability。

---

# 4. Finding the Signal in the Spam

方法名：**BoRaEM**

## 核心思想

联合学习：

- item reward \(r_j\)
- worker reliability \(\beta_s\)

模型：

\[
P[w\succ l\mid s]
=
\sigma(
\beta_s(r_w-r_l)
)
\]

## Reliability

\[
\beta_s\in[-1,1]
\]

含义：

- \(\beta_s>0\)：可靠 worker
- \(\beta_s\approx0\)：random/spam
- \(\beta_s<0\)：adversarial，可翻转其 signal

## 最大区别

它是：

> **worker-level global reliability**

不是：

\[
\rho(x)
\]

这种 sample/context dependent reliability。

它还：

- 不需要 gold Human anchor；
- 不训练 downstream RL policy；
- 主要研究静态 pairwise comparison reward inference。

## 对我们的影响

它说明：

> “联合学习 reward + source reliability”本身已经不是创新。

我们必须进一步强调：

- AI evaluator-specific；
- Human anchor；
- contextual / sample-wise reliability；
- downstream preference RL；
- policy-induced shift。

---

# 5. Trust, Don't Trust, or Flip

方法：**TriTrust-PBRL / TTP**

这是当前 **Preference RL 侧最危险的近邻之一**。

## 方法

每个 expert 有 trust parameter：

\[
\alpha_k
\]

modified Bradley-Terry：

\[
P(
y_{ij}^{(k)}=1
)
=
\sigma(
\alpha_k
[
R(\tau_i)-R(\tau_j)
]
)
\]

含义：

\[
\alpha_k>0
\Rightarrow Trust
\]

\[
\alpha_k\approx0
\Rightarrow Ignore
\]

\[
\alpha_k<0
\Rightarrow Flip
\]

## 是否进入 RL？

是。

它基于：

- PEBBLE
- SAC

实验包括：

- MetaWorld Door-Open-v2
- MetaWorld Sweep-Into-v2
- DMControl Cheetah-Run
- DMControl Walker-Walk

最终训练 reward model 和 standalone policy。

## 它没有做什么？

非常关键：

### 1. Reliability 是 expert-level global scalar

\[
\alpha_k
\]

一个 expert 只有一个全局 trust。

不是：

\[
\rho_k(x)
\]

### 2. 没有 Human gold anchor

trust 从 preference 数据里联合估计。

### 3. 不做 context-dependent trust

论文自己将：

> context-dependent trust

列为未来工作。

## 对我们的影响

这篇反而帮我们把 novelty 说得更清楚：

> **不能再做“multiple sources + reliability”，必须做 Human-anchored, context-dependent, sample-wise AI trust。**

---

# 6. Trust or Escalate

**Trust or Escalate: LLM Judges with Provable Guarantees for Human Agreement**

这是 calibration 侧最危险的一篇。

## 它已经做了什么？

使用小规模 Human preference calibration set。

对每个样本：

\[
c_{LM}(x)
\]

估计 judge confidence。

再通过 calibration set 学 threshold：

\[
\lambda
\]

若：

\[
c_{LM}(x)\ge\lambda
\]

则使用 LLM judge。

否则：

\[
\text{abstain / escalate}
\]

还支持：

\[
weak\ judge
\rightarrow
stronger\ judge
\rightarrow
human
\]

的 cascade。

## Human Agreement Guarantee

它直接针对：

\[
P(
LLM\ agrees\ with\ Human
)
\]

提供 selective evaluation guarantee。

因此：

> **“用少量 Human 校准 AI judge，使其对 Human agreement 可控”已经有人做。**

## 但没有做 RL

这是关键空位。

论文只做：

> evaluation / judge selection

没有：

\[
calibrated\ preference
\rightarrow
RM
\rightarrow
RL
\rightarrow
policy
\]

因此我们不能再把“Human calibrates AI judge”本身当贡献，而必须证明：

> **这种 calibration 如何改变 preference learning 和最终 policy。**

---

# 7. Calibrate, Don't Curate

这是另一个非常危险的 calibration 近邻。

## 已经做

小 Human-labeled calibration set：

\[
D_{cal}
\]

对多个 noisy LLM judges：

- estimate judge accuracy；
- judge weighting；
- Platt scaling；
- Beta calibration；
- conformal coverage；
- 可加入 contextual feature \(x\)。

例如：

\[
\operatorname{logit}
(\hat p_t^{corr})
=
a\operatorname{logit}(\hat p_t)
+
x_t^\top\gamma
+b
\]

这已经接近：

> context-aware Human-calibrated AI judge.

## 但它不做什么？

- 不做 active Human routing；
- 不训练 robot RL；
- 不做 policy-induced distribution shift；
- 不验证 calibration 是否传导到 downstream policy。

因此我们的空间进一步收缩成：

> **dynamic preference RL use of Human-calibrated AI judge reliability。**

---

# 8. COACT

COACT 做：

- AI self-preference / self-consistency；
- uncertain / OOD sample → oracle；
- Human / stronger AI verification；
- modified DPO；
- iterative policy improvement。

但是：

- 不显式学习 \(P(AI=Human)\)；
- 不做 robot RL；
- 主要 reasoning/LLM alignment。

说明：

> Human-AI synergy + active routing + policy learning 也不能单独算 novelty。

---

# 9. 当前真正的 novelty

经过这一轮全文核验，最安全的版本已经进一步收缩：

## Human-Anchored Contextual Reliability for AI-Assisted Preference RL under Policy Shift

定义：

\[
\rho_i
=
P(
y_i^{AI}=y_i^{Human}
\mid
x_i,\,
c_i^{AI},\,
d_i^{OOD},\,
stage_i
)
\]

其中：

- \(x_i\)：trajectory/context；
- \(c_i^{AI}\)：AI raw confidence；
- \(d_i^{OOD}\)：distribution-shift / novelty signal；
- \(stage_i\)：policy learning stage。

核心贡献至少要包括：

### 1. Human agreement calibration

不是：

\[
AI\ confidence
\]

而是：

\[
AI\text{-}Human\ agreement\ probability
\]

### 2. Sample/context-dependent trust

不是：

\[
\alpha_k
\]

这种 source-level scalar。

而是：

\[
\rho_k(x)
\]

### 3. Policy-induced shift

随着：

\[
\pi_0\rightarrow\pi_1\rightarrow\pi_2
\]

trajectory distribution 改变时：

\[
P(y_{AI}=y_{Human})
\]

也可能漂移。

需要：

- monitor；
- recalibrate；
- compare static vs dynamic calibration。

### 4. Reliability 进入 downstream learning

至少同时验证：

\[
\rho
\rightarrow
Preference\ Weighting
\]

和：

\[
\rho
\rightarrow
Selective\ Human\ Query
\]

最终必须传导到：

\[
Reward\ Model
\rightarrow
Policy\ Success
\]

### 5. Standalone deployment

训练完：

\[
s\rightarrow\pi_\theta(s)\rightarrow a
\]

不再调用 AI evaluator / Human。

---

# 10. 当前最关键的 baseline

现在 baseline 必须升级。

至少：

1. Human-only PbRL
2. AI/Jev-only
3. Naive Human + AI pooling
4. Raw AI confidence weighting
5. RIME-style noisy preference filtering
6. PrefVLM / ROVED-style uncertainty routing
7. TriTrust-style source-level reliability
8. Human-calibrated static reliability
9. **Ours: Human-calibrated contextual/dynamic reliability**
10. Ours + selective Human escalation

否则很难证明真正增量。

---

# 11. 难度修正

经过近邻工作全文核验后：

| 项目 | 难度 |
|---|---:|
| 跑通 PoC | 6/10 |
| Preference RL 基线 | 4/10 |
| Jev preference pipeline | 4/10 |
| Human annotation | 5/10 |
| Static calibration | 5/10 |
| Contextual reliability | 7/10 |
| Policy-shift re-calibration | 8/10 |
| 强 baseline 对比 | 8/10 |
| 证明最终 policy 增益 | 9/10 |
| 真机完整论文 | 9/10 |

当前最大难点已经不是工程，而是：

> **如何证明这不是已有模块的简单拼接。**

---

# 12. 当前判断

这个方向仍然可以做，但不能再停留在：

> Human-calibrated Jev Preference RL

这么宽泛的表述。

更合适的核心问题是：

> **When and where should an AI preference evaluator be trusted as the policy distribution shifts?**

对应方法：

> **Human-Anchored Contextual Reliability Calibration for AI-Assisted Preference-Based Reinforcement Learning**

Jev 可以作为最重要的 evaluator case study，但论文应保持 evaluator-agnostic。
