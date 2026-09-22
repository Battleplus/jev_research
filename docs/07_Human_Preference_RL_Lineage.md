# 07. 与人类偏好强化学习（PbRL / RLHF）的关系

## 0. 一句话定位

当前这个课题如果要和“人类偏好强化学习”真正连起来，最准确的定位不是：

> **Jev 给 RL 一个额外 reward。**

而应该是：

> **沿着 Christiano-style preference learning 的路线，用 Jev 作为可扩展的 AI preference evaluator，但保留少量 human preference 作为目标锚点和校准依据；再根据 Jev 与人类偏好的一致性，动态决定 AI feedback 应该被信任多少。**

也就是说，真正的研究主线可以从：

\`\`\`text
Human Preference → Reward Model → RL Policy
\`\`\`

扩展成：

\`\`\`text
                    Human Preference
                    (anchor / target)
                           │
                           ↓
trajectory pair → Jev preference + confidence
                           │
                    calibration / trust
                           │
                           ↓
                     Reward Model
                           │
                           ↓
                       PPO / SAC
\`\`\`

人类偏好不是被 Jev “替代掉”，而是从**高频在线标注者**变成：

1. **目标定义者（normative target）**
2. **校准锚点（calibration anchor）**
3. **低置信/高风险样本的仲裁者（escalation source）**
4. **最终 policy 的独立评估者**

---

# 1. 与 Christiano et al.《Deep Reinforcement Learning from Human Preferences》的直接关系

Christiano et al. 的核心思想不是“人给 reward”，而是：

> **人只需要比较两段行为哪个更好，再从这些 pairwise preferences 学一个 reward predictor，最后让 RL 去优化这个 learned reward。**

其基本流程：

\`\`\`text
Policy rollout
    │
    ↓
trajectory segments σA, σB
    │
    ↓
Human comparison
A ≻ B / B ≻ A / tie / incomparable
    │
    ↓
Reward predictor r̂ψ
    │
    ↓
predicted reward
    │
    ↓
TRPO / A2C
    │
    ↓
updated policy
\`\`\`

reward model 的 preference probability 可以写成 Bradley–Terry 形式：

\[
P_\psi(\sigma^A \succ \sigma^B)
=
\frac{
\exp\left(\sum_t \hat r_\psi(s_t^A,a_t^A)\right)
}{
\exp\left(\sum_t \hat r_\psi(s_t^A,a_t^A)\right)
+
\exp\left(\sum_t \hat r_\psi(s_t^B,a_t^B)\right)
}
\]

然后用人类 preference label 训练 reward model。

## 我们继承了什么？

我们想保留 Christiano 的三个核心结构：

### ① 不直接手写复杂 reward

当“行为好不好”容易判断、但 reward 很难数学表达时，用 preference 作为监督。

### ② preference learning 与 policy learning 分离

- evaluator / human 负责提供“哪个更好”
- reward model 负责压缩这些偏好
- RL 负责优化 policy

### ③ feedback 应该跟着 policy 在线更新

Christiano 的重要观察之一是：

> 静态 reward model 会被 policy 找到漏洞。

因此需要随着 agent 访问新状态，继续收集新的 preference feedback。

这和我们现在担心的 **Jev reward hacking / OOD** 是同一个基本问题。

---

# 2. 我们和 Christiano 的关键差别

Christiano：

\[
Human
\rightarrow
y_H(\sigma^A,\sigma^B)
\]

我们的设想：

\[
Jev
\rightarrow
(p_J, c_J)
\]

其中：

- \(p_J\)：Jev 对 A/B preference 的概率或选择
- \(c_J\)：Jev 原始 confidence

但这里有一个非常重要的问题：

> **Jev 的 confidence 不等于“它和人类偏好一致的概率”。**

因此不能直接认为：

\[
c_J = P(y_J=y_H)
\]

我们真正需要学习的是：

\[
\rho
=
P(
y_J = y_H
\mid
x,\,
c_J,\,
\mathrm{OOD},\,
\mathrm{context}
)
\]

这里的 \(\rho\) 才是我们想要的 **trust / reliability**。

所以相比 Christiano：

\`\`\`text
Christiano:
Human preference
      ↓
Reward Model
      ↓
Policy

Our idea:
Human preference ─────────────┐
(calibration anchor)          │
                              ↓
Trajectory → Jev → preference/confidence
                              │
                        Reliability ρ
                              │
                              ↓
                         Reward Model
                              │
                              ↓
                           Policy
\`\`\`

---

# 3. 与 Kaufmann et al.《A Survey of Reinforcement Learning from Human Feedback》的关系

Kaufmann et al. 给出了一个很重要的概念澄清：

> **RLHF 可以看成 PbRL（Preference-based RL）的广义扩展。**

## 3.1 PbRL 是什么？

PbRL 主要关注：

- binary trajectory comparisons
- trajectory rankings
- state preferences
- action preferences

即：

> 不要求人给绝对 reward，只要求人表达“更喜欢哪个”。

所以 Christiano 的方法严格来说属于 preference-based RL 的经典范式，同时又被广泛视为现代 RLHF 的奠基工作。

---

## 3.2 RLHF 比 PbRL 更广

Kaufmann 的 taxonomy 中：

### PbRL 典型 feedback
- pairwise preference
- ranking
- state preference
- action preference

### 更广义 RLHF 还包括
- scalar feedback
- critique
- corrections
- action advice
- implicit feedback
- natural language

这意味着 Jev 可以有两种研究定位。

### 路线 A：Preference-based（更推荐）

Jev 只回答：

\[
\sigma^A \succ \sigma^B?
\]

例如：

\`\`\`text
Which trajectory better matches the intended task?
A or B?
confidence?
\`\`\`

这样和 Christiano/PbRL 的关系最干净。

### 路线 B：General semantic feedback

Jev 输出：

- progress
- safety
- preference
- task completion
- confidence

这已经超出了最窄意义的 PbRL，更接近广义 RLHF / semantic-feedback RL。

---

# 4. 为什么我现在更推荐“Preference 作为主线”

如果直接做：

\[
Jev(state)\rightarrow scalar\ reward
\]

那么更像：

> Foundation model reward shaping

和“人类偏好强化学习”的关系反而会变弱。

如果改成：

\[
(\sigma^A,\sigma^B)
\rightarrow Jev
\rightarrow
P(\sigma^A\succ\sigma^B)
\]

再用少量 human preference 校准 Jev，则整个故事会非常清楚：

> **Christiano 是 human preference → reward model；  
> RLAIF 是 AI preference → reward model；  
> 我们研究的是 human-anchored calibrated AI preference → robust reward model / policy learning。**

这个研究谱系更完整。

---

# 5. 与 Zhong et al.《A Comprehensive Survey of Reward Models》的关系

Zhong et al. 把 Reward Model 体系拆成三个关键问题：

\[
\text{Preference Collection}
\rightarrow
\text{Reward Modeling}
\rightarrow
\text{Usage}
\]

这套 taxonomy 很适合定位我们的工作。

---

## 5.1 Preference Collection：Human → AI

Zhong 将 preference source 分成：

### Human Preference

\[
Human
\rightarrow
preference\ labels
\]

这是 Christiano-style RLHF/PbRL。

### AI Preference

\[
LLM / AI
\rightarrow
preference\ labels
\]

这就是 RLAIF。

所以如果 Jev 给 trajectory pair 做 preference 判断：

\[
Jev
\rightarrow
y_J
\]

那么它首先属于：

> **AI Preference Collection**

而不是一个全新的 RL 范式。

---

## 5.2 Reward Modeling：Jev 可以处于两个位置

### 模式 1：Jev 只是 Labeler

\`\`\`text
trajectory pair
      ↓
     Jev
      ↓
AI preference labels
      ↓
Local Reward Model
      ↓
RL Policy
\`\`\`

这个对应：

> AI preference + explicit reward model

也是最接近 Christiano/RLAIF 的版本。

---

### 模式 2：Jev 本身就是 Generative Reward Model

\`\`\`text
trajectory
    ↓
   Jev
    ↓
score / choice / probability
    ↓
PPO/SAC
\`\`\`

这更接近：

> generative RM / LLM-as-a-judge / direct AI reward

这个路线工程上简单，但和已有 Direct-RLAIF / VLM reward work 重合更大。

---

# 6. 三篇论文串起来后的研究谱系

## Stage 1 — Christiano：Human Preference → Reward Model

\[
Human
\rightarrow
Preference
\rightarrow
Reward\ Model
\rightarrow
RL
\]

核心贡献：

> 用“人能判断好坏”代替“人必须写出 reward”。

---

## Stage 2 — Kaufmann Survey：把它放进 PbRL / RLHF 大框架

\[
PbRL
\subset
RLHF
\]

并指出：

- preference 只是 human feedback 的一种形式
- reward learning 与 policy learning 可以解耦
- human feedback 本身有 noise、misspecification、distribution shift
- learned reward 可能产生 reward hacking

---

## Stage 3 — Zhong Survey：Preference Source 从 Human 扩展到 AI

\[
Human\ Preference
\rightarrow
AI\ Preference
\]

并形成：

\[
AI\ Preference
\rightarrow
Reward\ Model
\rightarrow
Policy
\]

或者：

\[
AI\ Judge
\rightarrow
Direct\ Reward
\rightarrow
Policy
\]

---

## Stage 4 — 我们真正可能推进的部分

不是：

\[
Human
\rightarrow
Jev
\]

也不是简单：

\[
Jev
\rightarrow
Reward
\]

而是：

\[
\boxed{
Human\ Preference
\rightarrow
Calibration
\rightarrow
Jev\ Reliability
\rightarrow
Preference/Reward\ Learning
\rightarrow
Policy
}
\]

换句话说：

> **我们关注的不是 feedback source 从 Human 换成 AI，而是 Human 与 AI feedback 的“信任分配”。**

---

# 7. 进一步文献检索得到的“中间桥梁”

## 7.1 RLAIF：AI 直接替换 Human Labeler

RLAIF 保留了 Christiano-style pipeline：

\`\`\`text
Christiano:
Human → preference → RM → Policy

RLAIF:
AI → preference → RM → Policy
\`\`\`

所以：

> “让 Jev 给 preference label”本身属于 RLAIF 思路延伸，不足以单独作为 novelty。

---

## 7.2 Direct-RLAIF：甚至可以删掉 Reward Model

\`\`\`text
AI Judge
    ↓
direct reward
    ↓
Policy Optimization
\`\`\`

因此：

> “Jev 直接给 PPO reward”同样已经有很明确的先例。

---

## 7.3 RL-VLM-F：把 RLAIF 思路带到机器人 RL

RL-VLM-F：

\[
VLM
\rightarrow
pairwise\ visual\ preference
\rightarrow
Reward\ Model
\rightarrow
SAC
\]

而且 VLM 可以输出 “unsure”，这些样本直接不用于 reward model。

因此它几乎就是：

> **机器人版本的 AI-preference PbRL。**

这篇是我们必须重点对比的 baseline。

---

## 7.4 RIME：Preference 本身可能是错的

RIME 已经研究：

> noisy preference labels 如何污染 reward model。

它使用 denoising mechanism 对可疑 preference 进行筛选/纠正。

所以我们的创新不能写成：

> “AI preference 可能不可靠，因此我们过滤错误 preference。”

这个已经有人做。

---

## 7.5 Hybrid Preferences：Human 和 AI 怎么分工也有人做

HYPER 已经研究：

> 哪些 preference 让 AI 标，哪些送给 human。

因此：

> “低 confidence 就问 human”本身也不能算完整 novelty。

---

## 7.6 Multi-source Imperfect Preferences

2026 年进一步出现了：

> multiple humans / AI models / heuristics 都是 imperfect preference sources。

理论工作已经说明：

- 多 source 可以降低 variance
- 但 source bias 不会因为简单平均而消失
- naïve pooling 可能有明显问题

这和我们很相关：

\[
Human + Jev + Strong\ Judge
\]

不能简单：

\[
y = \text{majority vote}
\]

而应该估计每个 source 在当前样本上的 reliability。

---

# 8. 因此，最建议的论文方法改成：Human-Anchored Calibrated AI Preference RL

目前可以把 CSF-RL 进一步具体化为一个 preference 版本。

## 8.1 两类 preference 数据

少量 human preference：

\[
D_H
=
\{
(\sigma_i^A,\sigma_i^B,y_i^H)
\}
\]

大规模 Jev preference：

\[
D_J
=
\{
(\sigma_j^A,\sigma_j^B,p_j^J,c_j^J)
\}
\]

其中：

- \(p_j^J\)：Jev preference distribution
- \(c_j^J\)：Jev raw confidence

---

## 8.2 用 Human Preference 校准 Jev

学习 reliability：

\[
\rho_j
=
g_\phi(
p_j^J,
c_j^J,
x_j
)
\]

目标：

\[
\rho_j
\approx
P(
y_j^J=y_j^H
\mid x_j
)
\]

重点：

> 我们不是校准“Jev 自己觉得多有把握”，而是校准“Jev 与人类目标一致的概率”。

这是和 Human Preference 真正建立联系的地方。

---

# 9. Reward Model 怎么训练

定义 reward model preference：

\[
P_\psi(\sigma^A\succ\sigma^B)
\]

建议 loss：

\[
\mathcal L_{RM}
=
\sum_{i\in D_H}
CE(P_\psi,y_i^H)
+
\alpha
\sum_{j\in D_J}
\rho_j
CE(P_\psi,y_j^J)
\]

解释：

### Human label

\[
weight=1
\]

作为 anchor。

### Jev label

\[
weight=\rho_j
\]

Jev 越可能符合 human preference，训练权重越高。

---

# 10. 低可信 Jev Preference 怎么处理

三种动作：

\[
\rho>\tau_h
\Rightarrow
\text{直接使用}
\]

\[
\tau_l<\rho\le\tau_h
\Rightarrow
\text{低权重使用}
\]

\[
\rho\le\tau_l
\Rightarrow
\text{升级到 human / stronger judge}
\]

因此：

\`\`\`text
trajectory pair
      │
      ↓
     Jev
      │
 preference + confidence
      │
      ↓
human-calibrated reliability ρ
   ┌────────┼─────────┐
   ↓        ↓         ↓
 trust    downweight  escalate
                       ↓
                 Human / Strong Judge
   └────────┬─────────┘
            ↓
        Reward Model
            ↓
         PPO / SAC
\`\`\`

---

# 11. Policy 端还有第二层 Trust Allocation

除了在 RM training 时对 preference 加权，还可以在 policy update 再做一次：

\[
\tilde A_t
=
A_t^{env}
+
\lambda w_tA_t^{RM}
\]

这样形成两个层次：

### Feedback-level reliability

决定：

> 这条 Jev preference 能不能拿去训练 RM？

### Policy-level reliability

决定：

> 这个 learned semantic reward 对当前 policy update 应该有多大影响？

这比单纯 confidence gate 更完整。

---

# 12. Human Preference 在最终论文中必须承担什么角色？

如果最终论文想声称自己和 RLHF / PbRL 有明确关系，不能完全没有 human preference。

建议至少加入以下实验。

## Experiment A：Human vs Jev Preference Agreement

收集一小批 trajectory pairs，同时获得：

\[
y_H,\quad y_J,\quad c_J
\]

测：

- agreement
- calibration
- ECE
- Brier
- disagreement by task difficulty

---

## Experiment B：四种 Feedback Source

比较：

1. Human only
2. Jev only
3. Human + Jev naïve pooling
4. Human-anchored calibrated Jev

核心问题：

> calibrated hybrid 是否能用更少 human feedback 达到接近 Human-only 的 policy quality？

---

## Experiment C：Human Budget Curve

控制 human query budget，例如从低到高逐步增加。

观察：

\[
Human\ Query\ Budget
\quad vs \quad
Policy\ Success
\]

真正好的结果应该是：

> 少量 human anchor + 大量 calibrated Jev feedback  
> 能接近甚至达到高 human-label budget 的效果。

---

## Experiment D：Policy-induced Distribution Shift

随着 policy 越来越强：

\[
D_0\rightarrow D_1\rightarrow D_2
\]

检查：

\[
P(y_J=y_H)
\]

是否发生变化。

这直接回应 Christiano 强调的：

> feedback 必须跟着 policy distribution 在线更新，否则 reward model 会被 exploit。

---

# 13. 这个版本的 Novelty 边界

## 已经有人做

- Human preference → RM → RL
- AI preference → RM → RL
- AI direct reward → RL
- noisy preference denoising
- human/AI routing
- reward uncertainty
- multi-source preference

## 我们真正需要争取的创新

可以定义为：

> **在 policy-induced distribution shift 下，以 human preference 为 anchor，对 fast AI evaluator 的 sample-wise reliability 进行 calibration，并联合控制 preference weighting、source escalation 和 policy update。**

如果再放进 robot RL：

> **Human-Anchored Calibrated AI Feedback for Robot Preference Reinforcement Learning**

这个故事比“Jev 辅助 PPO”明显更完整。

---

# 14. Jev 在这个故事里的最佳角色

不建议：

> Jev = 新 reward model

更建议：

> **Jev = scalable AI preference annotator / fast evaluator**

原因：

1. 和 Christiano lineage 最清楚；
2. 和 RLAIF / RL-VLM-F 可以直接对比；
3. Human Preference 可以作为真正的目标锚点；
4. Jev 不开源的问题影响更小——它只负责提供 labels；
5. 后续可以训练 local RM，RL 内循环不依赖 Jev API；
6. Jev 的 probability/confidence 正好可以用于研究 calibration。

---

# 15. 当前建议的研究表述

### 不推荐

> 用 Jev 给强化学习提供 reward。

### 更推荐

> **研究如何利用少量人类偏好校准可扩展 AI evaluator（以 Jev 为代表）的偏好反馈，并根据样本级可信度动态分配 AI feedback 对 reward learning 和 policy optimization 的影响。**

或者更简洁：

> **Human-Anchored Calibrated AI Feedback for Preference-Based Reinforcement Learning**

---

# 16. 三篇核心论文与本课题的最终对应关系

| 核心论文 | 它解决了什么 | 我们继承什么 | 我们往前推进什么 |
|---|---|---|---|
| Christiano et al. | Human pairwise preference → learned reward → RL | pairwise trajectory preference、reward model、online feedback | Human 不再承担所有标注；用 Jev 扩展，并显式建模 trust |
| Kaufmann et al. RLHF Survey | 统一 PbRL / RLHF taxonomy，梳理 feedback、RM、policy learning | 把课题放进 PbRL/RLHF 框架 | 从 human-only feedback 扩展到 human-anchored AI feedback |
| Zhong et al. RM Survey | Human/AI preference、RM 类型、usage taxonomy | Jev 属于 AI preference / generative evaluator | 研究 AI preference 与 human objective 的 calibration，以及其对 policy update 的影响 |

---

# 17. 目前最核心的研究假设

\[
\boxed{
\text{Small Human Preference Anchor}
+
\text{Large AI Preference Feedback}
+
\text{Calibrated Trust Allocation}
>
\text{Raw AI Feedback}
}
\]

更具体：

> 当 AI evaluator 的 preference 在不同状态、任务难度和 policy distribution 下具有非均匀可靠性时，使用少量 human preference 学习 sample-wise reliability，并将其用于 preference weighting、query escalation 和 policy update，可以比 raw RLAIF、hard filtering 和 naïve human-AI pooling 获得更稳定、更接近人类目标的 RL policy。

---

# 18. 需要学长帮忙重点判断

1. 是否应该把课题主线从“semantic reward”进一步收缩到 **Preference RL**？
2. **Human-Anchored AI Preference** 这个定位是否比单纯 Jev reward 更容易形成论文？
3. sample-wise reliability 同时作用在 RM training + policy update，创新是否足够？
4. 是否值得把 human query efficiency 作为核心指标？
5. 最终应该更偏：
   - Preference RL
   - Safe RL
   - Robot RL
   - 还是三者结合中的一个主线？
