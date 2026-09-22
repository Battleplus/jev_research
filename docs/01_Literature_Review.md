# 01. 相关方向调研与 Novelty 边界

## 1. 初始想法

最初设想是把 Jev 放进 RL 训练过程：

\`\`\`text
Environment
    ↓
trajectory / transition
    ↓
Jev
    ↓
semantic reward / preference / safety signal
    ↓
PPO / SAC
    ↓
Policy
\`\`\`

即：

\[
r_t = r_t^{env} + \lambda r_t^{Jev}
\]

但调研后发现，单纯“AI evaluator 产生 reward，再用 RL 优化”已经有大量工作。

## 2. 已有工作的主要路线

### 2.1 Preference RL / RLAIF

经典 preference-based RL 已经证明可以从 pairwise preference 学习 reward model，再优化 policy。RLAIF 进一步把 human feedback 替换为 AI feedback。

因此：

> “Jev 代替人给 preference”本身不足以成为主要创新。

### 2.2 VLM / LLM 作为 reward source

代表工作包括：

- **RL-VLM-F**：VLM 在线比较状态对，产生 preference label，训练 Bradley–Terry reward model，再使用 SAC 优化策略。
- **Vision-Language Models are Zero-Shot Reward Models for RL**：直接使用视觉-文本相似度等作为 reward。
- **Text2Reward**：LLM 生成可执行 dense reward function，再由 PPO/SAC 训练策略。
- **RoboReward**：通用 robot reward model，直接服务于 downstream robot RL。
- **DenseReward**：学习 per-timestep dense reward，用于 PPO / SAC robot policy fine-tuning。
- **Large Reward Models**：VLM 作为 frozen online reward engine，在 robot RL 中周期性生成 reward。

因此：

> “foundation model 给机器人 RL reward”已经是比较成熟的研究方向。

## 3. 与本课题最接近的 uncertainty / confidence 方向

### 3.1 Navigating Noisy Feedback

该工作针对 error-prone LLM feedback：

- 多次查询 LLM；
- 用输出一致性反映 reliability；
- 训练 state-score model；
- 使用 score difference 作为 potential-like shaping reward；
- PPO 在 GridWorld / MuJoCo 上训练。

### 3.2 Guiding RL Using Uncertainty-Aware LLMs

该工作：

- MC Dropout 得到 LLM action distribution；
- entropy 作为 uncertainty；
- 动态混合 LLM policy 与 PPO policy：

\[
P_t(a)
=
(1-H_t)P_{LLM}(a)
+
H_tP_{agent}(a)
\]

这是 **policy shaping**，不是 reward shaping。

### 3.3 RARM

RARM 使用：

- reference trajectory；
- visual similarity；
- 离线 calibration threshold；
- 只有超过 confidence threshold 时才推进 progress reward。

### 3.4 VLM-AR3L

同时使用：

- absolute reward；
- relative reward；
- symmetric confidence check；
- 低可信 relative reward 直接设为 0。

### 3.5 UARM

Uncertainty-Aware Reward Modeling 使用：

- quantile reward prediction；
- conformal calibration；
- uncertainty interval width；
- reliability-weighted advantage。

核心形式类似：

\[
\tilde A_i
=
\frac{\sigma^2_{signal}}
{\sigma^2_{signal}+\sigma^2_{noise,i}}
\cdot
\frac{r_i-\mu}{\sigma_{signal}}
\]

### 3.6 Ask a Strong LLM Judge when Your Reward Model is Uncertain

该工作：

- fast reward model 先判断；
- epistemic uncertainty 高时；
- 才升级给 stronger LLM judge；
- 用于在线 RLHF / policy optimization。

## 4. 仍然存在的研究空位

因此，本课题不应定义为：

> Jev + PPO

而应该定义为：

> **Calibrated, policy-aware trust allocation for semantic feedback in RL**

即：

1. 把 semantic evaluator 看作 **带噪 measurement**；
2. 对 evaluator confidence 做 task-specific calibration；
3. 动态控制 semantic feedback 对 policy update 的贡献；
4. 遇到 OOD / low confidence 时降权或升级 strong judge；
5. 重点验证最终 policy，而不是 evaluator 自身精度；
6. 专门研究 reward hacking、OOD、paraphrase fragility。

## 5. 需要重点防止的失败模式

### Reward hacking / overoptimization

若 RL 直接最大化：

\[
r^{semantic}
\]

agent 可能学会 exploit evaluator，而不是真正完成任务。

### Paraphrase fragility

ROBORMBENCH 表明，同一 trajectory 仅改变语义等价的任务描述，VLM reward 可能发生明显翻转。

建议加入：

\[
\text{same trajectory}
+
\text{equivalent paraphrases}
\]

测试 reward variance / flip rate。

### OOD / embodiment shift

需要测试：

- viewpoint shift；
- background shift；
- object appearance shift；
- robot embodiment shift；
- unseen task composition。

## 6. 当前最值得做的 novelty

最建议聚焦：

> **calibrated evaluator uncertainty 如何同时控制 reward weighting、advantage weighting 和 evaluator escalation，并证明这种 trust allocation 在 embodied RL 的 OOD / reward hacking 场景下优于 raw reward、hard gate 和 fixed-weight shaping。**
