# Human-Calibrated AI Preference Reinforcement Learning

本仓库用于整理 **Human Preference + AI/Jev Preference + Reliability Calibration + Preference-Based RL** 方向的调研、研究构想、实验方案与阶段性结论。Jev 当前被视为一个可替换的 fast probabilistic evaluator backend，而不是论文贡献本身。

> 当前核心判断：  
> **“Jev → reward/preference → RL → standalone policy”这一大框架已经有明确先例；当前真正值得验证的是：用少量 Human Preference 估计 AI/Jev 与 Human objective 的 sample-wise agreement probability，并让该 Human-Aligned Reliability 控制 AI preference weighting、Human escalation 与 downstream policy learning。**

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

**Human-Anchored Contextual Reliability for AI-Assisted Preference-Based Reinforcement Learning**

更完整的论文表述：

> **Human-Anchored Reliability Calibration for AI-Assisted Preference-Based Reinforcement Learning**
>
> 用少量 Human Preference 作为目标锚点，不直接相信 AI/Jev 的 raw confidence，而是学习：
>
> \[
> \rho_i=P(y_i^{AI}=y_i^{Human}\mid x_i,c_i^{AI})
> \]
>
> 再根据 \(\rho_i\) 决定 AI preference 的训练权重、是否升级到 Human/Strong Judge，以及其对 downstream policy learning 的影响。

## 当前结论

1. **AI / LLM / VLM feedback → RL 已经有大量先例**，因此不能把“使用 Jev 给 reward”作为主要 novelty。
2. **confidence-aware / uncertainty-aware RL 也已有工作**，因此简单 hard gate 同样不够新。
3. 仍有空间的核心问题是：
   - calibrated evaluator uncertainty 如何进入 reward / advantage；
   - 如何处理 OOD、reward hacking、paraphrase instability；
   - 如何根据 uncertainty 决定是否升级到 strong judge；
   - 如何在 robot RL 中验证最终 policy quality，而不仅仅是 evaluator accuracy。
4. **训练期 AI teacher、部署期 standalone policy 已经有大量先例**，包括 RL-VLM-F、LAPP、Preference VLM 等，因此这不能作为 novelty。
5. **Hybrid Human+AI preference、uncertain sample→Human、confidence weighting 也已有近邻工作**，包括 Preference VLM、ROVED、Hybrid Preferences、CW-PO、BACON、Conformal Feedback Alignment 等。
6. 当前最值得争取的 novelty 是：**Human-Aligned sample-wise reliability**，而不是 raw AI confidence。
7. 最新 Undermind Deep Search 共检索 70 篇高相关工作，**未发现完整实现“small Human anchor → context/sample-wise AI–Human agreement calibration → preference weighting/escalation → downstream PbRL/robot RL → standalone policy”整条链路的论文**。当前 novelty 必须限定在这一 end-to-end coupling，而不是任一单模块。
8. 当前建议进一步加入 **policy-induced AI feedback reliability shift**：同一个 evaluator 的 Human agreement 可能随着 policy distribution 改变而漂移。
9. Jev 是闭源模型，不需要训练 Jev 本身，也不应成为算法不可替代组件；部署阶段最终 policy 应独立运行，不再调用 Jev。

## 仓库结构

- [docs/01_Literature_Review.md](docs/01_Literature_Review.md)：相关方向调研与 novelty 边界
- [docs/02_Senior_Evaluation_Report.md](docs/02_Senior_Evaluation_Report.md)：给学长评估的完整报告
- [docs/03_Method_v1.md](docs/03_Method_v1.md)：当前方法设计与 Jev 闭源调用方案
- [docs/04_Experiment_Plan.md](docs/04_Experiment_Plan.md)：实验路线和止损计划
- [docs/05_References.md](docs/05_References.md)：核心参考文献
- [docs/06_Reading_Plan.md](docs/06_Reading_Plan.md)：文献精读顺序
- [docs/07_Human_Preference_RL_Lineage.md](docs/07_Human_Preference_RL_Lineage.md)：与人类偏好强化学习 / RLHF 的关系与最终定位
- [docs/08_Current_Progress_2026-09-22.md](docs/08_Current_Progress_2026-09-22.md)：当前进度、novelty 边界、难度与风险
- [docs/09_Closest_Prior_Work_Map.md](docs/09_Closest_Prior_Work_Map.md)：最接近 prior work 与模块级撞题地图
- [docs/10_2026_Closest_Work_Verification.md](docs/10_2026_Closest_Work_Verification.md)：配额恢复后对 2026 最危险近邻工作的全文核验
- [docs/11_Final_Deep_Search_Result.md](docs/11_Final_Deep_Search_Result.md)：70 篇 Deep Search 最终结论、novelty 边界与当前定稿研究问题
- [experiments/00_POC_Plan.md](experiments/00_POC_Plan.md)：第一版两周 proof-of-concept 计划
- [experiments/01_Human_Calibrated_Jev_POC_v2.md](experiments/01_Human_Calibrated_Jev_POC_v2.md)：当前推荐的 Human-Calibrated Jev PoC v2

## 当前建议的目标

第一阶段先做 **MetaWorld / structured-state proof-of-concept**，只回答三个问题：

1. Jev raw confidence 是否能够预测 Jev–Human preference agreement？
2. 少量 Human Preference 能否把 Jev confidence 校准成更可靠的 Human-Aligned Reliability？
3. Jev–Human agreement 是否随 trajectory difficulty / OOD / **policy stage** 系统变化？
4. Static calibration 是否会随着 policy distribution shift 失效，而 online recalibration 能否维持 reliability？
5. 更好的 reliability estimation 是否最终改善 Reward Model 和 RL policy，而不只是改善 ECE/Brier？

如果这三条成立，再进入 selective escalation、distribution shift、OOD、reward hacking、ManiSkill/LIBERO 和真机。

更新时间：2026-09-22
