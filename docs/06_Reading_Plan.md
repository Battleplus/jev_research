# 06. 文献精读顺序

本清单不是按“时间先后”排列，而是按**对当前课题决策的重要性**排序。目标是先判断方向是否值得做，再补齐方法细节。

## 第一组：先回答“这个方向是不是已经被做完了”

### 1. RL-VLM-F: Reinforcement Learning from Vision Language Foundation Model Feedback
**目的**：理解“外部基础模型反馈 → reward model → RL”的标准范式。  
重点看：
- VLM feedback 如何进入训练循环；
- reward model 如何训练；
- SAC 如何使用 learned reward；
- VLM 不确定样本怎么处理；
- 哪些任务上失败。

**读完要能回答**：
> 我们和 RL-VLM-F 的本质区别是什么？

---

### 2. Navigating Noisy Feedback: Enhancing Reinforcement Learning with Error-Prone Language Models
**目的**：看别人怎么处理 LLM feedback 不可靠的问题。  
重点看：
- 多次查询和 consistency；
- confidence weighting；
- potential-like reward；
- PPO 中怎么使用；
- 理论上为什么错误反馈不会无限放大。

**读完要能回答**：
> “Jev confidence-aware RL”是不是已经被这篇做掉了？

---

### 3. Guiding Reinforcement Learning Using Uncertainty-Aware Large Language Models
**目的**：明确“uncertainty-aware LLM guidance”已有工作做到哪。  
重点看：
- MC-dropout；
- entropy；
- LLM policy 与 PPO policy 的动态混合；
- 它做的是 policy shaping，而不是 reward shaping。

**读完要能回答**：
> 我们的方法为什么不是简单的 uncertainty-aware policy shaping？

---

### 4. RARM: Confidence-Gated Progress Reward Modeling for RL in Manipulation
**目的**：看机器人方向的 confidence gate 已做到什么程度。  
重点看：
- progress reward；
- offline calibrated threshold；
- hard gate；
- robot manipulation setup；
- DSRL / DrQ-v2 训练方式。

**读完要能回答**：
> 我们为什么不能只做 hard confidence gate？

---

### 5. VLM-AR3L
**目的**：了解 absolute / relative reward + confidence consistency。  
重点看：
- absolute reward；
- relative reward；
- bidirectional confidence check；
- SAC / PPO；
- VLM 调用成本怎么压缩。

**读完要能回答**：
> 我们相对“relative reward + confidence check”的增量是什么？

---

## 第二组：确定“真正可以做的创新点”

### 6. Uncertainty-Aware Reward Modeling for Stable RLHF
**目的**：重点学习 uncertainty 如何直接进入 policy update。  
重点看：
- quantile prediction；
- conformal calibration；
- uncertainty interval；
- reliability-weighted advantage。

重点公式：

\[
\tilde A_i
=
\frac{\sigma^2_{signal}}
{\sigma^2_{signal}+\sigma^2_{noise,i}}
\cdot
\frac{r_i-\mu}{\sigma_{signal}}
\]

**读完要能回答**：
> 我们能不能把“reward uncertainty → advantage weighting”迁移到 robot/general RL？

---

### 7. Ask a Strong LLM Judge when Your Reward Model is Uncertain
**目的**：学习 uncertainty routing。  
重点看：
- fast reward model；
- epistemic uncertainty；
- threshold；
- strong judge escalation；
- cost / quality trade-off。

**读完要能回答**：
> Jev 是否适合作为 fast evaluator，strong LLM 是否只在困难样本上调用？

---

### 8. Automating Potential-based Reward Shaping with Vision Language Model Guidance
**目的**：学习如何给 semantic reward 加理论保护。  
重点看：
- VLM feedback 如何构造 potential；
- PBRS；
- policy invariance；
- noisy evaluator 对 optimal policy 的影响。

重点公式：

\[
F(s,s')
=
\gamma\Phi(s')
-
\Phi(s)
\]

**读完要能回答**：
> 我们是否应该把 calibrated semantic feedback 放在 potential learning，而不是直接乘 reward？

---

## 第三组：机器人方向与最终实验设计

### 9. RoboReward
**目的**：看强 robot reward model 的数据、架构和 downstream RL。  
重点看：
- reward score 设计；
- robot dataset；
- real-world；
- reward model quality 与 RL quality 的相关性；
- generalization failure。

---

### 10. Large Reward Models
**目的**：看 online reward model 真正放进 robot PPO 时怎么解决延迟。  
重点看：
- online reward engine；
- interval-hold；
- PPO；
- real robot；
- latency / query frequency。

**读完要能回答**：
> Jev 不可能每 step 调用时，我们的调用策略应该是什么？

---

### 11. DenseReward
**目的**：看 dense robot reward model 的训练与失败数据设计。  
重点看：
- phase decomposition；
- failure synthesis；
- per-step reward；
- PPO / SAC downstream。

---

### 12. Online Preference-based RL with Self-augmented Feedback from LLM
**目的**：看在线 preference RL 中 LLM judge 的实际循环。  
重点看：
- trajectory pair；
- double-check；
- inconsistent feedback filtering；
- SAC；
- self-augmentation。

---

## 第四组：风险与鲁棒性

### 13. Scaling Laws for Reward Model Overoptimization
**目的**：理解为什么“reward 越高”不代表“任务越好”。  
读完要能回答：
> policy 会不会学会骗 Jev？

---

### 14. Reward Model Ensembles Help Mitigate Overoptimization
**目的**：理解 ensemble / uncertainty 如何缓解 reward hacking。

---

### 15. ROBORMBENCH: Same Trajectory, Contradictory Rewards
**目的**：看 semantic reward 的 paraphrase fragility。  
重点指标：
- Score Crossing Rate；
- Flip Rate；
- Mean Error。

建议直接把它的评测方式搬进本课题 robustness 实验。

---

### 16. Learning Robot Safety From Sparse Human Feedback Using Conformal Prediction
**目的**：如果后续转 Safe RL，这篇是很重要的理论/机器人参考。  
重点看：
- conformal prediction；
- sparse feedback；
- safety set；
- robot real-world。

---

# 建议阅读节奏

## 第 1 天
- RL-VLM-F
- Navigating Noisy Feedback
- Uncertainty-Aware LLM Guidance

目标：
> 判断这个课题是不是已经基本被做完。

## 第 2 天
- RARM
- VLM-AR3L
- UARM

目标：
> 确定 novelty 是否应该落在 calibration + policy update。

## 第 3 天
- Strong Judge Routing
- PBRS + VLM
- RoboReward
- Large Reward Models

目标：
> 形成 Method v2 和 Jev 调用架构。

## 第 4 天
- Reward Overoptimization
- ROBORMBENCH
- Conformal Robot Safety

目标：
> 形成 robustness 与安全实验。

---

# 每篇文献统一记录模板

建议每篇只记录 7 个问题：

1. 论文解决什么问题？
2. evaluator 是什么？
3. feedback 是 reward / preference / action / cost 中哪一种？
4. uncertainty / confidence 怎么定义？
5. uncertainty 最终影响 reward、action 还是 gradient？
6. 实验环境是什么？
7. 和 CSF-RL 的差别到底是什么？

不要做大段摘要，优先做“和自己课题的差异表”。
