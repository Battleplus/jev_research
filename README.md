# Human-Aligned Adaptation of Frozen AI Feedback for Preference-Based Reinforcement Learning

> **一句话：不是研究“怎么后训练 Jev/ChatGPT”，而是研究“面对一个不可修改的 frozen black-box AI teacher，如何用少量 Human Preference 修正、校准并可靠利用它的 feedback 来训练 RL”。**

本仓库用于整理 **Frozen Black-box AI Teacher + Human-Aligned Feedback Adaptation + Preference-Based RL** 方向的调研、研究构想、实验方案与阶段性结论。Jev / GPT / VLM 当前都被视为可替换的 frozen evaluator backend，而不是论文贡献本身。

当前完整研究链路：

[
oxed{
Small Human Anchor
ightarrow
Contextual AI	ext{-}Human Agreement
ightarrow
Preference Weighting/Escalation
ightarrow
Reward Model
ightarrow
Preference	ext{-}Based RL
ightarrow
Standalone Policy
}
]

核心不是 AI 自己“有多自信”，而是：

[
oxed{
ho_i=P(y_i^{AI}=y_i^{Human}mid x_i,c_i^{AI},difficulty_i,OOD_i,stage_i)
}
]

即 **同一个 AI evaluator 在当前具体 trajectory/context 上，与 Human Preference 一致的概率**。

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

**Human-Aligned Adaptation of Frozen AI Feedback for Preference-Based Reinforcement Learning**

更聚焦的方法名：

**Policy-Conditional Human-Aligned Feedback Correction for Frozen AI Teachers**

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

## 为什么不是直接相信 AI confidence？

Jev / LLM / VLM 可以输出 preference 和 confidence，但：

[
c_i^{AI}
eq P(y_i^{AI}=y_i^{Human})
]

同一个 evaluator 可能对简单 trajectory 很准，对相似 trajectory、OOD trajectory 或训练后期出现的新行为失准。因此我们希望用少量 Human Preference 定义：

[
a_i=mathbb{1}[y_i^{AI}=y_i^{Human}]
]

再学习：

[
ho_i
=
g_phi(
c_i^{AI},
z(	au_i^A,	au_i^B),
difficulty_i,
OOD_i,
stage_i
)
]

使 (ho_i) 逼近 Human-Aligned Reliability。

### Reliability 如何进入 RL？

首先用于 Reward Model：

[
mathcal L_{RM}
=
sum_{iin D_H}CE(P_psi,y_i^H)
+
lambdasum_{jin D_{AI}}ho_j CE(P_psi,y_j^{AI})
]

其次用于 selective escalation：

[
ho_i>	au_hRightarrow Use AI
]

[
	au_l<ho_ile	au_hRightarrow Downweight
]

[
ho_ile	au_lRightarrow Query Human/Strong Judge
]

第一版先训练 Reward Model + SAC；只有基础结果成立后，再考虑让 reliability 直接进入 policy advantage。

## Policy-Induced AI Feedback Reliability Shift

RL policy 会不断变化：

[
pi_0ightarrowpi_1ightarrowcdotsightarrowpi_T
]

因此 trajectory distribution 也变化：

[
d^{pi_0}(s,a)
eq d^{pi_T}(s,a)
]

我们重点验证：

[
oxed{
P(AI=Humanmid d^{pi_0})

eq
P(AI=Humanmid d^{pi_T})
}
]

也就是说，训练早期校准好的 evaluator 到训练后期可能失准。后续比较 **Static Calibration** 与 **Online Human Recalibration**，观察 ECE/Brier 与最终 Policy Success 是否随 policy shift 发生系统变化。

## 2026-09-23 V2 路线修正：Frozen Black-box Teacher

新的关键约束：

> **如果 Jev / ChatGPT / 其他闭源 evaluator 没有官方 fine-tuning/post-training 接口，我们不能直接更新其模型参数。**

这不是方案漏洞，而是研究设定：

\[
\boxed{\theta_{AI}\ \text{fixed}}
\]

当前不再研究：

\[
Human\ Feedback
\rightarrow
Post\text{-}train\ Jev/GPT
\]

而是研究：

\[
\boxed{
Frozen\ AI\ Teacher
\rightarrow
Human\text{-}Aligned\ Feedback\ Adapter
\rightarrow
Preference\ RL
}
\]

建议的外部可训练模块：

\[
F_\phi(
\tau_A,\tau_B,
y_{AI},c_{AI},
difficulty,OOD,policy\ stage
)
\rightarrow
(\hat y_H,\rho)
\]

其中：

- \(\hat y_H\)：预测 Human 更可能选择的 preference；
- \(\rho=P(y_{AI}=y_H\mid context,\pi_t)\)：AI 与 Human 在当前样本上的 agreement reliability。

也就是说，方法从单纯 **Reliability Calibration** 升级为：

\[
\boxed{Human\text{-}Aligned\ Feedback\ Adaptation}
\]

专项 Undermind Deep Search 共返回 **236 篇相关工作**。全文核验后确认：

- Frozen foundation model → preference/reward → RL 已有充分先例；
- small Human + AI routing 已有先例；
- sample-wise filtering/correction 已有先例；
- black-box AI + external human correction mapping 已有先例；
- frozen VLM + prompt optimization 已有先例；
- policy-aware reward learning 已有先例。

因此当前最值得守住的 novelty 不是任一单模块，而是：

\[
\boxed{
Human\text{-}Anchored
+
Frozen\ Black\text{-}box\ AI
+
Sample/Context\text{-}wise\ Correction
+
Policy\text{-}Conditional\ Reliability
+
Online\ Recalibration
+
Downstream\ Robot\ PbRL
}
\]

新的核心实验假设：

\[
\boxed{
P(AI=Human\mid d^{\pi_0})
\neq
P(AI=Human\mid d^{\pi_T})
}
\]

即：**同一个 frozen AI teacher 的 Human agreement 可能随着 policy-induced trajectory distribution shift 系统变化。**

详见：

[docs/12_Frozen_Black_Box_Teacher_V2.md](docs/12_Frozen_Black_Box_Teacher_V2.md)

---

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
6. 当前最值得争取的 novelty 不是单独的 sample-wise reliability，而是：**Human-Anchored + AI-Specific + Contextual + Sample-wise + Policy-Aware + downstream Preference RL 的完整耦合**。
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
- [docs/12_Frozen_Black_Box_Teacher_V2.md](docs/12_Frozen_Black_Box_Teacher_V2.md)：236 篇专项检索 + 全文核验后的 V2 路线，解释闭源模型为何不需要后训练、外部 feedback adapter 应如何设计
- [experiments/00_POC_Plan.md](experiments/00_POC_Plan.md)：第一版两周 proof-of-concept 计划
- [experiments/01_Human_Calibrated_Jev_POC_v2.md](experiments/01_Human_Calibrated_Jev_POC_v2.md)：当前推荐的 Human-Calibrated Jev PoC v2

## 最危险的近邻工作

当前 Related Work 必须重点对照：

| 工作 | 已覆盖部分 | 与当前方案的关键差别 |
|---|---|---|
| Trust or Escalate (2024) | Human agreement calibration + trust/escalate | 没有 downstream Preference RL |
| Preference VLM (2025) | VLM + minimal Human + filtering/routing + SAC | 主要依据 RM↔VLM disagreement，而非显式 (P(AI=Human\mid x)) |
| ROVED (2026) | uncertainty + targeted Oracle + PbRL | uncertainty routing 不等于 Human-anchored agreement calibration |
| Hybrid Preferences (2024) | Human/AI routing | 不显式学习 AI–Human agreement probability 作为训练权重 |
| Judging with Confidence (2025) | sample-wise Human preference calibration | calibration 后没有继续进入 RL |
| TriTrust-PBRL (2026) | expert trust + weighted RM + SAC | 主要是 source-wise (alpha_k)，而我们研究 sample/context-wise (ho_{ik}) |
| Feature-Dependent Noise in PbRL (2026) | trajectory-dependent preference noise + RL | 不是 small Human anchor → AI–Human agreement calibration |

最新专项 Deep Search 共返回 **70 篇高相关工作**，未发现完整覆盖：

[
Human Anchor
ightarrow
P(AI=Humanmid sample,context,policy stage)
ightarrow
Weighting/Escalation
ightarrow
PbRL/Robot RL
ightarrow
Standalone Policy
]

这只是当前检索范围内的 novelty boundary，**不能表述成“确定首次”**。

## 当前建议的目标

第一阶段先做 **MetaWorld / structured-state proof-of-concept**，只回答三个问题：

1. **Q1：** Jev raw confidence 能否预测 Jev–Human preference agreement？
2. **Q2：** 少量 Human Preference 能否把 raw confidence 校准成更可靠的 Human-Aligned Reliability？
3. **Q3：** Jev–Human agreement 是否随 trajectory difficulty / OOD / policy stage 系统变化？
4. **Q4：** Static calibration 是否会随 policy distribution shift 失效，online recalibration 能否恢复？
5. **Q5：** calibration 改善是否真正传导为：
   [
   Better Preference
   ightarrow Better Reward Model
   ightarrow Better Policy
   ]

第一阶段建议只收集约 **200–500 个 MetaWorld trajectory pairs**，覆盖 random / early / middle / near-converged policy，并同时保存 Human label、Jev label、Jev confidence、task、difficulty、OOD 与 policy stage。

关键指标：**Agreement Accuracy、ECE、Brier、NLL、Risk-Coverage、Policy Success Rate**。

如果这三条成立，再进入 selective escalation、distribution shift、OOD、reward hacking、ManiSkill/LIBERO 和真机。

更新时间：2026-09-23
