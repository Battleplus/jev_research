# 15. Jev Article Implications for the Current Research Route

更新时间：2026-09-23

状态：**补充分析，不替代 `docs/13` 与 `docs/14` 的执行基线**

参考文章：

- Datawhale，《这是一篇把“Jev模型”讲明白的科普级解读！》：<https://mp.weixin.qq.com/s/ynHlQijihuEZ3cqkQtXvrA>

注意：该文章是二手科普材料。采用率、融资、延迟、价格、模型训练方式与 calibration 能力等产品事实，正式论文中必须回到官方文档、模型卡或自行 benchmark 后再引用，不能直接把科普文章当作学术证据。

---

## 1. 总体判断

文章增强了当前方向的**工程可行性**，同时提高了 **novelty 压力**。

Jev 的结构化概率输出、单次前向、低延迟和多问题并行能力，使它很适合充当 scalable preference evaluator。但官方使用方式已经包含：

- confidence threshold；
- uncertain sample routing；
- Human review；
- structured decision pipeline。

因此，项目不应再问：

> Jev 能否给 RL 提供 feedback？

而应进一步收紧为：

> **一个原本以概率输出和 calibration 为核心能力的 frozen decision model，在 on-policy trajectory distribution 改变后，为什么仍可能不再符合 Human Preference？能否用少量 Human Preference 检测并修正这种变化，并最终改善 Reward Model 与 RL policy？**

这与当前主线一致：

\[
Frozen\ AI\ Teacher
\rightarrow
Human\text{-}Aligned\ Feedback\ Adapter
\rightarrow
Reward\ Model
\rightarrow
Preference\text{-}Based\ RL
\rightarrow
Standalone\ Policy
\]

---

## 2. 文章确认的 Jev 特性

### 2.1 判别式、结构化输出

Jev 不生成开放文本，而是在预先定义的选项之间输出概率。文章介绍了：

- `Choice`：离散类别选择；
- `Score`：有序等级分布及期望分数；
- `Noul`：二元命题概率。

对于 trajectory preference，第一版最适合使用：

\[
Choice(A, B, Tie)
\]

而不是要求模型生成自由文本解释，也不建议把同一 preference 拆成两个互相独立的二元问题。

### 2.2 Confidence 不是 Human-Aligned Reliability

文章明确区分：

- probabilities：各候选项上的模型分布；
- confidence：该分布集中程度的摘要；
- calibration：大量样本上的预测概率与经验正确率匹配。

因此：

\[
c_i^{Jev}
\neq
P(y_i^{Jev}=y_i^H)
\]

即使 Jev 在其训练分布和原始标签定义下 calibration 良好，也不意味着：

1. Jev 的评价标准与 Human objective 相同；
2. 在 robot trajectory 上仍然 calibrated；
3. early-policy calibration 能迁移到 late-policy；
4. OOD 或 prompt conflict 下的高 confidence 代表正确；
5. 高 confidence 代表符合 Human Preference。

### 2.3 已知失效模式与本项目高度相关

文章列出的失效模式包括：

- 字面化理解；
- 不擅长算术、日期和多步逻辑；
- 无关长上下文干扰；
- criteria 与 question 冲突；
- 对抗输入；
- 反向问题概率不一致；
- OOD 语言上高 confidence 错误。

这些现象支持对以下变量进行专门测试：

- trajectory difficulty；
- prompt paraphrase；
- A/B order swap；
- irrelevant-context injection；
- criteria conflict；
- OOD state；
- policy stage。

---

## 3. 对 Feedback Adapter 定义的修正

当前仓库主要写作：

\[
F_\phi(x_i,y_i^{AI},c_i^{AI})
\rightarrow
(\hat y_i^H,\rho_i)
\]

根据文章所述的输出结构，实验中不应只保留 argmax label 与 scalar confidence，而应保存完整概率向量：

\[
q_i=(p_A,p_B,p_{Tie})
\]

建议优先统一建模：

\[
F_\phi(q_i,x_i,stage_i)
\rightarrow
P(y_i^H\mid q_i,x_i,stage_i)
\]

再由 Human preference distribution 导出：

\[
\hat y_i^H
=
\arg\max_y P(y_i^H=y\mid q_i,x_i,stage_i)
\]

以及：

\[
\rho_i
=
P(y_i^H=y_i^{AI}\mid q_i,x_i,stage_i)
\]

这样可以避免 correction head 与 reliability head 给出互相矛盾的结果。

建议保存或派生的 AI 特征包括：

- 完整 probability vector；
- top-1 probability；
- top-1 / top-2 margin；
- entropy；
- Tie probability；
- A/B swap consistency；
- paraphrase variance；
- model/version/timestamp。

如果 `confidence` 只是 probability distribution 的确定性摘要，那么同时输入完整 probabilities 和 confidence 可能是冗余的。必须通过 ablation 比较：

1. confidence-only；
2. full-probability；
3. probability + context；
4. probability + context + policy information。

---

## 4. 对第一阶段实验的具体影响

### 4.1 Preference Schema

主问题使用：

\[
Choice(A\ better,B\ better,Tie/Indistinguishable)
\]

Human 与 AI 使用相同的选项语义和 Tie 定义。

Jev 支持对同一 state 并行提出多个问题，因此可以低成本增加诊断维度：

- task progress；
- safety；
- efficiency；
- stability；
- task completion。

第一阶段应把这些维度当作 error-analysis covariates，而不是立即扩展成多目标 RL 方法。

### 4.2 Consistency Test

由于 Jev 是单次前向、无自由文本生成的模型，反复发送完全相同请求的 repeated-query consistency 价值有限。更重要的是：

1. A/B trajectory order swap；
2. candidate label permutation；
3. equivalent instruction paraphrase；
4. criteria order change；
5. irrelevant context injection；
6. positive/negative formulation consistency。

这些测试直接对应文章中提到的反向问题不一致、字面化和上下文干扰。

### 4.3 Structured Trajectory Serializer

文章指出 Jev 不适合自己完成算术与复杂时序计算。因此 serializer 应在代码中计算必要的客观统计量，例如：

- contact event；
- grasp event；
- collision count；
- displacement trend；
- action smoothness；
- trajectory length。

但不能直接输入等价于最终答案的 privileged oracle，例如：

- environment reward；
- success flag；
- “trajectory A progress larger than B”；
- 已经排序好的 goal distance comparison。

否则 AI evaluator 只是在读取 handcrafted reward，无法证明 semantic feedback 的价值。

必须增加：

- deterministic heuristic baseline；
- oracle-like feature removal ablation；
- raw state summary vs derived semantic events comparison。

### 4.4 API Reproducibility

每次 query 至少缓存：

- exact state；
- exact questions/criteria；
- 完整 response；
- full probabilities；
- confidence；
- model/version；
- timestamp；
- latency；
- token/cost；
- retry/error information。

模型版本变化必须视为 evaluator distribution shift，不能把不同版本的输出直接混入同一实验。

---

## 5. 对 Go/No-Go 的影响

文章声称 Jev 本身重视 probability calibration，因此项目不能把“发现一点 raw-confidence miscalibration”作为主要成功标准。

更强的 Go hypothesis 应是：

> **Frozen Jev 的 Human agreement 在 policy-induced trajectory shift 下发生具有实际量级、可重复、且无法只由 task composition 或 pair difficulty 解释的变化；少量新阶段 Human labels 能够比 raw confidence 与 global/static calibration 更有效地预测并修正这种变化。**

进入 Reward Model + SAC 前至少需要：

1. AI preference 优于 chance、majority 与 deterministic heuristic；
2. stage/context effect 在多个 policy seeds 上存在；
3. 控制 task、difficulty、pair similarity 后 effect 仍然存在；
4. early-stage static calibrator 在 mid/late data 上出现退化；
5. contextual/online adapter 在 held-out trajectories 上改善 Brier、NLL、Risk-Coverage；
6. 改善不依赖 simulator oracle leakage。

如果没有 policy-stage shift，但存在可泛化的 systematic context bias，则按照 `docs/13` 转向：

> **Human-Aligned Feedback Correction for Frozen AI Teachers**

如果 correction 也不能跨 trajectory、checkpoint 和 policy seed 泛化，则 No-Go。

---

## 6. 对 Novelty Boundary 的进一步收紧

文章让以下 claim 更不安全：

- Jev 输出结构化概率；
- Jev 比生成式 LLM 更快；
- confidence 可以做 gate；
- low-confidence sample 可以交给 Human；
- frozen decision model 可以嵌入工作流。

这些属于产品能力或标准工程模式，不是论文贡献。

当前仍可保护的核心是：

\[
\boxed{
Human\text{-}Anchored
+
Frozen\ Decision\ Model
+
Context/Sample\text{-}wise\ Correction
+
Policy\text{-}Conditional\ Agreement
+
Online\ Recalibration
+
Downstream\ Robot\ PbRL
}
\]

最安全的论文表述是：

> **We study whether the human-aligned reliability of a probability-calibrated frozen decision model changes under policy-induced trajectory distribution shift, and use a small human anchor to correct and recalibrate its sample-wise preferences for downstream reward and policy learning.**

Jev 仍然应当是：

> one frozen evaluator backend / case study

而不是算法名称或不可替代组件。

---

## 7. 新增的工程与论文机会

文章列出的开源同类模型为 evaluator-agnostic 验证提供了条件，例如：

- Laya；
- Metask-Jev；
- Nimble / 其他 system-one decision models。

第一阶段可以只用 Jev，但正式论文至少应加入一个可复现的 open/local evaluator，用于回答：

1. agreement shift 是否只属于 Jev；
2. adapter 是否能够跨 evaluator 使用；
3. probability features 是否具有通用性；
4. black-box API 版本变化是否影响结论。

开源 backend 的加入应放在核心现象成立之后，不能拖慢 Phase 1。

---

## 8. 对投稿强度的判断

### 仅完成以下内容

- MetaWorld 两任务；
- Jev 单 backend；
- agreement/calibration analysis；

只能视为 PoC，不足以支撑 RA-L 级完整稿。

### RA-L / Q1 级核心版本

至少需要：

- MetaWorld 多任务；
- 严格 Human annotation 与 held-out evaluation；
- policy-stage shift；
- Feedback Adapter；
- Reward Model + SAC 完整因果链；
- fixed Human budget；
- strong baselines；
- systematic bias / OOD / paraphrase robustness；
- 至少一个可复现 evaluator baseline。

### CoRL / RSS 升级条件

需要进一步出现：

\[
Static
<
Policy\text{-}Conditional
<
Online\ Recalibration
\]

并且该排序在最终 policy success 上稳定成立，再加入第二环境、真机或跨 evaluator 泛化。

---

## 9. 最终结论

这篇文章没有要求项目转向“研究 Jev 模型本身”。相反，它进一步支持当前判断：

> **Jev 是一个非常合适、同时也非常有挑战性的 frozen probabilistic evaluator case study。**

项目最有价值的问题仍然是：

\[
\boxed{
P(AI=Human\mid trajectory,\pi_t)
\text{ 是否随 policy/context 系统变化}
}
\]

文章带来的最重要方法修正是：

1. 使用完整 choice probability vector，而非只保存 scalar confidence；
2. 将 Human preference distribution、correction 与 reliability 统一建模；
3. 把 A/B swap、paraphrase、context interference 加入 consistency test；
4. 防止 structured state 泄漏 simulator oracle；
5. 将 Jev confidence gating 明确降级为 baseline；
6. 在核心现象成立后加入 open/local evaluator，增强复现性与通用性。
