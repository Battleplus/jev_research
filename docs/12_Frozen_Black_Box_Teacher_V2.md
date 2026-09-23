# 12. Frozen Black-Box AI Teacher — V2 Research Route

更新时间：2026-09-23

## 1. 为什么需要这次路线修正

原始构思是：

[
Jev / AI Evaluator
ightarrow
Preference / Reward
ightarrow
PPO/SAC
]

随后逐步加入：

[
Human Preference
ightarrow
AI Reliability Calibration
ightarrow
Reward Model
ightarrow
Preference	ext{-}Based RL
]

新的关键约束是：

> Jev、ChatGPT 等强 evaluator 可能是 **closed black-box models**。如果厂商没有提供 fine-tuning / post-training 接口，则研究者无法直接更新模型本体参数。

因此，论文主线不应该建立在：

[
Human Feedback
ightarrow
Post	ext{-}train Jev/GPT
]

上。

更合理的研究设定是：

[
oxed{
Frozen Black	ext{-}box AI Teacher
}
]

并研究：

> **在不能修改 AI teacher 参数的前提下，如何用少量 Human Preference 学习一个可训练的外部反馈适配层，使 AI feedback 更接近 Human objective，并可靠地进入 downstream Preference RL。**

---

## 2. 这次 Undermind 专项检索

专项 Deep Search：

> **Frozen closed AI evaluator feedback adaptation for RL**

共返回 **236 篇相关工作**。

重点问题包括：

1. frozen / closed LLM 或 VLM 作为 RL teacher / judge / reward source；
2. 不修改 foundation model 权重时的 adaptation；
3. small Human labels + black-box AI；
4. local surrogate / reward model / student distillation；
5. selective escalation；
6. sample-wise reliability / correction；
7. policy-induced distribution shift；
8. robotics / embodied RL。

本轮全文核验重点包括：

- RL-VLM-F
- LAPP
- Preference VLM
- ROVED
- BACON
- Trust or Escalate
- Aligning Black-box Language Models with Human Judgments
- Demo2Reward
- VARP
- Training Fast Robot Policies with Slow Foundation Models

---

## 3. 首要结论：闭源 teacher 不需要后训练才能做 RL

文献已经明确证明：

[
Frozen Foundation Model
ightarrow
AI Feedback
ightarrow
Local Reward/Preference Model
ightarrow
RL
ightarrow
Standalone Policy
]

是成立的。

### RL-VLM-F

- GPT-4V / Gemini-Pro 作为冻结 VLM；
- trajectory/image pair → VLM preference；
- preference → Bradley-Terry reward model；
- reward model → SAC；
- foundation model 不更新；
- 最终 policy 独立部署。

因此：

> **“closed/frozen evaluator 无法 fine-tune”并不意味着 AI-assisted PbRL 不成立。**

### LAPP

- GPT-4o-mini 作为 frozen black-box LLM；
- structured robot trajectory logs → LLM preference；
- 本地 Transformer reward/preference predictor 学习；
- PPO 训练 robot policy；
- 真机 Unitree Go2 部署不再依赖 LLM。

这与 Jev 的潜在使用方式非常接近。

---

## 4. 但哪些内容已经不能作为 novelty

### 4.1 Frozen AI → Preference → RL

已被 RL-VLM-F、LAPP、Preference VLM 等覆盖。

因此：

> 不能把“Jev 作为闭源 teacher 给 RL preference”作为核心创新。

### 4.2 AI + 少量 Human + uncertain → Human

Preference VLM、ROVED、Hybrid Preferences 等已有明确先例。

因此：

> “AI 不确定时问人”不新。

### 4.3 Sample-wise filtering / correction

Preference VLM / ROVED 已经使用 reward model 与 VLM label 的 KL divergence 对单个 trajectory pair 分成：

- clean；
- noisy；
- uncertain。

其中：

- uncertain → Oracle/Human；
- noisy → label flipping；
- clean → 直接使用。

因此：

> “sample-wise AI feedback filtering/correction”本身也不新。

### 4.4 Frozen black-box + small Human → external correction

**Aligning Black-box Language Models with Human Judgments** 已经做到：

[
Black	ext{-}box LLM categorical judgment
+
small Human calibration
ightarrow
external mapping W
ightarrow
human	ext{-}aligned judgment
]

特点：

- black-box LM 完全冻结；
- 无需 logits / weights；
- 20–100 个 Human samples 即可拟合外部映射；
- 但属于静态 evaluation；
- 不做 downstream RL；
- 不做 on-policy / policy-shift adaptation。

因此：

> “不改闭源模型，在外面加 Human-trained correction mapping”这个大概念也已经有 prior。

### 4.5 Frozen model + prompt adaptation

Demo2Reward 已经做：

[
Frozen VLM
+
few expert demonstrations
ightarrow
test	ext{-}time prompt optimization
ightarrow
better reward
ightarrow
robot RL
]

它不训练 VLM 权重，而是优化 prompt 来减少 false positive / reward hacking。

因此：

> “不改模型参数，优化 teacher 的使用方式”也已有工作。

### 4.6 Policy evolution / policy-distribution-aware reward learning

VARP 已经研究：

> reward model 应随 agent policy 的 evolving behaviors 保持一致。

其 agent-aware regularization 直接依赖当前 policy trajectory。

因此不能笼统声称：

> “第一次考虑 policy distribution shift。”

---

## 5. 当前真正值得守住的 gap

我们需要把问题进一步收紧为：

[
oxed{
	extbf{Policy-Conditional Human-Aligned Feedback Correction for Frozen AI Teachers}
}
]

核心不是：

[
	ext{Can AI give feedback?}
]

也不是：

[
	ext{Can humans calibrate an AI judge?}
]

而是：

> **当一个 frozen black-box AI teacher 面对不断变化的 on-policy trajectory distribution 时，它与 Human Preference 的 agreement 是否发生 context-dependent / policy-induced shift？能否用少量 Human Preference 学习一个 sample-wise correction + reliability model，并让该模型持续控制 downstream PbRL？**

---

## 6. 从 Reliability Calibration 升级为 Feedback Adaptation

之前主要学习：

[
ho_i
=
P(
y_i^{AI}=y_i^{Human}
mid
x_i,c_i^{AI}
)
]

现在建议升级为一个外部可训练模块：

[
oxed{
F_phi
(
	au_i^A,	au_i^B,
y_i^{AI},
c_i^{AI},
d_i,
o_i,
t_i
)
ightarrow
(
hat y_i^{H},
ho_i
)
}
]

其中：

[
hat y_i^{H}
=
	ext{estimated human-aligned preference}
]

[
ho_i
=
P(
y_i^{AI}=y_i^{Human}
mid
trajectory/context/policy stage
)
]

这意味着系统不仅回答：

> “AI 这次可信不可信？”

还进一步回答：

> “如果 AI 在这个 context 下具有系统性偏差，Human 更可能选哪一个？”

因此方法从：

[
Reliability Estimation
]

升级为：

[
oxed{
Human	ext{-}Aligned Feedback Adaptation
}
]

注意：

> **Adaptation 的对象是 feedback/interface，不是 frozen AI teacher 的参数。**

---

## 7. 推荐最终训练结构

[
oxed{
Trajectory Pair
ightarrow
Frozen Jev/GPT/VLM
ightarrow
Raw AI Feedback
ightarrow
Human	ext{-}Aligned Feedback Adapter
ightarrow
Reward Model
ightarrow
SAC/PPO
ightarrow
Standalone Policy
}
]

Human Preference 只需提供一个小型 trusted anchor set。

### Trainable components

[
F_phi
]

Human-aligned feedback adapter / reliability model。

[
R_psi
]

Preference reward model。

[
pi_	heta
]

RL policy。

### Frozen component

[
AI Teacher
]

如 Jev / GPT / VLM。

因此：

[
oxed{
	heta_{AI} 	ext{fixed}
}
]

不是临时权宜，而是研究设定。

---

## 8. 三种使用 (ho_i) 的方式

### 8.1 Preference weighting

[
mathcal L_{RM}
=
sum_{iin D_H}
CE(P_psi,y_i^H)
+
lambda
sum_{jin D_{AI}}
ho_j
CE(P_psi,hat y_j)
]

### 8.2 Feedback correction

当模型识别出 AI 有 systematic bias 时：

[
y_j^{AI}
ightarrow
hat y_j^H
]

而不是仅仅：

[
y_j^{AI}
	imes ho_j
]

### 8.3 Selective Human escalation

[
ho_j > 	au_h
Rightarrow
Accept
]

[
	au_l < ho_j le 	au_h
Rightarrow
Correct/Downweight
]

[
ho_j le 	au_l
Rightarrow
Query Human
]

---

## 9. 最关键的新假设：Policy-Conditional AI–Human Agreement

RL policy 不断变化：

[
pi_0
ightarrow
pi_1
ightarrow
cdots
ightarrow
pi_T
]

因此 trajectory distribution 变化：

[
d^{pi_0}(	au)

eq
d^{pi_T}(	au)
]

我们真正要验证的不是笼统的 reward-model distribution shift，而是：

[
oxed{
P(
y^{AI}=y^H
mid
d^{pi_0}
)

eq
P(
y^{AI}=y^H
mid
d^{pi_T}
)
}
]

即：

[
oxed{
Policy	ext{-}Induced AI	ext{-}Human Agreement Shift
}
]

例如：

- early policy：失败轨迹差异大，AI 容易判断；
- late policy：轨迹都接近成功，区别落在稳定性、安全性、效率等细节上；
- AI confidence 可能仍然很高；
- 实际 Human agreement 却下降。

这必须通过实验验证，不能预设成立。

---

## 10. 与最危险 prior work 的区别

| Prior work | 已做 | 当前需要保留的差别 |
|---|---|---|
| RL-VLM-F | Frozen VLM → preference → RM → SAC | 无 Human-aligned correction/reliability |
| LAPP | Closed LLM → local preference predictor → robot PPO | 无 Human anchor / AI-human reliability model |
| Preference VLM | sample-wise clean/noisy/uncertain + Human + adapter + SAC | 主要以 RM↔VLM KL disagreement 判 noise；不是显式 (P(AI=Humanmid x,pi_t)) |
| ROVED | uncertainty filtering + oracle + VLE adaptation + PbRL | teacher 可适配；不是 frozen black-box Human-aligned agreement model |
| Trust or Escalate | frozen judge + calibrated Human agreement + escalation | 无 downstream PbRL |
| BACON | small Human + AI judge + context → sample-wise human score | 静态 evaluation，无 policy/RL loop |
| Black-box Human Judgments | frozen black-box + small Human → external label mapping | global/static mapping，无 sample-wise trajectory/context/policy stage |
| Demo2Reward | frozen VLM + few demos → task-level prompt optimization → RL | task-level prompt，不是 sample-wise Human preference correction |
| VARP | frozen GPT-4o + policy-aware reward regularization | 无 Human calibration / explicit AI–Human agreement |
| ROVED/PrefVLM | distribution-shift handling | 不是以 Human agreement drift 为 calibration target |

因此最安全的 novelty 边界是：

[
oxed{
Human	ext{-}Anchored
+
Frozen Black	ext{-}box AI
+
Sample/Context	ext{-}wise Correction
+
Policy	ext{-}Conditional Reliability
+
Online Recalibration
+
Downstream Robot PbRL
}
]

不是上述任意单个模块的“首次”。

---

## 11. 新版最小 PoC

不要先训练完整方法。

第一步先验证核心现象。

### Tasks

建议先选 3–4 个 MetaWorld manipulation tasks：

- Drawer Open
- Button Press
- Pick Place
- Sweep Into

### Policy stages

至少采：

[
pi_{random},
pi_{early},
pi_{mid},
pi_{late}
]

### 每个 pair 保存

[
(
	au_A,
	au_B,
y_H,
y_{AI},
c_{AI},
task,
difficulty,
OOD,
policy stage
)
]

### 第一轮规模

[
400sim800
]

个 trajectory pairs 更适合检查 stage effect。

如果 Human 标注压力较大，可以先：

[
200sim400
]

做 pilot。

### 首先比较

1. Raw AI confidence
2. Global/static calibration
3. Context-aware calibration
4. Policy-stage-aware calibration
5. Feedback correction model

### 核心指标

- AI–Human Agreement Accuracy
- ECE
- Brier Score
- NLL
- Risk-Coverage
- Correction Accuracy
- Human Query Budget
- 后续 Policy Success Rate

---

## 12. Go / No-Go

### Go 1

AI preference 必须明显优于随机。

否则 teacher 本身不具备基本使用价值。

### Go 2

必须出现以下至少一种现象：

- raw confidence 与 Human agreement 不匹配；
- systematic context-dependent bias；
- agreement 随 policy stage 漂移；
- static mapping 在 later-policy data 上退化。

### Go 3

外部 Human-aligned adapter 必须比：

- raw AI；
- naïve pooling；
- global calibration；
- RM↔AI disagreement filtering；

更好地预测 Human preference / agreement。

### Go 4

最终必须证明：

[
Better Feedback
ightarrow
Better Reward Model
ightarrow
Better Policy
]

如果只改善 evaluator metric 而 policy success 不提高，则论文贡献明显减弱。

---

## 13. 当前研究定位

建议项目主线从：

> **Jev-Assisted RL**

升级为：

> **Human-Aligned Adaptation of Frozen AI Feedback for Preference-Based Reinforcement Learning**

更具体的论文方向：

> **Policy-Conditional Human-Aligned Feedback Correction for Frozen AI Teachers in Preference-Based Reinforcement Learning**

Jev 的角色改为：

> **one frozen black-box evaluator backend / case study**

而不是：

> the algorithm itself。

这样即使：

- Jev 无 fine-tuning API；
- Jev 后续版本发生变化；
- Jev 效果不够强；
- 未来换成 GPT / Gemini / VLM / local judge；

论文核心问题仍然成立。

---

## 14. 当前判断

本轮 236 篇专项检索 + 重点全文核验后：

> **“闭源模型不能后训练”不是该方向的致命问题，反而应该成为研究设定的一部分。**

真正需要研究的是：

[
oxed{
	ext{How to adapt the feedback of a frozen AI teacher, not its weights.}
}
]

当前最值得验证的核心现象：

[
oxed{
P(AI=Humanmid trajectory,pi_t)
 	ext{是否随 policy/context 系统变化}
}
]

如果该现象成立，则后续：

[
Human	ext{-}Aligned Feedback Adapter
+
Online Recalibration
+
PbRL
]

具有继续形成完整论文的合理性。

如果该现象不成立，则应及时弱化 policy-shift 故事，转向：

- systematic feedback correction；
- reward hacking robustness；
- OOD correction；
- human-budget-efficient teacher adaptation。

