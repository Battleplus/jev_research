# 04. 实验路线与止损计划

## Phase 0：接口与标注验证

目标：确认 Jev 是否能稳定理解 structured simulator state。

数据：

- 100–500 个人工可验证 transition / short trajectories。

测试：

- progress classification
- safety classification
- preference consistency
- repeated-query consistency
- paraphrase consistency

输出：

- accuracy
- ECE
- Brier
- NLL
- flip rate

如果 Jev 在结构化状态上表现明显不稳定，直接停止后续大规模 RL。

## Phase 1：最小 RL Proof-of-Concept

环境建议：

- MetaWorld Drawer Open
- Button Press
- Soccer
- Sweep Into

第一阶段不使用视觉，只使用 simulator privileged state。

比较：

1. PPO / SAC + sparse reward
2. + handcrafted dense reward
3. + raw Jev semantic reward
4. + fixed-weight Jev reward

核心问题：

> Jev semantic feedback 是否真的提供额外 learning signal？

## Phase 2：核心算法

加入：

1. raw Jev
2. hard gate
3. calibrated continuous weight
4. reliability-aware advantage
5. calibrated + strong-judge escalation

Calibration 候选：

- temperature scaling
- isotonic regression
- repeated-query consistency
- conformal calibration

## Phase 3：Robustness

### Evaluator noise

人工注入：

- 10%
- 20%
- 30%
- 40%

错误 semantic feedback。

### Paraphrase test

同一 trajectory 使用多组语义等价 instruction。

观察：

- reward variance
- label flip rate
- policy degradation

### OOD

测试：

- object appearance shift
- viewpoint shift
- scene background shift
- unseen goal composition

### Reward hacking

检查：

\[
r^{semantic}\uparrow
\quad\text{但}\quad
r^{true}\downarrow
\]

的行为。

## Phase 4：机器人扩展

如果 Phase 1–3 成立，再进入：

- ManiSkill
- LIBERO
- VLA fine-tuning
- Unitree / 机械臂真机

视觉版本：

\`\`\`text
RGB
 ↓
Perception / VLM
 ↓
structured state
 ↓
semantic evaluator
 ↓
calibrated feedback
 ↓
RL
\`\`\`

需要把 perception error 与 evaluator error 分开报告。

## 必做 baseline

- sparse/environment reward
- handcrafted dense reward
- RL-VLM-F-style preference reward
- raw VLM/LLM semantic reward
- consistency-filtered feedback
- hard confidence gate
- uncertainty-aware policy shaping
- PBRS-style semantic shaping
- evaluator ensemble / pessimistic baseline

## 必报指标

### RL
- success rate
- true return
- sample efficiency
- AUC learning curve
- multi-seed mean ± CI

### Evaluator
- ECE
- Brier
- NLL
- risk-coverage / selective risk
- confidence-error correlation

### System
- evaluator calls
- strong judge escalation rate
- latency
- wall-clock
- API cost

## 止损点

### 止损点 1
Jev 对 structured state 的基础判断不稳定。

### 止损点 2
raw semantic feedback 对 RL 没有任何稳定增益。

### 止损点 3
calibration accuracy 提升，但最终 policy performance 不提升。

### 止损点 4
收益完全来自额外 query budget，而不是 trust allocation 本身。

只有通过前 3 个阶段，才值得投入真机和更大模型。
