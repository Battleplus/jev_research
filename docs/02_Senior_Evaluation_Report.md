# 02. 给学长的方向评估报告

## 一、想法来源与核心问题

最近关注到 TypeSafe 新发布的 Jev。Jev 主要面向结构化决策，可以针对给定状态输出 Choice、Score、概率/置信度等结果。

最开始的想法是：

> 能否把 Jev 作为强化学习训练过程中的辅助评价模块，让它对 agent / 机器人产生的状态、动作或轨迹进行评价，再把评价结果作为额外 reward、preference 或 safety signal，辅助 PPO、SAC 等算法训练？

最基础形式：

\[
r_t = r_t^{env} + \lambda r_t^{Jev}
\]

进一步调研之后发现：

> **单纯“Jev → reward → PPO/SAC”这个想法本身已经不够新。**

真正可能形成研究价值的问题应该变成：

> **当外部语义评价器并不完全可靠时，RL 应该什么时候相信它、相信多少，以及什么时候不应该使用它？**

因此目前更倾向于把课题抽象成：

## Calibrated Semantic Feedback Reinforcement Learning

即：

> 将 Jev 或其他语义评价模型看作一个“带噪外部反馈源”，利用经过 calibration 的置信度动态决定其对 RL 更新的影响。

Jev 是重要 evaluator，但方法不绑定 Jev。


## 二、和“人类偏好强化学习”的关系

这次进一步看了三篇核心文献：

1. **Christiano et al., Deep Reinforcement Learning from Human Preferences**
2. **Kaufmann et al., A Survey of Reinforcement Learning from Human Feedback**
3. **Zhong et al., A Comprehensive Survey of Reward Models**

现在我认为，这个课题如果要和 Preference RL / RLHF 真正接上，最合理的逻辑不是“Jev 直接给 reward”，而是：

\`\`\`text
Christiano:
Human Preference → Reward Model → RL Policy

RLAIF:
AI Preference → Reward Model → RL Policy

我们的设想:
Human Preference (少量 anchor)
            ↓
      calibrate / validate
            ↓
Jev Preference + Confidence
            ↓
sample-wise reliability
            ↓
Reward Model / Policy Update
            ↓
PPO / SAC
\`\`\`

Christiano 的核心是让人比较两段 trajectory 哪个更好，而不是要求人写出数值 reward。Reward predictor 从 pairwise preference 中学习潜在 reward，policy 再优化这个 reward。

Kaufmann 的 survey 把现代 RLHF 看成 PbRL 的广义扩展：pairwise trajectory comparison 属于最典型的 PbRL，同时也是 RLHF 的经典形式。也就是说，Christiano-style pipeline 本质上是“preference collection → reward learning → policy learning”。

Zhong 的 Reward Model survey 又把 preference collection 分成 **Human Preference** 和 **AI Preference**。因此，若 Jev 负责比较 trajectory pair，它首先属于 **AI preference source / RLAIF lineage**；若 Jev 直接给 scalar reward，则更接近 generative reward model / Direct-RLAIF。

所以：

> **单纯把 Human 换成 Jev，并不是新的研究问题。**

我现在更想研究的是：

> **Human 和 AI feedback 之间的信任分配。**

具体是保留少量 human preference 作为目标锚点，用它来估计：

\[
\rho_i
=
P(
y_i^{Jev}=y_i^{Human}
\mid
x_i,c_i^{Jev}
)
\]

然后：

- 高 \(\rho_i\)：正常使用 Jev preference；
- 中等 \(\rho_i\)：低权重使用；
- 低 \(\rho_i\)：升级给 human / stronger judge。

如果训练 local reward model：

\[
\mathcal L_{RM}
=
\sum_{i\in D_H} CE(P_\psi,y_i^H)
+
\alpha
\sum_{j\in D_J}
\rho_j CE(P_\psi,y_j^J)
\]

这里 human label 是 anchor，Jev label 的权重由它与 human objective 的一致概率决定。

这样本课题和“人类偏好强化学习”的联系就不再是口头上的，而是直接进入算法结构。

### 这个定位的好处

1. 保留 Christiano-style preference RL 的基本逻辑；
2. 比“Jev 直接给 reward”更容易和 RLAIF / RL-VLM-F 做清晰对比；
3. Jev 闭源影响变小，因为它主要做 labeler，而不是 RL 内循环模型；
4. 可以把 **human query efficiency** 作为一个明确贡献；
5. calibration 的目标也更清楚：不是校准 Jev 自己的 confidence，而是校准 **Jev 与 human preference 一致的概率**。

因此目前更具体的题目可以考虑：

> **Human-Anchored Calibrated AI Feedback for Preference-Based Reinforcement Learning**

详细推导见 [07_Human_Preference_RL_Lineage.md](07_Human_Preference_RL_Lineage.md)。


## 三、目前相关方向做到什么程度

目前比较接近的路线包括：

| 工作方向 | 已有做法 | 对本课题的影响 |
|---|---|---|
| RL-VLM-F | VLM 产生 preference → reward model → SAC | AI feedback → robot RL 已成立 |
| RLAIF | AI feedback 替代 human feedback | “AI 当老师”本身不新 |
| Noisy LLM feedback | 多次查询、一致性过滤、potential-like reward | 已处理 evaluator 不可靠问题 |
| Uncertainty-aware LLM guidance | uncertainty 动态混合 LLM policy 与 PPO policy | confidence-aware RL 已有先例 |
| RARM | confidence threshold 门控 robot progress reward | hard gate 已有人做 |
| VLM-AR3L | absolute + relative reward + confidence check | reward reliability 已进入 robot RL |
| Large Reward Models | VLM online reward → PPO | online foundation-model reward 已出现 |
| UARM | reward uncertainty → advantage reweighting | uncertainty 进入 update 已有工作 |
| Strong Judge Routing | RM 不确定时调用 stronger judge | uncertainty routing 已有先例 |

因此：

> **“LLM/VLM/Judge 给 RL reward”已经不是空白。**

## 四、目前认为还有价值的创新点

### Calibrated Trust Allocation

让 evaluator 输出：

\[
E(\tau_t)\rightarrow(q_t,c_t)
\]

其中：

- \(q_t\)：semantic progress / preference / safety；
- \(c_t\)：confidence / reliability。

不是固定：

\[
r_t = r_t^{env} + \lambda q_t
\]

而是：

\[
r_t
=
r_t^{env}
+
\lambda w_t q_t
\]

其中：

\[
w_t
=
f(c_t,\mathrm{OOD}_t,\mathrm{disagreement}_t)
\]

进一步可直接进入 PPO advantage：

\[
\tilde A_t
=
A_t^{env}
+
\lambda w_tA_t^{semantic}
\]

希望研究：

> calibration 是否真的能够改善最终 RL policy，而不只是提升 evaluator accuracy。

## 五、Jev 的角色

\`\`\`text
Environment
     │
     ↓
Policy → trajectory
             │
             ↓
      Semantic Evaluator
       Jev / LLM / RM
             │
       ┌─────┴─────┐
       ↓           ↓
 semantic score  confidence
       │           │
       └─────┬─────┘
             ↓
      Trust Allocation
             │
    ┌────────┼─────────┐
    ↓        ↓         ↓
  使用    降低权重   不采用/升级
                     Strong Judge
             │
             ↓
          PPO / SAC
             │
             ↓
           Policy
\`\`\`

Jev 更适合做：

> fast semantic teacher / evaluator

而不是 RL policy 本身。

## 六、Jev 不开源的问题

Jev 是闭源 API，因此不能下载权重进行 end-to-end fine-tuning。

但本课题并不需要训练 Jev。

建议方案：

### Jev 做 teacher，本地 evaluator 做 student

先让 Jev 对部分 trajectory 评价：

\[
D_J=
\{(\tau_i,q_i,c_i)\}_{i=1}^{N}
\]

可选训练本地 evaluator：

\[
f_\phi(\tau)
\rightarrow
(\hat q,\hat c)
\]

高频 RL inner-loop 使用本地 evaluator，只在 calibration、OOD、low-confidence、key transition 情况下调用 Jev / strong judge。

需要提前确认 TypeSafe 服务条款是否允许使用 API 输出训练 surrogate model。

## 七、主要实验问题

最终必须证明的不是：

> Jev 打分准不准。

而是：

\[
\text{better calibrated feedback}
\Rightarrow
\text{better learned policy}
\]

至少比较：

1. PPO/SAC baseline
2. handcrafted dense reward
3. raw semantic evaluator reward
4. fixed-weight semantic reward
5. hard confidence gate
6. calibrated continuous weighting
7. calibrated weighting + escalation
8. alternative LLM/VLM/local evaluator

指标包括：

- Success Rate
- True Return
- Sample Efficiency
- ECE
- Brier Score
- NLL
- OOD Performance
- Reward Hacking
- API Calls / Cost / Latency

## 八、第一阶段建议

先不碰 RGB / VLM perception。

直接用 MetaWorld simulator privileged state：

- gripper position
- object position
- goal position
- is_grasped
- distance-to-goal
- collision

转换成 structured description 再交给 evaluator。

第一阶段只回答：

> **Jev semantic feedback 到底有没有对 RL 提供有效 learning signal？**

如果没有，及时止损。

如果有效，再继续：

\[
Jev\ confidence
\rightarrow
calibration
\rightarrow
trust weighting
\]

然后再进入 ManiSkill / LIBERO / 真机。

## 九、难度判断

- 简单 Jev + PPO：约 5/10
- calibration + uncertainty + OOD + reward hacking：约 7.5–8/10
- 多机器人任务 + 真机 + 理论：约 9/10

## 十、希望学长重点帮忙判断

1. calibrated semantic feedback + RL 的创新量是否够？
2. 更适合做 general RL、robot RL 还是 Safe RL？
3. 应该把主线放在 reward weighting，还是 reliability-aware advantage？
4. Jev 闭源是否会导致复现风险过大？
5. 是否应该一开始就加入 open/local evaluator？
6. 这个方向是否值得投入 2–3 个月做完整验证？

目前个人更倾向于：

> **Calibrated Semantic Feedback Reinforcement Learning：将 Jev 等外部 evaluator 的输出视为带噪反馈，通过 calibration 与 uncertainty-aware trust allocation 动态控制其对 PPO/SAC policy update 的影响，并重点研究 OOD、reward hacking 与机器人任务下的鲁棒性。**


---

## 十一、2026-09-22 最新调研后的方向修正

继续使用 Undermind 对“AI evaluator 只在训练期当老师、最终 policy 独立部署”“少量 Human + 大量 AI Preference”“Human 校准 AI reliability”进行了更窄的检索后，当前结论进一步收缩。

### 已经明确有人做的部分

以下内容不能再作为主要创新：

1. **AI evaluator 只在训练阶段提供 preference/reward，最终 policy 独立部署**
   - RL-VLM-F
   - RLAIF / Direct-RLAIF
   - LAPP
   - Preference VLM

2. **AI Preference 替代 Human Preference**
   - RLAIF 已经系统建立该路线。

3. **少量 Human + 大量 AI Preference**
   - Preference VLM、ROVED、Hybrid Preferences 等已开始使用 hybrid feedback。

4. **uncertain AI sample → Human**
   - Preference VLM / ROVED 已经非常接近。

5. **raw confidence / uncertainty → loss weight**
   - Confidence-Weighted Preference Optimization、Conformal Feedback Alignment、UARM 等已有明确先例。

### 当前最值得继续验证的核心

不是：

\[
c_i^{Jev}
\]

而是：

\[
\rho_i
=
P(
y_i^{Jev}=y_i^{Human}
\mid
x_i,c_i^{Jev}
)
\]

即：

> **Jev 自己有多自信，不等于它有多符合 Human Preference。**

因此现在最推荐的主线是：

> **用少量 Human Preference 把 AI/Jev 的自信度转换成 Human-Aligned sample-wise reliability，再让该 reliability 联合控制 AI preference weighting、Human escalation 和 downstream RL。**

当前建议题目：

> **Human-Calibrated AI Preference Reinforcement Learning**

详细最新状态见：

- [08_Current_Progress_2026-09-22.md](08_Current_Progress_2026-09-22.md)
- [09_Closest_Prior_Work_Map.md](09_Closest_Prior_Work_Map.md)
- [../experiments/01_Human_Calibrated_Jev_POC_v2.md](../experiments/01_Human_Calibrated_Jev_POC_v2.md)

### 最新撞题风险

2026 年已经出现多个高度相关主题：

- Preference-Calibrated Human-in-the-Loop RL for Robotic Manipulation
- TrustRoboReward
- Joint Reward / Worker Reliability Learning
- Multi-Expert Preference Reliability

这些是下一轮必须优先全文核验的工作。

因此当前不是“完全空白的新领域”，而是：

> **2025–2026 正快速从“AI 能否给 feedback”转向“AI feedback 什么时候值得相信”的新窗口。**

实现上第一版 PoC 可控，但论文级难度主要来自：

\[
Calibration
\rightarrow
Better\ Preference
\rightarrow
Better\ Reward\ Model
\rightarrow
Better\ Policy
\]

这条因果链是否能在实验中真正成立。
