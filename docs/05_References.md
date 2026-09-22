# 05. 核心参考文献

1. **Deep Reinforcement Learning from Human Preferences** — Christiano et al., 2017  
   https://arxiv.org/abs/1706.03741

2. **RLAIF vs. RLHF: Scaling Reinforcement Learning from Human Feedback with AI Feedback** — Lee et al., 2023  
   https://arxiv.org/abs/2309.00267

3. **RL-VLM-F: Reinforcement Learning from Vision Language Foundation Model Feedback** — Wang et al., ICML 2024  
   https://doi.org/10.48550/arXiv.2402.03681

4. **Vision-Language Models are Zero-Shot Reward Models for Reinforcement Learning** — Rocamonde et al., 2023  
   https://arxiv.org/abs/2310.12921

5. **Text2Reward: Reward Shaping with Language Models for Reinforcement Learning** — Xie et al.  
   https://arxiv.org/abs/2309.11489

6. **Navigating Noisy Feedback: Enhancing Reinforcement Learning with Error-Prone Language Models** — Lin et al., EMNLP 2024  
   https://doi.org/10.48550/arXiv.2410.17389

7. **Guiding Reinforcement Learning Using Uncertainty-Aware Large Language Models** — Shoaeinaeini & Harrison  
   https://doi.org/10.1109/TPS-ISA67132.2025.00045

8. **Ask a Strong LLM Judge when Your Reward Model is Uncertain** — Xu et al., 2025  
   https://doi.org/10.48550/arXiv.2510.20369

9. **Uncertainty-Aware Reward Modeling for Stable RLHF** — Pan et al., 2026  
   https://doi.org/10.48550/arXiv.2606.19818

10. **RoboReward: General-Purpose Vision-Language Reward Models for Robotics** — Lee et al., 2026  
    https://doi.org/10.48550/arXiv.2601.00675

11. **Large Reward Models: Generalizable Online Robot Reward Generation with Vision-Language Models** — Wu et al., 2026  
    https://doi.org/10.48550/arXiv.2603.16065

12. **RARM: Confidence-Gated Progress Reward Modeling for RL in Manipulation** — Yang et al., 2026  
    https://doi.org/10.48550/arXiv.2606.22027

13. **VLM-AR3L: Vision-Language Models for Absolute and Relative Rewards in Reinforcement Learning** — Chen et al., 2026  
    https://doi.org/10.48550/arXiv.2607.00483

14. **Automating Potential-based Reward Shaping with Vision Language Model Guidance** — Müller & Kudenko, 2026  
    https://doi.org/10.48550/arXiv.2606.27180

15. **Scaling Laws for Reward Model Overoptimization** — Gao, Schulman & Hilton, 2022  
    https://arxiv.org/abs/2210.10760

16. **Same Trajectory, Contradictory Rewards (ROBORMBENCH): Paraphrase Fragility in Vision Language Reward Models** — 2026  
    https://arxiv.org/abs/2609.05401

17. **Learning Robot Safety From Sparse Human Feedback Using Conformal Prediction** — Feldman et al., IEEE Transactions on Robotics  
    https://doi.org/10.1109/TRO.2026.3706570

18. **Online Preference-based Reinforcement Learning with Self-augmented Feedback from Large Language Model** — Tu et al., 2024  
    https://doi.org/10.48550/arXiv.2412.16878

19. **Enhancing Rating-Based Reinforcement Learning to Effectively Leverage Feedback from Large Vision-Language Models** — Luu et al., ICML 2025  
    https://doi.org/10.48550/arXiv.2506.12822


## 与 Human Preference / AI Preference 桥接最相关的补充文献

20. **Hybrid Preferences: Learning to Route Instances for Human vs. AI Feedback** — Miranda et al., 2024  
    https://arxiv.org/abs/2410.19133

21. **RIME: Robust Preference-based Reinforcement Learning with Noisy Preferences** — Cheng et al., ICML 2024  
    https://proceedings.mlr.press/

22. **Reward Uncertainty for Exploration in Preference-based Reinforcement Learning** — Liang et al., ICLR 2022  
    https://doi.org/10.48550/arXiv.2205.12401

23. **Direct Language Model Alignment from Online AI Feedback** — Guo et al., 2024  
    https://arxiv.org/

24. **Regret Bounds for Reinforcement Learning from Multi-Source Imperfect Preferences** — Shi et al., 2026  
    https://arxiv.org/abs/2603.20453

25. **Human–AI preference routing / hybrid annotation** should be treated as a direct comparison class when evaluating Jev-based selective escalation.

> 注：20–24 的作用不是单纯扩充 related work，而是明确 novelty 边界：human→AI label replacement、noisy preference robustness、reward uncertainty、online AI feedback、multi-source imperfect preference 都已有先例。新工作需要进一步研究 human-anchored calibration 与 policy-level trust allocation。
