# 00. 两周 Proof-of-Concept 实施清单

目标：

> 用最小成本回答一个问题：**Jev semantic feedback 是否真的能给 RL 提供稳定、可利用的 learning signal？**

第一阶段暂时不追求论文完整度，不碰复杂视觉，不碰真机。

---

# Week 1：先验证 Jev 本身

## Day 1：环境与任务选型

建议 MetaWorld 中选 2–4 个任务：

- Drawer Open
- Button Press
- Soccer
- Sweep Into

要求：

- simulator 有 ground-truth success；
- 有明确 progress；
- structured state 容易提取；
- 任务难度不要过高。

先只选其中 **2 个任务**跑通。

---

## Day 2：定义 Structured State

从 simulator 提取：

- gripper position
- object position
- target position
- distance to target
- object grasped
- contact / collision
- task-specific progress

不要直接给原始连续状态向量。

转换成可读 structured description，例如：

\`\`\`text
Task: open the drawer.

Previous state:
- drawer open ratio: 0.18
- gripper distance to handle: 0.06 m
- handle contact: false

Current state:
- drawer open ratio: 0.31
- gripper distance to handle: 0.02 m
- handle contact: true
\`\`\`

---

## Day 3：定义 Jev 输出

先不要让 Jev 返回复杂自由分数。

建议使用离散 Choice：

### Progress
- regression
- no_progress
- weak_progress
- strong_progress

### Safety
- safe
- risky
- violation

### Optional
- task_complete: true / false

保留 Jev 的原始概率/confidence。

---

## Day 4：离线 Evaluation Set

从 random / scripted / baseline policy 中收集：

- 100–500 transition pairs；
- 成功；
- 失败；
- 接近成功；
- 明显倒退；
- ambiguous transition。

利用 simulator ground truth 构建 oracle label。

评估：

- accuracy
- confusion matrix
- ECE
- Brier score
- repeated-query consistency

核心问题：

> Jev confidence 与实际 correctness 是否正相关？

---


## Day 4B：Human Preference Anchor（建议加入）

如果目标论文要明确与 PbRL / RLHF 建立联系，第一阶段最好额外收集一小批真实 human pairwise preferences。

对同一批 trajectory pairs：

\[
(\sigma^A,\sigma^B)
\]

同时获得：

\[
y_H,\quad y_J,\quad c_J
\]

其中：

- \(y_H\)：human preference；
- \(y_J\)：Jev preference；
- \(c_J\)：Jev confidence。

第一阶段不需要大量人类标注，重点是验证：

\[
c_J
\stackrel{?}{\longrightarrow}
P(y_J=y_H)
\]

至少报告：

- Human–Jev agreement
- Jev confidence vs agreement
- ECE / Brier
- disagreement by task difficulty
- low-confidence sample examples

如果两周内暂时不方便做人类标注，可以先用 simulator oracle 验证工程链路；但最终若声称属于 human-preference / RLHF 方向，需要补真实 human preference anchor。


## Day 5：Paraphrase Test

同一个 state pair 构造 3–5 个语义等价 task descriptions。

例如：

- Open the drawer.
- Pull the drawer outward until it is open.
- Make the drawer more open.
- Increase the drawer opening.

观察：

- prediction flip rate；
- confidence variance；
- score crossing。

如果同一 trajectory 因措辞不同发生大量翻转，需要在后续方法中专门处理。

---

# Week 2：最小 RL 闭环

## Day 6–7：跑 baseline

先得到可靠 baseline：

1. PPO/SAC + sparse environment reward
2. PPO/SAC + handcrafted dense reward

至少：
- 3 seeds
- 固定训练 steps
- success curve
- return curve

---

## Day 8：Raw Jev Reward

最简单映射，例如：

\[
r_t^{Jev}
=
\begin{cases}
-1 & regression \\
0 & no\_progress \\
0.5 & weak\_progress \\
1 & strong\_progress
\end{cases}
\]

然后：

\[
r_t
=
r_t^{env}
+
\lambda r_t^{Jev}
\]

注意：

- 不要每 env step 在线请求；
- 先离线缓存；
- 或每 \(K\) step 调一次；
- 记录所有 API response。

---

## Day 9：Fixed Weight vs Hard Gate

比较：

### Fixed
\[
r_t
=
r_t^{env}
+
\lambda r_t^{Jev}
\]

### Hard Gate
\[
r_t
=
r_t^{env}
+
\lambda
\mathbb{1}[c_t>\tau]
r_t^{Jev}
\]

先看最简单 confidence gate 是否有效。

---

## Day 10：简单 Calibration

先不要一上来做复杂 conformal。

优先测试：

- temperature scaling
- isotonic regression

得到：

\[
c_t\rightarrow \hat c_t
\]

然后：

\[
r_t
=
r_t^{env}
+
\lambda\hat c_t r_t^{Jev}
\]

---

## Day 11–12：核心比较

至少跑：

1. Sparse RL
2. Handcrafted dense reward
3. Raw Jev reward
4. Fixed-weight Jev
5. Hard-gated Jev
6. Calibrated-weight Jev
7. Human-only preference RM（小规模 anchor baseline）
8. Human + Jev naïve pooling
9. Human-anchored calibrated Jev

指标：

- success rate
- sample efficiency
- true return
- training variance

---

## Day 13：Noise Stress Test

人为把 Jev feedback 翻转：

- 10%
- 20%
- 30%

看：

> calibrated weighting 是否比 raw reward 更抗 evaluator error？

---

## Day 14：Go / No-Go 决策

### Go 条件

满足至少 2 条：

- Raw Jev 比 sparse baseline 有稳定增益；
- confidence 与 correctness 有明显相关；
- calibrated weight 比 raw / hard gate 更稳定；
- 在 noise injection 下优势明显。

### No-Go 条件

出现任一严重问题：

- Jev structured-state 判断接近随机；
- confidence 与 correctness 无关；
- 加 Jev 后 RL 明显更差；
- calibration 只改善 evaluator metric，不改善 policy；
- API / latency / reproducibility 成本无法接受。

---

# 两周结束必须产出

1. 一张 Jev calibration 图
2. 一张 RL learning curve
3. 一张 human/Jev agreement + calibration 图
4. 一张 Human-only / Jev-only / naïve hybrid / calibrated hybrid comparison table
5. 一张 noise robustness 曲线
6. 一页 Go / No-Go 结论

---

# 第一阶段明确不做

- 不做 RGB vision
- 不做 VLA
- 不做真机
- 不训练大模型
- 不追求 end-to-end
- 不一开始就做理论证明

先证明：

> **semantic feedback + calibrated trust 是否真的能改变 RL 学习结果。**

这个成立之后，再决定是否升级为完整论文。
