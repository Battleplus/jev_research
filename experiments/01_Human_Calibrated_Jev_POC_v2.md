# 01. Human-Calibrated Jev Preference RL — PoC v2

## 0. PoC 目标

第一阶段只回答三个问题：

\[
Q_1:
c_{Jev}
\text{ 是否能预测 }
Jev\text{-Human agreement?}
\]

\[
Q_2:
Human\text{-}calibration
\text{ 是否能显著改善 reliability estimation?}
\]

\[
Q_3:
更好的 reliability
\text{ 是否最终改善 RL policy?}
\]

如果前三问不能形成清晰正结果，不建议继续投入真机。

---

## 1. Environment

优先：

- MetaWorld Drawer Open
- MetaWorld Button Press

第二批：

- Pick Place
- Sweep Into

第一阶段不使用 RGB。

使用 simulator structured state。

---

## 2. Trajectory Pair Dataset

从三种 policy 收集轨迹：

1. Random policy
2. Partially trained policy
3. Near-converged policy

这样可以覆盖：

- obvious easy pairs
- ambiguous pairs
- near-optimal pairs
- failure vs failure
- success vs near-success

每个 pair：

\[
x_i=(\sigma_i^A,\sigma_i^B)
\]

记录 simulator ground truth progress，但 ground truth 不直接替代 Human Preference。

---

## 3. Human Preference

第一版建议：

\[
N_H=200\sim500
\]

pairwise comparisons。

标签：

- A better
- B better
- Tie / indistinguishable

保留：

- annotator ID
- response time
- optional confidence

如果资源不足，先单 annotator 做工程验证，正式实验再多人交叉标注。

---

## 4. Jev Preference

对同样的 trajectory pairs 调 Jev：

\[
(p_i^J,c_i^J)
\]

至少缓存：

- exact input
- model/version
- timestamp
- response
- probabilities
- confidence
- latency
- token/cost

部署阶段不调用 Jev。

---

## 5. Baseline 0：Raw Jev Agreement

直接看：

\[
P(y_J=y_H)
\]

并画：

\[
c_J
\rightarrow
Human\ Agreement
\]

输出：

- agreement accuracy
- ECE
- Brier
- NLL
- reliability diagram

---

## 6. Baseline 1：Raw Confidence Weighting

\[
w_i=c_i^J
\]

reward-model loss：

\[
\mathcal L
=
\sum_i
w_i
CE(P_\psi,y_i^J)
\]

这是必须做的强 baseline。

---

## 7. Method：Human-Calibrated Reliability

最简单：

### Logistic

\[
\rho_i
=
\sigma(ac_i^J+b)
\]

### Isotonic

\[
\rho_i
=
f_{iso}(c_i^J)
\]

### Contextual

\[
\rho_i
=
g_\phi(
c_i^J,
\Delta progress,
task,
policy\ stage,
OOD
)
\]

核心目标：

\[
\rho_i
\approx
P(y_i^J=y_i^H)
\]

---

## 8. Reward Model

比较：

### Human-only

\[
D_H
\rightarrow
RM_H
\]

### Jev-only

\[
D_J
\rightarrow
RM_J
\]

### Naive Hybrid

\[
D_H\cup D_J
\rightarrow
RM_{naive}
\]

### Raw Confidence

\[
w_i=c_i^J
\]

### Human-Calibrated

\[
w_i=\rho_i
\]

---

## 9. Selective Escalation

固定 Human budget。

例如：

\[
B_H\in\{5\%,10\%,20\%\}
\]

比较 query 策略：

1. Random Human Query
2. Lowest Jev Confidence
3. Reward-model disagreement
4. Lowest Human-Calibrated Reliability

看谁能在相同 human budget 下得到更好的 RM 和 policy。

---

## 10. RL

第一版建议 SAC。

原因：

- PbRL 文献大量使用；
- 与 PEBBLE / RL-VLM-F / PrefVLM 更容易直接比较；
- replay buffer 便于 reward relabeling。

---

## 11. 最终 baseline

至少：

1. Environment sparse reward
2. Human-only PbRL
3. Jev-only PbRL
4. Human + Jev naive pooling
5. Raw Jev confidence weighting
6. PrefVLM/RIME-style filtering baseline
7. Human-calibrated reliability
8. Human-calibrated reliability + selective escalation

---

## 12. 最重要的图

### Figure A：Calibration

\[
Jev\ confidence
\quad vs\quad
Human\ agreement
\]

### Figure B：Human Budget Curve

横轴：

\[
Human\ Preference\ Budget
\]

纵轴：

\[
Policy\ Success
\]

### Figure C：Policy Learning Curve

不同 feedback strategy 下：

\[
Success\ vs\ Environment\ Steps
\]

### Figure D：Distribution Shift

随着 policy 从 early → mid → late：

\[
P(y_J=y_H)
\]

和 calibration error 是否漂移。

---

## 13. Go 条件

建议满足以下至少三条：

1. Jev preference 明显优于随机；
2. raw confidence 与 Human agreement 有可利用相关性；
3. post-hoc calibration 明显改善 ECE/Brier；
4. calibrated weighting 优于 raw confidence；
5. selective escalation 在同样 Human budget 下更有效；
6. 最终 RL policy 有稳定、多 seed 增益。

---

## 14. No-Go 条件

任一出现都需要慎重：

- Jev preference 与 Human agreement 接近随机；
- confidence 和 correctness 几乎无关；
- calibration 只改善 metric，不改善 RM；
- RM 改善不传递到 policy；
- PrefVLM-style baseline 已经完全覆盖增益；
- 2026 最近论文已完整覆盖同一方法链。

---

## 15. 第一阶段工程量

预计需要实现：

- MetaWorld data collector
- trajectory serializer
- Jev query/cache wrapper
- simple Human annotation UI / CSV workflow
- calibrator
- preference RM
- SAC integration
- evaluation scripts

不需要：

- 训练 Jev
- 训练大模型
- VLM
- 真机
- world model

因此第一阶段工程可控，研究风险高于工程风险。
