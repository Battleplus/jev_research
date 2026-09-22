# 03. Method v1：Calibrated Semantic Feedback RL

## 1. 问题定义

给定 MDP：

\[
\mathcal{M}=(\mathcal{S},\mathcal{A},P,r^{env},\gamma)
\]

外部 semantic evaluator：

\[
E(\tau_t)
\rightarrow
(q_t,c_t,z_t)
\]

其中：

- \(q_t\)：semantic score / progress；
- \(c_t\)：raw confidence；
- \(z_t\)：可选 category，例如 safe / risky / violation。

由于 \(c_t\) 不一定是真实概率，需要 calibration：

\[
c_t
\rightarrow
\hat c_t
\]

目标是让：

\[
\hat c_t
\approx
P(E(\tau_t)\text{ is correct}\mid\tau_t)
\]

## 2. Trust Weight

定义：

\[
w_t
=
g(
\hat c_t,
u_t^{OOD},
u_t^{disagreement},
b_t
)
\]

其中：

- \(\hat c_t\)：calibrated confidence；
- \(u_t^{OOD}\)：OOD / epistemic uncertainty；
- \(u_t^{disagreement}\)：multi-evaluator disagreement；
- \(b_t\)：query / cost budget。

## 3. Reward-level 版本

\[
r_t^{train}
=
r_t^{env}
+
\lambda w_t r_t^{sem}
\]

baseline：

- \(w_t=1\)：raw semantic reward；
- \(w_t=\mathbb{1}[\hat c_t>\tau]\)：hard gate；
- \(w_t=\hat c_t\)：continuous confidence weight；
- learned \(g(\cdot)\)：完整方法。

## 4. Advantage-level 版本

优先考虑：

\[
\tilde A_t
=
A_t^{env}
+
\lambda w_tA_t^{sem}
\]

相比直接改 reward，这个版本更容易把“信任”明确作用到 policy update。

## 5. Two-tier Evaluator

\`\`\`text
trajectory
    │
    ↓
Fast Evaluator
(Jev / local RM)
    │
 calibrated confidence
 ┌──┴───────────────┐
 ↓                  ↓
high confidence   low confidence
 ↓                  ↓
accept / weight   Strong Judge
                     │
                     ↓
               corrected feedback
\`\`\`

规则示例：

\[
\hat c_t>\tau_h
\Rightarrow
\text{accept}
\]

\[
\tau_l<\hat c_t\le\tau_h
\Rightarrow
\text{down-weight}
\]

\[
\hat c_t\le\tau_l
\Rightarrow
\text{escalate}
\]

## 6. Jev 闭源调用方案

### 方案 A：直接 sparse querying

不按 environment step 调用，而是：

- 每 trajectory；
- 每 subgoal boundary；
- 每 \(K\) steps；
- 只在 uncertainty 高时；

调用 Jev。

### 方案 B：Teacher → Local Evaluator

数据：

\[
D_J
=
\{(\tau_i,q_i,c_i)\}
\]

训练：

\[
f_\phi(\tau)
\rightarrow
(\hat q,\hat c)
\]

RL inner-loop 使用 \(f_\phi\)。

注意：

> 是否允许使用 Jev API 输出训练 surrogate model，需要先确认 TypeSafe 服务条款。

### 方案 C：缓存

每个 evaluator request 保存：

- model version；
- timestamp；
- exact prompt；
- structured state；
- raw response；
- probability；
- latency；
- token / cost；

实验复现优先 replay cached evaluator output。

## 7. Reproducibility

Jev 是闭源 API，因此论文方法不能依赖单一 vendor。

至少加入：

1. Jev
2. open/local evaluator
3. ordinary LLM / VLM judge
4. oracle / ground-truth evaluator

最终证明：

> CSF-RL 是 evaluator-agnostic，而 Jev 只是一个有代表性的 fast calibrated evaluator。
