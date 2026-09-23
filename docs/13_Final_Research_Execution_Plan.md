# 13. Final Research Execution Plan — Frozen AI Feedback Adaptation for PbRL

更新时间：2026-09-23  
状态：**当前执行版规划**

## 0. 研究主线

当前项目不再以“训练 Jev / 后训练闭源模型”为目标。

最终研究问题定义为：

> **当一个 frozen black-box AI teacher 面对不断变化的 on-policy trajectory distribution 时，它与 Human Preference 的 agreement 是否会发生 context-dependent / policy-induced shift？能否使用少量 Human Preference 学习一个外部 Human-Aligned Feedback Adapter，对 AI feedback 进行 sample-wise correction、reliability estimation 与 selective escalation，并最终提升 downstream Preference-Based RL policy？**

核心链路：

\[
\boxed{
Trajectory\ Pair
\rightarrow
Frozen\ AI\ Teacher
\rightarrow
Raw\ AI\ Feedback
\rightarrow
Human\text{-}Aligned\ Feedback\ Adapter
\rightarrow
Reward\ Model
\rightarrow
SAC/PPO
\rightarrow
Standalone\ Policy
}
\]

其中：

- AI teacher：冻结，不训练；
- Feedback Adapter：训练；
- Reward Model：训练；
- RL Policy：训练；
- Human Preference：作为少量 trusted anchor；
- 部署阶段不再调用 AI teacher / Human / Reward Model。

---

## 1. 最终方法定义

### 1.1 Frozen AI Teacher

输入：

\[
x_i=(\tau_i^A,\tau_i^B)
\]

AI teacher 输出：

\[
y_i^{AI},\quad c_i^{AI}
\]

其中：

- \(y_i^{AI}\)：A / B / Tie；
- \(c_i^{AI}\)：AI raw confidence / probability；
- AI teacher 可以是 Jev、GPT、VLM 或其他 black-box evaluator。

关键设定：

\[
\boxed{\theta_{AI}\ \text{fixed}}
\]

如果厂商没有 fine-tuning/post-training 接口，不假设可以修改其参数。

### 1.2 Human Anchor

对少量 trajectory pairs 获取：

\[
y_i^H
\]

形成 trusted set：

\[
D_H=
\{
(\tau_i^A,\tau_i^B,y_i^H,y_i^{AI},c_i^{AI})
\}
\]

Human 用于：

1. 定义 AI–Human agreement；
2. 学习 sample-wise reliability；
3. 学习 systematic feedback correction；
4. 在低可靠样本上 selective escalation；
5. 随 policy shift 周期性 recalibrate。

### 1.3 Human-Aligned Feedback Adapter

统一记为：

\[
F_\phi
\]

输入：

\[
F_\phi(
z(\tau_i^A,\tau_i^B),
y_i^{AI},
c_i^{AI},
d_i,
o_i,
t_i
)
\]

输出：

\[
(\hat y_i^H,\rho_i)
\]

其中：

\[
\hat y_i^H
=
P(y_i^H=A\mid x_i,AI\ feedback)
\]

\[
\rho_i
=
P(
y_i^{AI}=y_i^H
\mid
x_i,c_i^{AI},d_i,o_i,t_i
)
\]

变量含义：

- \(z(\tau_A,\tau_B)\)：trajectory-pair representation；
- \(d_i\)：difficulty / similarity；
- \(o_i\)：OOD score；
- \(t_i\)：policy stage；
- \(\hat y_i^H\)：Human-aligned corrected preference；
- \(\rho_i\)：sample-wise Human-aligned reliability。

---

## 2. 方法的三种作用方式

### 2.1 Preference Weighting

\[
\mathcal L_{RM}
=
\sum_{i\in D_H}
CE(P_\psi,y_i^H)
+
\lambda
\sum_{j\in D_{AI}}
\rho_j
CE(P_\psi,\hat y_j)
\]

比较：

- raw AI label；
- raw confidence weighting；
- calibrated reliability weighting；
- corrected label + reliability weighting。

### 2.2 Feedback Correction

不是只做：

\[
y_i^{AI}\times \rho_i
\]

而是允许：

\[
y_i^{AI}
\rightarrow
\hat y_i^H
\]

重点验证 AI 是否存在可学习的 systematic bias。

### 2.3 Selective Human Escalation

\[
\rho_i>\tau_h
\Rightarrow Accept
\]

\[
\tau_l<\rho_i\le\tau_h
\Rightarrow Correct/Downweight
\]

\[
\rho_i\le\tau_l
\Rightarrow Query\ Human
\]

所有 query strategy 必须在 **固定 Human budget** 下比较。

---

## 3. 核心科学假设

### H1：AI confidence 不等于 Human-aligned reliability

\[
c_i^{AI}
\neq
P(y_i^{AI}=y_i^H)
\]

### H2：AI reliability 是 context-dependent 的

\[
P(y^{AI}=y^H\mid x_1)
\neq
P(y^{AI}=y^H\mid x_2)
\]

重点关注 easy vs ambiguous、success vs near-success、similar trajectories、safety/efficiency trade-offs、OOD states。

### H3：AI–Human agreement 随 policy stage 漂移

\[
\boxed{
P(
y^{AI}=y^H
\mid
d^{\pi_0}
)
\neq
P(
y^{AI}=y^H
\mid
d^{\pi_T}
)
}
\]

定义为：

> **Policy-Induced AI–Human Agreement Shift**

### H4：Static calibration 会在 later-policy data 上退化

比较：

\[
F_{\phi_0}
\]

固定不变，与：

\[
F_{\phi_t}
\]

周期性更新。

### H5：Feedback improvement 必须传递到 policy

\[
\boxed{
Better\ Feedback
\rightarrow
Better\ Reward\ Model
\rightarrow
Better\ Policy
}
\]

如果只改善 calibration metric，而不改善 policy，则方法价值不足。

---

## 4. Phase 0 — 工程底座

目标：先把数据、环境、日志链路做对。

### 环境

第一优先：

- MetaWorld Drawer Open
- MetaWorld Button Press

第二批：

- Pick Place
- Sweep Into

算法：

- SAC 作为主 PbRL policy optimizer；
- PPO 作为可选 robustness / transfer baseline。

### Structured Trajectory

第一版不用 RGB。

保存：

- gripper position；
- object position；
- target position；
- contact；
- grasp；
- task progress；
- collision / safety；
- trajectory length；
- task-specific state。

同时保留 raw simulator state，避免后续重新采集。

建议目录：

\`data/raw/\`

\`data/trajectory_pairs/\`

\`data/human_labels/\`

\`data/ai_labels/\`

\`data/calibration/\`

\`results/\`

---

## 5. Phase 1 — 先验证核心现象，不训练完整方法

这是当前最重要阶段。

### 5.1 Policy Stage Collection

至少采集：

\[
\pi_{random},
\pi_{early},
\pi_{mid},
\pi_{late}
\]

建议：

- random：未训练；
- early：约 10–20% 训练进度；
- mid：约 40–60%；
- late：约 80–100%。

同时记录 policy success / return 水平。

### 5.2 Pair 数量

Pilot：

\[
N=200\sim400
\]

正式现象验证：

\[
N=400\sim800
\]

### 5.3 每个 pair 保存

\[
(
\tau_A,
\tau_B,
y_H,
y_{AI},
c_{AI},
task,
policy\ stage,
difficulty,
OOD,
metadata
)
\]

### 5.4 必做分析

1. Overall AI–Human Agreement；
2. Agreement by policy stage；
3. Agreement by pair difficulty；
4. raw confidence vs actual agreement；
5. ECE；
6. Brier；
7. NLL；
8. Risk-Coverage；
9. systematic error examples；
10. cross-stage generalization。

### Phase 1 Go 条件

至少出现一个核心现象：

- raw confidence miscalibrated；
- context-dependent bias；
- early→late agreement shift；
- static mapping 在 later stage 明显退化。

如果全部都不存在，则弱化 policy-shift 方向。

---

## 6. Phase 2 — Feedback Adapter PoC

从简单到复杂。

### Baseline A：Raw AI

\[
\hat y=y^{AI}
\]

### Baseline B：Global Calibration

只使用：

\[
c^{AI}
\]

候选：

- Logistic calibration；
- Isotonic regression；
- Temperature scaling（若概率结构允许）。

### Baseline C：Contextual Reliability

\[
\rho_i=
g_\phi(
c_i^{AI},
difficulty_i,
task_i,
OOD_i
)
\]

### Baseline D：Policy-Conditional Reliability

\[
\rho_i=
g_\phi(
c_i^{AI},
z_i,
difficulty_i,
OOD_i,
stage_i
)
\]

### Method E：Correction + Reliability

\[
F_\phi(x_i)
\rightarrow
(\hat y_i^H,\rho_i)
\]

模型先从轻量版本开始：

- Logistic / multinomial regression；
- shallow MLP；
- small Transformer 仅在必要时加入。

原则：

> 第一篇不要让 method complexity 盖过研究问题。

---

## 7. Phase 3 — Reward Model

建议使用 Bradley-Terry preference reward model。

比较：

1. Human-only RM；
2. AI-only RM；
3. Human + AI naïve pooling；
4. raw confidence weighted RM；
5. PrefVLM/RIME-style filtering；
6. source-wise trust baseline；
7. static Human-calibrated RM；
8. policy-conditional calibrated RM；
9. correction + reliability RM；
10. correction + reliability + selective Human escalation。

RM 指标：

- held-out Human preference accuracy；
- NLL；
- calibration；
- reward ranking quality；
- cross-stage performance；
- OOD performance。

---

## 8. Phase 4 — Preference-Based RL

主算法：

\[
Reward\ Model
\rightarrow
SAC
\]

必须报告：

- Success Rate；
- True Environment Return；
- Sample Efficiency；
- AUC of learning curve；
- mean ± CI；
- 至少 3 seeds，正式版本建议 5 seeds。

重点比较：

1. Environment sparse reward；
2. Human-only PbRL；
3. AI-only PbRL；
4. Naïve Hybrid；
5. Raw Confidence Weighting；
6. PrefVLM/RIME-style Filtering；
7. Source-wise Trust；
8. Static Human Calibration；
9. Policy-Conditional Reliability；
10. **Ours: Correction + Reliability + Escalation**。

---

## 9. Phase 5 — Human Budget Experiment

固定 Human budget：

\[
B_H\in
\{0,5\%,10\%,20\%,50\%,100\%\}
\]

比较 query strategy：

1. Random；
2. Lowest raw AI confidence；
3. RM↔AI disagreement；
4. Highest OOD；
5. Lowest calibrated \(\rho_i\)；
6. correction uncertainty；
7. mixed acquisition。

核心图：

\[
Human\ Budget
\rightarrow
Policy\ Success
\]

目标：

> 在相同 Human budget 下，当前方法是否能更有效分配 Human queries。

---

## 10. Phase 6 — Online Recalibration

流程：

\[
D_H^0
\rightarrow
F_{\phi_0}
\rightarrow
\pi_1
\rightarrow
D_H^1
\rightarrow
F_{\phi_1}
\rightarrow
\pi_2
\rightarrow
\cdots
\]

比较：

- Static Adapter；
- Periodic Recalibration；
- Triggered Recalibration。

Trigger 候选：

- ECE drift；
- OOD increase；
- disagreement increase；
- policy performance plateau；
- agreement-risk estimate deterioration。

必须控制 Human budget 一致。

---

## 11. Phase 7 — Robustness

### 11.1 Evaluator Noise

人工注入：

- 10%
- 20%
- 30%
- 40%

label corruption。

### 11.2 Systematic Bias

构造：

- 偏好更短 trajectory；
- 偏好更快 trajectory；
- 忽略 safety；
- 对 near-success 过度乐观；
- 对某类 task stage 系统性偏置。

这是验证 correction 模块最重要的实验。

### 11.3 OOD

- object position shift；
- initial state shift；
- target shift；
- task variation；
- unseen policy behavior。

### 11.4 Paraphrase

同一 trajectory pair 使用多个语义等价 instructions。

报告：

- Flip Rate；
- confidence variance；
- correction robustness。

### 11.5 Reward Hacking

检查：

\[
R_{AI/RM}\uparrow
\]

但：

\[
True\ Task\ Success\downarrow
\]

的情况。

---

## 12. Phase 8 — 第二环境

只有 MetaWorld 成立后再做。

候选：

- ManiSkill；
- LIBERO。

目的：

> 验证方法不是 MetaWorld-specific。

---

## 13. Phase 9 — 真机扩展

只有以下条件满足后才值得投入：

1. MetaWorld 多任务稳定；
2. 第二环境复现；
3. Human budget advantage 明确；
4. policy-stage shift 有实证；
5. correction 真正改善 downstream policy。

真机候选：

- Unitree；
- 机械臂。

最终部署：

\[
Observation
\rightarrow
\pi_\theta
\rightarrow
Action
\]

不调用 Jev / GPT / Human / Reward Model / Adapter。

---

## 14. 资源规划

### PoC

推荐：

- 1 × 24GB GPU；
- 16–32 CPU cores；
- 32–64GB RAM。

核心瓶颈：

- trajectory collection；
- API labeling；
- Human annotation；
- 多 seed 实验。

不是大模型训练。

### 完整论文

比较舒适：

- 2–4 × 24GB GPU，或同等级共享服务器资源；
- 多任务 / 多 seed 并行。

如果始终使用 frozen black-box evaluator：

> 不需要为 Jev/GPT 准备训练显存。

只有未来额外做 local open evaluator adaptation baseline 时，才需要考虑 LoRA/QLoRA。

---

## 15. 时间规划

### 第 1–2 周：Phenomenon Pilot

完成：

- MetaWorld 环境；
- stage policy collection；
- trajectory serializer；
- AI query wrapper；
- Human annotation；
- 200–400 pairs；
- agreement/calibration/stage-shift 图。

产出：

- Figure 1：AI–Human Agreement by Stage；
- Figure 2：Reliability Diagram；
- Figure 3：Error Pattern Examples；
- Go / No-Go。

### 第 3–4 周：Adapter PoC

完成：

- static calibration；
- contextual calibration；
- stage-aware calibration；
- correction head；
- held-out evaluation。

### 第 5–7 周：Reward Model + SAC

完成：

- RM；
- SAC integration；
- core baselines；
- 3 seeds。

### 第 8–10 周：Human Budget + Online Recalibration

完成：

- fixed-budget query；
- static vs online recalibration；
- 5 seeds for main tasks。

### 第 11–13 周：Robustness

完成：

- systematic bias；
- OOD；
- paraphrase；
- reward hacking；
- evaluator corruption。

### 第 14 周以后

根据结果选择：

- ManiSkill / LIBERO；
- 真机；
- 理论分析；
- local open evaluator baseline。

---

## 16. 必做 Ablation

完整论文至少包括：

1. 去掉 Human anchor；
2. 去掉 AI confidence；
3. 去掉 trajectory context；
4. 去掉 policy stage；
5. 去掉 correction head；
6. 去掉 reliability head；
7. 去掉 online recalibration；
8. 去掉 escalation；
9. static vs online；
10. raw label vs corrected label；
11. raw confidence vs calibrated reliability；
12. source-wise trust vs sample-wise trust。

---

## 17. 最危险 Baseline

必须重点实现：

- RL-VLM-F-style preference reward；
- LAPP-style frozen LLM preference predictor；
- PrefVLM/RIME-style disagreement filtering；
- ROVED-style selective Oracle logic；
- raw confidence weighting；
- source-wise trust；
- global external black-box correction mapping；
- task-level prompt optimization（若工程允许）。

原则：

> 不能只挑弱 baseline。

---

## 18. 最重要 Figure 规划

1. **Problem Figure**：AI–Human Agreement by Policy Stage；
2. **Calibration Figure**：Raw confidence vs Human agreement；
3. **Method Figure**：Frozen AI Teacher → Human-Aligned Feedback Adapter → RM → RL；
4. **Cross-Stage Reliability**：Static vs policy-aware；
5. **Human Budget Curve**：Human budget vs policy success；
6. **RL Learning Curves**：主要 baselines；
7. **Systematic Bias / OOD**：correction robustness；
8. **Ablation**：correction、reliability、stage、online recalibration 的贡献。

---

## 19. 最终 Go / No-Go

### 强 Go

同时满足：

1. AI preference 明显高于随机；
2. raw confidence 存在 miscalibration；
3. Human agreement 有 context/stage dependence；
4. policy-aware adapter 优于 static/global mapping；
5. corrected feedback 提高 RM；
6. RM 改善传递到 policy；
7. 固定 Human budget 下 outperform strong baselines。

### 弱 Go / 改题

如果：

- 没有 policy-stage drift；
- 但存在强 systematic bias；
- correction head 明显有效；

则将论文主线改成：

> **Human-Aligned Feedback Correction for Frozen AI Teachers**

弱化 policy shift。

### No-Go

如果：

- AI 与 Human 接近随机；
- Human calibration 无法泛化；
- correction 只拟合 calibration set；
- RM improvement 不转化为 policy improvement；
- strong baselines 完全覆盖收益；

则停止继续投入该方法。

---

## 20. 当前论文定位

推荐主标题方向：

> **Policy-Conditional Human-Aligned Feedback Correction for Frozen AI Teachers in Preference-Based Reinforcement Learning**

更宽泛版本：

> **Human-Aligned Adaptation of Frozen AI Feedback for Preference-Based Reinforcement Learning**

当前最安全的 contribution 不是：

- “第一次用 AI feedback”；
- “第一次 calibration”；
- “第一次 Human escalation”；
- “第一次 sample-wise reliability”；
- “第一次考虑 policy shift”。

而是：

> **明确研究 frozen AI teacher 的 Human-aligned reliability/correction 如何随 on-policy trajectory distribution 改变，并把这种 policy-conditional、sample-wise Human anchor 真正耦合进 reward learning、Human query allocation 与 downstream robot policy optimization。**

---

## 21. 下一步立刻执行

当前不要再继续扩展方法。

下一步只做：

1. 跑通 MetaWorld Drawer Open + Button Press；
2. 保存 random / early / mid / late policy checkpoints；
3. 自动生成 trajectory pairs；
4. 定义统一 AI prompt / structured input schema；
5. 建立 Human pairwise annotation 表；
6. 完成第一批 200–400 pairs；
7. 画 AI–Human agreement by policy stage；
8. 画 raw confidence reliability diagram；
9. 比较 static vs stage-aware calibrator；
10. 根据结果决定是否进入完整 RM + SAC。

**只有 Phase 1 核心现象成立，才进入后续完整论文开发。**
