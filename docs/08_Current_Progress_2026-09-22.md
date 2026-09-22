# 08. Current Progress — 2026-09-22

## 1. 当前研究问题已经从“Jev 辅助 RL”收缩为更具体的问题

最初想法：

\[
\text{Jev} \rightarrow \text{reward} \rightarrow \text{PPO/SAC}
\]

经过多轮文献检索后，这一版本已经明确不够新。

当前更值得继续验证的方向是：

> **Human-Anchored Calibrated AI Preference for Preference-Based Reinforcement Learning**

即：

\[
\text{少量 Human Preference}
\rightarrow
\text{校准 AI/Jev 可靠性}
\rightarrow
\text{大规模 AI Preference}
\rightarrow
\text{Reward Model / Policy Learning}
\rightarrow
\text{Standalone Policy}
\]

Jev 最适合的角色不是最终决策器，也不是 RL policy，而是：

> **训练阶段的 scalable AI preference annotator / fast evaluator**

部署阶段：

\[
s_t \rightarrow \pi_\theta(a_t|s_t) \rightarrow a_t
\]

不再需要 Jev，也不再需要 Human。

---

## 2. 已确认“不再构成创新”的部分

### 2.1 AI evaluator 只在训练阶段当老师

这一模式已经明确存在。

代表工作：

- RL-VLM-F
- RLAIF / Direct-RLAIF
- LAPP
- Vision-Language Models are Zero-Shot Reward Models for RL
- Preference VLM

这些工作已经证明：

\[
AI\ feedback
\rightarrow
reward/preference learning
\rightarrow
RL
\rightarrow
standalone\ policy
\]

因此：

> **“训练时用 Jev，部署时不用 Jev”本身不是 novelty。**

---

### 2.2 AI Preference 替代 Human Preference

RLAIF 已经建立：

\[
Human\ Preference
\rightarrow
AI\ Preference
\]

作为可扩展反馈来源。

因此：

> **“Jev 代替人做 preference annotation”本身不新。**

---

### 2.3 少量 Human + 大量 AI Preference

Preference VLM / ROVED / Hybrid Preferences 等工作已经开始研究：

- AI 负责大量 preference；
- uncertain / difficult samples 交给 Human 或 Oracle；
- Human feedback 用于纠错、适应或 routing。

因此：

> **“AI 不确定时问人”本身也不够新。**

---

### 2.4 Confidence weighting / uncertainty weighting

2025–2026 已出现大量工作研究：

- noisy preference filtering；
- confidence-weighted preference optimization；
- reward-model uncertainty；
- uncertainty-aware PPO / DPO；
- strong-judge escalation；
- multi-source imperfect preferences。

因此：

> **“confidence 乘一个 loss / reward 权重”本身不能作为核心贡献。**

---

## 3. 当前真正保留下来的研究问题

现在最值得研究的是：

\[
\boxed{
\rho_i
=
P(
y_i^{AI}
=
y_i^{Human}
\mid
x_i,c_i^{AI}
)
}
\]

其中：

- \(x_i\)：trajectory pair / context；
- \(y_i^{AI}\)：Jev / AI preference；
- \(y_i^{Human}\)：human preference；
- \(c_i^{AI}\)：AI 自己输出的 confidence；
- \(\rho_i\)：真正需要学习的 Human-Aligned Reliability。

关键区别：

\[
c_i^{AI}
\neq
P(y_i^{AI}=y_i^{Human})
\]

也就是说：

> **AI 自己“有多自信”，不等于“它有多符合人的目标”。**

因此真正的研究问题是：

> 能否用少量 human preference，把 AI evaluator 的 confidence 转换成 sample-wise human-aligned trust，再让这个 trust 控制 preference learning、human escalation 和 policy learning？

---

## 4. 当前推荐方法框架

### 4.1 两类数据

少量 Human Preference：

\[
D_H
=
\{
(\sigma_i^A,\sigma_i^B,y_i^H)
\}
\]

大量 Jev Preference：

\[
D_J
=
\{
(\sigma_j^A,\sigma_j^B,p_j^J,c_j^J)
\}
\]

---

### 4.2 Human-Anchored Calibrator

学习：

\[
g_\phi(x_j,p_j^J,c_j^J)
\rightarrow
\rho_j
\]

目标：

\[
\rho_j
\approx
P(y_j^J=y_j^H)
\]

第一版可以从简单方法开始：

- Logistic calibration
- Temperature scaling
- Isotonic regression

后续再扩展：

- task difficulty
- OOD score
- disagreement
- context embedding
- policy stage

---

### 4.3 Reward Model Training

一个直接版本：

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

其中：

- Human label：高可信 anchor；
- Jev label：根据 \(\rho_j\) 动态加权。

---

### 4.4 Selective Escalation

\[
\rho_j>\tau_h
\Rightarrow
\text{直接使用}
\]

\[
\tau_l<\rho_j\le\tau_h
\Rightarrow
\text{低权重使用}
\]

\[
\rho_j\le\tau_l
\Rightarrow
\text{Human / Strong Judge}
\]

---

### 4.5 Policy Learning

第一阶段建议：

\[
Reward\ Model
\rightarrow
SAC
\]

先避免过早同时修改 RM 和 policy gradient。

后续如果基础版本成立，再研究：

\[
\tilde A_t
=
A_t^{env}
+
\lambda w_t A_t^{pref}
\]

让 trust 进一步作用到 policy update。

---

## 5. 当前最接近的 prior work

### RL-VLM-F
VLM preference → Reward Model → SAC → standalone policy。

结论：

> AI teacher-only pipeline 已经成立。

### LAPP
LLM 比较 structured robot trajectories → preference predictor → RL → standalone robot policy。

结论：

> “structured trajectory → LLM preference → robot RL”已经非常接近 Jev 用法。

### Preference VLM
VLM + minimal human feedback + noisy/uncertain sample handling → RM → SAC。

结论：

> AI +少量 Human + selective querying 已经存在，是当前最重要的 baseline 之一。

### ROVED
VLE/VLM + targeted Oracle feedback，减少 preference annotation 成本。

结论：

> “减少人工 query”已经是明确研究方向。

### Hybrid Preferences
学习哪些样本应该交给 Human，哪些交给 AI。

结论：

> routing 本身不是 novelty。

### RIME
处理 noisy preference labels，通过 reward-model disagreement / KL 等识别异常反馈。

结论：

> preference denoising 本身不是 novelty。

### Confidence-Weighted Preference Optimization
弱 AI 模型提供 preference + confidence，confidence 直接加权 preference loss。

结论：

> confidence-weighted policy alignment 已经存在。

### BACON
少量 Human calibration + 多 AI judges，估计 judge/sample reliability。

结论：

> Human-calibrated AI evaluation 已经出现，但主要用于 evaluation，而不是 robot preference RL policy learning。

### Conformal Feedback Alignment
用 calibration / conformal reliability 直接调节 PPO/DPO-style training。

结论：

> calibration → optimization weight 已经出现，但主要在 LLM alignment。

---

## 6. 当前 novelty 边界

### 已经有人做

- Human preference → RM → RL
- AI preference → RM → RL
- AI teacher only during training
- standalone policy at deployment
- AI +少量 Human
- uncertain sample → Human
- noisy preference filtering
- confidence weighting
- multi-source preference
- AI judge calibration

### 目前还值得争取的组合

> **在 robot/general Preference RL 中，以少量 Human Preference 作为目标锚点，显式估计 AI evaluator 与 Human Preference 的 sample-wise agreement probability，并让该 reliability 联合控制 AI preference weighting、Human escalation 和 downstream policy learning。**

进一步可加入：

- policy-induced distribution shift；
- OOD；
- reward hacking；
- human query budget curve；
- evaluator backend replacement。

---

## 7. 当前难度评估

| 部分 | 工程难度 | 研究难度 |
|---|---:|---:|
| Preference RL baseline | 4/10 | 4/10 |
| Jev API preference labeling | 3–4/10 | 3/10 |
| Human preference collection | 4/10 | 5/10 |
| Human-aligned calibration | 6/10 | 8/10 |
| RM weighted training | 5/10 | 7/10 |
| Selective Human escalation | 5/10 | 7/10 |
| 证明 calibration 改善最终 policy | 7/10 | 9/10 |
| 真机扩展 | 9/10 | 9/10 |

总体：

- PoC：约 **6/10**
- 完整论文：约 **8/10**
- 真机 + 强 robustness：约 **8.5–9/10**

---

## 8. 当前最重要的三个 Go / No-Go 问题

### Q1

\[
c_{Jev}
\stackrel{?}{\longrightarrow}
P(y_{Jev}=y_{Human})
\]

Jev confidence 是否对 human agreement 有预测价值？

### Q2

经过 calibration 后：

\[
ECE,\ Brier,\ NLL
\]

是否显著改善？

### Q3

calibration 的改善是否真正传导到：

\[
Preference\ Quality
\rightarrow
Reward\ Model
\rightarrow
Policy\ Success
\]

如果只改善 evaluator metric，而不改善最终 policy，则论文价值有限。

---

## 9. 第一阶段推荐实验

任务：

- MetaWorld Drawer Open
- Button Press
- Pick Place
- Sweep Into

比较：

1. Human Preference only
2. Jev only
3. Human + Jev naïve pooling
4. Raw Jev confidence weighting
5. PrefVLM-style filtering
6. Human-calibrated Jev
7. Human-calibrated Jev + selective escalation

重点横轴：

\[
Human\ Preference\ Budget
\]

例如：

\[
0\%,5\%,10\%,20\%,50\%,100\%
\]

重点纵轴：

\[
Policy\ Success\ Rate
\]

理想目标：

\[
10\%-20\%\ Human
+
80\%-90\%\ calibrated\ Jev
\approx
100\%\ Human
\]

---

## 10. 当前风险

### 撞题风险：高

2026 年相关主题明显快速增加：

- preference reliability
- multi-source imperfect preference
- human-AI calibration
- trust-aware reward modeling
- confidence-weighted preference optimization
- robot preference calibration

因此需要尽快完成 PoC，并持续追踪 2026 最新工作。

### Jev 闭源风险

Jev 应当只是 evaluator backend，不应成为算法不可替代部分。

至少需要：

- Jev
- 一个 open/local evaluator
- Human / Oracle
- 可选 strong judge

证明方法是 evaluator-agnostic。

---

## 11. 最新 Undermind 检索状态

已完成的深度检索：

1. **Calibrated decision feedback for reinforcement learning**
2. **Human preference RL to calibrated AI feedback**
3. **Training only AI preference teacher then standalone RL policy**

当前新检索：

4. **Human calibrated AI preference RL novelty and implementation difficulty**

本轮检索还发现以下非常接近的 2026 新工作，需要下一轮优先全文核验：

- Preference-Calibrated Human-in-the-Loop Reinforcement Learning for Robotic Manipulation
- TrustRoboReward: Preference-Ordered Isotonic Score Editing for Multi-Paradigm Robot Reward Models
- Finding the Signal in the Spam: Jointly Learning Rewards and Worker Reliability from Pairwise Comparisons
- Trust, Don't Trust, or Flip: Robust Preference-Based Reinforcement Learning with Multi-Expert Feedback

当前 Undermind 高成本调用额度暂时耗尽，工具返回短周期 reset 倒计时。待额度恢复后，下一步优先逐篇读取上述四篇全文，重新确认 novelty 边界。

---

## 12. 当前结论

这个领域不是“没人做”，而是：

> **2025–2026 正在从“AI 能否给 Preference/Reward”快速转向“AI Feedback 什么时候值得相信”。**

因此目前更准确的课题不是：

> Jev-Assisted RL

而是：

> **Human-Calibrated AI Preference Reinforcement Learning**

Jev 是其中一个非常适合测试的 fast probabilistic evaluator，而不是论文贡献本身。
