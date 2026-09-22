# Jev-Assisted Reinforcement Learning Research

本仓库用于整理 **Jev / calibrated semantic evaluator 辅助强化学习训练** 方向的调研、研究构想、实验方案与阶段性结论。

> 当前核心判断：  
> **“Jev → reward → PPO/SAC”本身已经不足以作为主要创新点。更有研究价值的方向是：把外部语义评价器视为带噪反馈源，通过 calibration / uncertainty-aware trust allocation 动态控制其对 RL policy update 的影响。**

## 研究问题

我们希望研究：

> 当外部语义评价器并不完全可靠时，强化学习应该 **什么时候相信它、相信多少，以及什么时候不应该使用它**？

基础形式：

\[
r_t = r_t^{env} + \lambda r_t^{sem}
\]

进一步考虑评价器可靠性：

\[
r_t = r_t^{env} + \lambda w_t r_t^{sem},
\qquad
w_t = f(c_t, \mathrm{OOD}_t, \mathrm{disagreement}_t)
\]

其中：

- \(r_t^{env}\)：环境奖励；
- \(r_t^{sem}\)：Jev / LLM / VLM / local RM 产生的语义反馈；
- \(c_t\)：评价器 confidence / reliability；
- \(w_t\)：经过 calibration 后的信任权重。

更进一步，可直接作用到 advantage：

\[
\tilde A_t = A_t^{env} + \lambda w_t A_t^{sem}
\]

## 当前推荐名称

**Calibrated Semantic Feedback Reinforcement Learning (CSF-RL)**

Jev 在这里作为一个 fast semantic evaluator / teacher，而不是论文方法本身。

## 当前结论

1. **AI / LLM / VLM feedback → RL 已经有大量先例**，因此不能把“使用 Jev 给 reward”作为主要 novelty。
2. **confidence-aware / uncertainty-aware RL 也已有工作**，因此简单 hard gate 同样不够新。
3. 仍有空间的核心问题是：
   - calibrated evaluator uncertainty 如何进入 reward / advantage；
   - 如何处理 OOD、reward hacking、paraphrase instability；
   - 如何根据 uncertainty 决定是否升级到 strong judge；
   - 如何在 robot RL 中验证最终 policy quality，而不仅仅是 evaluator accuracy。
4. Jev 是闭源模型，不需要训练 Jev 本身。建议：
   - Jev 冻结做 teacher / evaluator；
   - PPO / SAC policy 正常训练；
   - 可选训练本地 surrogate reward model；
   - 高频 inner-loop 优先使用本地 evaluator；
   - 低置信样本再调用 Jev / strong judge。

## 仓库结构

- [docs/01_Literature_Review.md](docs/01_Literature_Review.md)：相关方向调研与 novelty 边界
- [docs/02_Senior_Evaluation_Report.md](docs/02_Senior_Evaluation_Report.md)：给学长评估的完整报告
- [docs/03_Method_v1.md](docs/03_Method_v1.md)：当前方法设计与 Jev 闭源调用方案
- [docs/04_Experiment_Plan.md](docs/04_Experiment_Plan.md)：实验路线和止损计划
- [docs/05_References.md](docs/05_References.md)：核心参考文献

## 当前建议的目标

第一阶段先做 **MetaWorld / structured-state proof-of-concept**，验证：

> Jev semantic feedback 是否真的提供了可用于 RL 的有效 learning signal？

如果成立，再加入 calibration、uncertainty-aware weighting、reward hacking / OOD 测试，最后进入 ManiSkill / LIBERO / 真机。

更新时间：2026-09-22
