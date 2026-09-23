# 16. Undermind 最新近邻核验与实验修订建议

核验日期：2026-09-23（Asia/Shanghai）。基线：`65fb943d57ed7c87b2fd297bb1656c6a1ce0400d`。

**结论：Weak-Go，保留 frozen teacher → 外部 Human-aligned adapter → RM → PbRL → standalone policy，收缩现象与方法的创新主张。** 本轮没有找到经全文证据确认、完整覆盖目标链路的单篇论文；这不是“确定首次”。已经找到足以否定宽泛现象首创表述的直接先例：Metcalf et al., AAAI 2024 已测量 human–synthetic label agreement 随 policy training 的变化，只是 synthetic labeler 是环境奖励 oracle，而非 LLM/VLM。最有价值的增量应是：**对固定 AI 的人类偏好误差进行可复现的跨策略分布测量，在控制任务/难度/标注选择后，用等人工预算的在线外部纠正改善独立测试集、RM 和最终机器人策略。**

本文是 evidence audit 与 change proposal；不替代 [docs/13](13_Final_Research_Execution_Plan.md) 的执行基线或 [docs/14](14_Publication_Timeline_Roadmap.md) 的时间/投稿基线。[docs/15](15_Jev_Article_Research_Implications.md) 的 full probability、统一分布、oracle leakage 防护继续适用。未实施实验，也没有把预测收益写成实验结果。

## 1. 实际检索范围与可追溯性

已按要求阅读 README、docs/12–15、docs/09–11、experiments/01，再阅读 docs/01–08 和 experiments/00。较早文档中的 Jev reward shaping/advantage 改动是历史方案，不作为本轮基线。

实际调用的是 **Undermind MCP**，认证成功，并非普通网页搜索替代：

- 检查已有六次 Deep Search；复核其中 frozen-evaluator 236 篇及 sample-wise reliability 70 篇的历史结果。它们不是本轮新检索数量。
- 新启动并完成一次 Deep Search：**Policy evolving human agreement of frozen robot preference evaluators**；服务端记录 2026-09-23 07:56–07:58 UTC，返回 **164 篇候选**。这是摘要/元数据层的候选集合，不是 164 篇全文核验。
- 分别执行 Q1–Q10 十个 semantic queries，每项 `year_min=2024, year_max=2026, limit=10`；另做 2026 最新排序与 system-one/Jev 专项补搜。具体提交文本见附录 A。
- 对用户列出的 16 篇危险近邻全部重新定位和全文查询；按两组种子分别执行 citing papers 与 references 检索（每组每方向取 25 条）。搜索结果有交叠，不能把各项数量相加当独立论文数。
- 对作者后续工作调用 `find_papers_by_author`。确认 PrefVLM → ROVED 为同一团队的直接延续；作者 ID 后续查询发生超时，作者作品清单覆盖不完整。
- 对 **31 篇不同论文**调用 `read_pdfs`，并对 PrefVLM、ROVED、Metcalf、Calibrating the Evaluator 追加针对性全文核查。下载并本地抽查 PrefVLM、ROVED、Metcalf、BACON、TrustRoboReward 五篇原始 PDF，纠正自动摘要的概念误判。

[打开本轮 Undermind Deep Search](https://app.undermind.ai/projects/016486a2-aaad-4ee4-af73-3fae5bccad92?path=/02_Reinforcement_Learning/03_Jev_Assisted_RL/Policy%20evolving%20human%20agreement%20of%20frozen%20robot%20preference%20evaluators)。

**日期与版本限制：** 检索重点是 2024–2026，Deep Search 也保留必要的更早基础工作。最近六个月及 2026-09 的候选单独按日期检查；最新入选全文之一为 2026-09 的 PreferenceEKF。`year_max=2026` 不等于精确截止日过滤，入选证据的可见日期均不晚于本次核验日。部分数据库日期与 PDF/arXiv 编号月份不一致（例如 BACON、Demo2Reward），本文不把数据库日字段当作正式首发日。

### 检索故障与版本核验边界

1. 16 条书目批量定位返回 `internal_error: No papers resolved`，并明确提示 lookup backend 部分不可用；随后通过已有库、语义检索和 `get_paper_info` 完成定位。
2. 引用网络接口实际每次最多接受 10 个种子，首次 14 个种子请求被校验拒绝；拆为两组各 8 个后成功，包含全部 16 篇。
3. 作者后续查询超时，不能声称“没有后续工作”。
4. 本机直接访问 24 个 arXiv landing pages 进行 version-history/链接验证均 `ReadTimeout`；因此“Undermind 当前可读 PDF 的版本”不等于“已确认 arXiv 最新版本”。所有 31 篇入选论文都获得全文工具答复，但最新修订、正式接收和作者全集仍有缺口。
5. 未观察到认证失败或额度不足。以上故障不影响已完成的 Deep Search，但限制查全和版本结论。

## 2. 本轮新增、升级为重点的工作

“新增”指本轮相对仓库主要近邻清单的新纳入或证据升级，不表示刚发表，也不表示此前 Undermind 从未返回。

| 工作 | 新增的重要证据 | 对本项目的影响 |
|---|---|---|
| [Metcalf 2024][Met24b] | Fig. 4 直接画出 human–synthetic matching 随反馈轮次变化；PDF pp. 3–5 定义环境奖励 oracle、真实 crowd 和策略训练 | 必须放在核心问题引入中；撤回宽泛的 agreement-by-stage 首创暗示 |
| [DA-RAC][Wu26f]，2026-08 | §3.4 将邻域样本的 vanilla-judge/human agreement 加权为预测概率 | “context-wise human agreement estimator”本身已有非常直接的近邻，需实现轻量邻域 baseline |
| [UrbanAlign][Zha26j]，2026 | §3.5 局部 ridge、§4.4 修正原判断，frozen VLM + 真实人类偏好 | 外部 contextual correction 不能仅靠“非全局映射”声称创新 |
| [PrefMoE][Yua26]，2026 | 轨迹级 MoE reward routing；真实人类数据；offline IQL | 需要区分 evaluator error 与人类偏好异质性；更复杂 RM 可能替代 adapter 收益 |
| [Calibrating the Evaluator][Liu26d]，2026 | §3.2 sliding-window isotonic calibration，策略权重在线调整 | online evaluator calibration 已有相邻先例；但 `binary_outcome` 来源未明确，不能宣称已证明 human alignment |
| [PromptShift-CRC][Opo26]，2026 | 上下文加权与在线 risk update，§5–6、§9 | drift-aware calibration 不新；需说明反馈标签可得性和预算，与纯阈值风险控制区分 |
| [PreferenceEKF][Zho26b]，2026-09 | pool-based active preference querying、RM → offline IQL；§5.5、App. A.3 | 最新强 active-learning 比较类；没有 frozen AI correctness adapter |
| [Localize-Then-Decide][Li26i]、[Multi-Expert CRC][Che26k]，2026-08 | human-calibrated shortlist/选择性风险保证；静态评测 | 不能把集合风险保证误写为逐样本 correctness probability，也不能宣称“首次 human-calibrated judge cascade” |
| [Disagreement Prediction][Eha26]，2026 | 无生成概率的几何一致性指标预测人类分歧 | 必须比较 context-only/consistency-only；AI probability vector 不是必需的信息优势 |

另全文排查 [Reliability-Aware LLM Alignment][Hua26d]、[Deployable Human Preference Alignment][Kim26e]：前者针对人类标注者混淆矩阵与 DPO，后者针对离线偏好聚类与 IQL。它们不能被标题中的 reliability/frozen encoder 误认作 frozen AI evaluator correction。

## 3. 最危险的十篇工作

排序依据是对当前 **具体 claim** 的威胁，不是引用数或论文本身质量。

| 排名 | 工作 | 最危险的重合 | 经核验仍缺少的关键环 |
|---:|---|---|---|
| 1 | [Can You Rely on Synthetic Labellers…][Met24b] | 已有随训练阶段变化的 human–synthetic agreement 实测及 RM/SAC | synthetic 是手写环境奖励；无 frozen AI 概率输出、人类纠正 adapter、在线重校准比较 |
| 2 | [ROVED][Gho26] | 少量 oracle、逐样本 filtering/flipping、预算查询、embedding adapter、RM/SAC | label-output-only 黑盒设定、人类 agreement 概率目标、等预算 static/online calibration |
| 3 | [Preference VLM][Gho25] | frozen encoder + adapter + 初始 anchor + query budget + 下游机器人 PbRL，已有 policy-shift 讨论 | 需访问 embedding；KL proxy 非 calibrated AI–Human correctness；真人实验来源未明确 |
| 4 | [BACON][Shi26b] | 小人工预算、AI outputs + uncertainty + context → human outcome distribution/score | 静态评分/统计推断；无 evolving robot policy、RM/SAC 闭环 |
| 5 | [DA-RAC][Wu26f] | 人类参考样本、context 邻域、逐样本 agreement 概率近邻估计 | 静态 judge auditing；无 robot RM/RL、等预算在线重校准 |
| 6 | [Feature Dependent Noise in PbRL][Li26f] | 明确 trajectory-feature-dependent noise；LM feedback；下游 PbRL | 标签基准为 simulator oracle；无真实人类 anchor 与动态 agreement correction |
| 7 | [RLTHF][Xu25b] | frozen AI 初标、RM-based错误识别、翻转/定向人工纠正、预算与 DPO | 静态文本池的迭代纠正；无 robot trajectory drift 或显式 calibrated correctness target |
| 8 | [Aligning Black-box Language Models…][Bur25] | frozen 黑盒、少量人类标签、外部修正映射 | per-task categorical mapping；无轨迹条件动态重校准或 RL |
| 9 | [PromptShift-CRC][Opo26] | 外部上下文/漂移感知 calibration，在线阈值调整 | 非机器人 policy-induced 数据、非 fixed-human-budget correction → RM → RL |
| 10 | [TrustRoboReward][Wan26l] | teacher + 人工冲突仲裁 + 数据修正 + robot reward model + 下游优化 | 修正 score/pair consistency；下游为视频生成，未证实动作控制 PbRL；无目标 agreement 概率/在线预算实验 |

[VARP][Sin25]、[Trust or Escalate][Jun24]、[Calibrate, Don’t Curate][Li26d]、[TriTrust][Hos26] 同样是必要对照，排在十篇之外不代表可以跳过。

## 4. 逐篇证据表

统一判定规则：**有** = 所读方法/实验存在；**无** = 所读方法/实验没有该组件；**未明** = 正文未明确，不能转写为没有；**未取全文** = 无法验证。31 篇本轮均有全文工具答复，不存在用摘要填充全文状态的行。表中“无”不意味着已证明所有未来版本永远不会加入该组件。

- `Frozen AI` 指外部评价器冻结，不能把冻结 feature encoder 自动算成 AI judge。
- `ρ` 指 human-anchored correctness 概率；从完整 human preference distribution 可导出，但不等于论文已将其用作 reliability 控制。
- `Policy Shift` 区分 on-policy 数据更新、agreement-by-stage 实测和 difficulty-controlled conditional drift。
- `Online` 指在线 **agreement recalibration**；只更新 RM/embedding 不算。
- `Escalation` 指实际路由/升级；只扫描标注预算或做静态采样不算。
- `RM/RL` 区分 judge 训练、文本 DPO、视频生成优化和机器人动作策略。
- `独立部署` 只判断最终行为策略能否脱离评价器；静态 evaluator 论文记“不适用”，不能把不再问人写成不再调用 evaluator。

### 4.1 用户指定的十六篇

| Title | Year/Venue | URL/DOI | Frozen AI | Human Anchor | Sample-wise Reliability / ρ | Policy Shift | Online | Correction | Escalation | RM/RL | Robot | 独立部署 | 与本项目重合度 | 未覆盖部分 | 证据位置 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Preference VLM | 2025 preprint，所读 v1 | [Gho25] | encoder冻结；需embedding | 方法有；真人实验来源未明 | KL逐样本proxy；非ρ | policy/data shift讨论 | 更新G，不是ρ重校准 | flipping+adapter | 有预算查询 | RM+SAC | MetaWorld | actor结构支持 | 很高 | 黑盒输出接口、ρ及公平static/online | §4.2–4.3 Eq.5–9；§5.4 Fig.8；App.A/B |
| ROVED | 2026 preprint，所读v1；正式venue未核实 | [Gho26] | encoder冻结；需embedding | oracle；真人未证实，Gemini实验明确 | KL逐样本proxy；非ρ | evolving buffer | 更新G/动力学头 | flipping+adapter | 有oracle预算 | RM+SAC | MetaWorld 8任务 | actor结构支持 | 很高 | 真实human anchor、输出黑盒、ρ目标 | §IV-B/C/D，§V-A/B pp.4–6 |
| Trust or Escalate | 2024 preprint，所读v1 | [Jun24] | 有 | 有，静态人类数据 | confidence+接受集合风险保证；非逐点ρ保证 | 跨数据/模型shift测试；非policy演化 | 无 | abstain，不修正原标签 | judge cascade；非等预算在线human | 无下游RM/RL | 无 | 不适用 | 高，上游 | 动态robot学习闭环 | §2 Eq.1–2；§2.3；§3.4 |
| BACON | 2026 preprint，所读v1 | [Shi26b] | 有 | 有，评分/评测数据 | contextual human outcome；分类形式可导出ρ | 静态 | 周期校准为讨论，未比较 | 外部human score correction | 有预算采样；非RL escalation | 无下游RM/RL | 无 | 不适用 | 很高，上游 | policy演化与下游闭环 | §2.2–2.3 pp.4–5，Table1；App.D |
| Aligning Black-box LMs | 2025 preprint，所读v1 | [Bur25] | 有；无需logits | 有，Judge-Bench | per-task W；非context ρ | 无 | 无 | 有，categorical mapping | 无 | 无 | 无 | 不适用 | 高，上游 | contextual/dynamic/RL | §2.1–2.4 Eq.2–3；§3 |
| TriTrust-PBRL | 2026 preprint，所读v1 | [Hos26] | 非AI teacher设定 | synthetic experts，非human gold | source-wise α；非ρ | policy训练；非agreement实测 | 学α，不是human recalibration | 负trust反向利用 | budget sweep，非escalation | RM+SAC | MetaWorld/DMC | actor结构支持 | 高，RM | human anchor、sample-conditioned AI | §3.1–3.2；§5.1；§6限制 |
| Feature Dependent Noise | 2026 preprint，所读v2 | [Li26f] | LM实验有 | environment oracle | feature-wise噪声模型；非human ρ | 部分噪声随模型变化 | 无human recalibration | 评测RIME等纠错 | 无human escalation | RM+SAC类PbRL | MetaWorld/DMC | actor结构支持 | 很高，现象/噪声 | real-human动态纠正 | §3 Eq.3–4；§4.2；§5 |
| VARP | 2025 preprint，所读v1 | [Sin25] | GPT-4o | 无；oracle作对照 | pair-difference准确率分析；非ρ | agent-aware reward regularization | 无human recalibration | sketches改善输入；不修正标签 | 无 | RM+SAC | MetaWorld/DMC | actor结构支持 | 高，policy相关 | human correction目标 | §IV Eq.5–6；§V；Fig.3 |
| Demo2Reward | 2026 preprint，所读v1 | [Gum26] | 有，改prompt | success标签/demos；非pairwise anchor | task-level TPR/TNR；非ρ | prompt在RL前优化 | 无 | task prompt优化 | 无 | VLM reward+IBRL/RLPD；非所提独立PbRL RM | 仿真+Franka | 策略部署结构支持 | 中高 | pairwise human distribution与online | §3.1、§3.3；§4.6；App.B.1 |
| LAPP | 2025；元数据TMLR，所读v1仍under review | [Jia25] | GPT-4o-mini | 无pairwise human anchor | 全局ε噪声假设；非ρ | 在线更新优于static predictor | 非human recalibration | 多次采样取mode；非human纠正 | 无 | Transformer RM+PPO | Go2/Shadow Hand | 有真机部署 | 高，结构轨迹 | human anchor+ρ+预算 | §3 Eq.4；§4.1–4.4；§5.4 |
| RL-VLM-F | ICML 2024；所读v4 | [Wan24] | GPT-4V/Gemini | 无；oracle对照 | 难度分箱准确率；非ρ | 在线收集；非human drift | 无 | unsure过滤 | 无human escalation | BT RM+SAC | MetaWorld/SoftGym等 | actor结构支持 | 高，标准链路 | human correction/recalibration | §5 Alg.1；§6.3 Fig.6；App.B |
| Judging with Confidence | 2025 preprint，所读v1 | [Li25j] | 无；全参数训练autorater | 训练为persona AI；PandaLM真人测试 | 预测偏好分布；可导出匹配概率 | 无下游policy演化 | 无 | 学分布；非外部黑盒修正 | label-efficiency，非escalation | SFT/GRPO训练judge；无下游策略 | 无 | 不适用 | 高，概率目标 | frozen external+robot loop | §2.1–3；§4；§5.3；App.C.1 |
| Calibrate, Don’t Curate | 2026 preprint，所读v1 | [Li26d] | 有 | benchmark human/客观标签混合 | source accuracy+contextual p(y)；非独立ρ模块 | 静态 | 无 | residual/beta correction | calibration-budget sweep；非active escalation | judge aggregation；无策略RL | 无 | 不适用 | 高，上游 | dynamic robot loop | §2 Eq.3；§2.5；§4.5；App.H |
| Hybrid Preferences | 2024首稿；所读2025 v5 | [Mir24] | 有 | MULTIPREF真实标注 | router预测RM表现，非ρ | 静态数据构建 | 无 | source routing，不是label mapping | 有human比例预算 | RM评测+文本DPO | 无 | 文本policy支持 | 高，预算 | trajectory shift+external correction | §2.2–2.3；§3–4；App.J Table18 |
| RLTHF | 2025；PDF列ICML；所读v3 | [Xu25b] | 有 | HH-RLHF/TL;DR已有human labels | reward-margin错误识别，非calibrated ρ | fixed池迭代非新policy rollout | 无目标static/online对照 | 有flipping+人工更正 | 有targeted human预算 | RM+文本DPO；PPO为可用途 | 无 | 文本policy支持 | 很高，方法 | dynamic robot distributions+ρ | §3.2–3.3；§4.2；App.Alg.1 |
| TrustRoboReward | 2026 preprint，所读v1 | [Wan26l] | GPT-5-mini teacher | cycle仲裁+gold evaluation | score-order consistency；非ρ | 无agreement-by-stage | 无 | POISE编辑pointwise score | 冲突仲裁；无预算曲线 | judge SFT/GRPO；下游DiffusionNFT | 机器人视频；非已证实动作控制 | robot policy未证实 | 高，但链路须纠偏 | 动作PbRL、ρ、online/fixed budget | §3.2；§4.2 p.9；App.E.1 Table6/E.2 Table7 |

### 4.2 新纳入或重新升级的十五篇

| Title | Year/Venue | URL/DOI | Frozen AI | Human Anchor | Sample-wise Reliability / ρ | Policy Shift | Online | Correction | Escalation | RM/RL | Robot | 独立部署 | 与本项目重合度 | 未覆盖部分 | 证据位置 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Can You Rely on Synthetic Labellers… | AAAI 2024 | [Met24b] | 非AI；手写reward oracle | 有，crowd | empirical matching，不拟合ρ | **有stage-wise实测** | 无 | agreement筛选实验 | 无hybrid escalation | RM+SAC | DMC Walker | actor结构支持 | **现象直接先例** | frozen AI+adapter+online | pp.3–5 Fig.4；p.7 Table3 |
| DA-RAC | 2026-08 preprint；模板venue未独立核实 | [Wu26f] | 有，prompting | LLMEval2人类标签 | 邻域vanilla-judge agreement估计 | 静态 | 无 | few-shot grounding；非本项目分布纠正 | triage建议，非预算闭环 | 无 | 无 | 不适用 | 很高，上游 | robot drift+RM/RL | §3.1–3.4 Eq.2；§5.5 |
| UrbanAlign | 2026 preprint，所读v4 | [Zha26j] | 有 | Place Pulse pairwise votes | local score correction；非ρ | 静态 | 无 | LWRR局部修正 | 无预算升级实验 | 无策略RL | 城市场景非robot RL | 不适用 | 高，上游 | dynamic robot loop | §3.5 Eq.5–6；§4.1/4.4 |
| PrefMoE | 2026 preprint，所读v1 | [Yua26] | 非AI judge设定 | UniRLHF/PrefMMT人类数据 | trajectory gating；非correctness ρ | offline | 无 | 建模异质偏好；非AI标签修正 | 无 | RM+IQL | D4RL/MetaWorld | actor结构支持 | 中高 | AI-human disagreement及online | §III-B Eq.6/10；§IV-A/C；§V-B |
| Calibrating the Evaluator | 2026 preprint，所读v1 | [Liu26d] | 有 | `binary_outcome`来源未明；未证实真人 | confidence isotonic；非human ρ证据 | strategy weights更新 | 有window calibration，非已证实human校准 | confidence gating/weighting | 无 | TTRL策略权重；非独立RM/动作RL | 无 | evaluator-free未证实 | 高，相邻动态 | 标签来源、human/robot闭环 | §3.1–3.2 Eq.1–2；§4 |
| Cost-Effective Proxy RM | 2024 preprint，所读v2 | [Che24f] | GPT-4 oracle | 实验为GPT labels | active informativeness；非ρ | on-policy采样 | 非human recalibration | proxy labeling | 有AI query预算 | proxy RM→DPO | 无 | 文本policy支持 | 中高 | human-anchored correctness | §3–4；§4.3 Table1 |
| Reliability-Aware LLM Alignment | 2026 preprint，所读v1 | [Hua26d] | 非AI teacher设定 | 多人数据；无额外gold必要 | annotator confusion+item consistency；非AI-human ρ | 固定池训练；非drift实测 | 无 | latent label推断+tie筛选 | 无 | DPO，无独立RM | 无 | 文本policy支持 | 中 | frozen AI+online robot | §3 Eq.3/9/13/15；§4.1 |
| Conformal Feedback Alignment | EACL 2026元数据；所读v1 | [Che26e] | 有black-box变体 | 实验为GPT偏好 | answer conformal weight；非ρ | 无目标实测 | 无human recalibration | weighting，不是纠正 | 无 | RM/PPO及DPO变体 | 无 | 文本policy支持 | 中高 | real-human+robot+shift | §3.2；§4.2–4.3；§6 |
| Judgment Distribution | EMNLP 2025元数据；所读v2 | [Wan25j] | frozen；需概率/logits接口 | human benchmark评测 | 利用完整分布；非ρ | 静态 | 无 | inference聚合非human adapter | 无 | 无下游RL | 无 | 不适用 | 中，输出表示 | dynamic correction/RL | §3–4；Limitations p.9 |
| PreferenceEKF | 2026-09；PDF列TMLR，独立venue未核实 | [Zho26b] | frozen特征骨干不等于AI judge | 主体oracle；App.A.2.9有人类实验 | reward posterior/InfoGain；非ρ | fixed trajectory pool | Bayesian更新，非AI agreement calibration | 无AI纠正 | active query，非AI-human升级 | RM+offline IQL | D4RL/V-D4RL/SOAR数据 | offline actor | 中高，强baseline | frozen AI/online policy drift | §5/5.5；App.A.2.9/A.3 |
| PromptShift-CRC | 2026 preprint，所读v1 | [Opo26] | 有 | 允许human/ verifier；真人anchor未逐实验确认 | contextual risk阈值；非逐点ρ模型 | prompt/domain shift，非RL | 有online risk update | 阈值校准非preference correction | 可escalate；无固定human预算 | 无learned RM/策略RL证据 | 无 | 不适用 | 高，动态上游 | human-budget robot loop | §3–6；§9 |
| Localize-Then-Decide | 2026-08 preprint，所读v1 | [Li26i] | frozen judge | 有calibration数据 | shortlist+接受集合保证；非逐点ρ | exchangeable静态 | 无 | shortlist/selection，非纠正 | model cascade | 无 | 无 | 不适用 | 中高 | dynamic correction/RL | Assump.2.1；Cor.3.3；§4.4 |
| Deployable Human Preference Alignment | 2026 preprint，所读v1 | [Kim26e] | 冻结encoder，非AI judge | 含模拟user reward weights；真人anchor未证实 | user-cluster posterior，非ρ | offline pool | 无 | cluster reward modeling | 固定labels/user，非升级 | RM+IQL | MuJoCo仿真 | actor结构支持 | 中 | frozen evaluator错误建模 | §3.3 Eq.8；§3.4；§4.4 |
| Disagreement Prediction | 2026 preprint，所读v1 | [Eha26] | 有 | CEFR-SP真人用于评测 | 几何confidence proxy；非calibrated ρ | 静态 | 无 | ranking难例非纠正 | Precision@20非实际预算闭环 | 无 | 无 | 不适用 | 中，上游 | calibrate→robot loop | §3 Eq.1–2；§4 Tables2–5 |
| Multi-Expert CRC | 2026-08 preprint，所读v1 | [Che26k] | frozen；需logits | 有 | expected/selective risk；非逐点ρ | 静态 | 仅limitations建议 | consensus/abstention非修正 | human review建议，非预算实验 | 无 | 无 | 不适用 | 中高 | black-box输出限制及online/RL | §2 Eq.3；§4 Eq.5；Alg.1；§9 |

### 4.3 对原始全文答复的纠错

不能机械复制全文工具给出的 YES/NO。以下判定由公式、正文与本地 PDF 抽查纠正：

1. PrefVLM/ROVED 的 `KL(label || RM preference)` 是噪声 proxy，不是直接监督的 `P(AI=Human | x)`。PrefVLM §4.2 明确 **freeze VLM + train G_L/G_I**，不能用“对方更新teacher全部权重，我们freeze”制造差异。
2. PrefVLM 正文多次写 human feedback，却未明确标注实验真人来源；没有参与者/IRB 描述不能据此断言是 simulator oracle。ROVED 的 Gemini oracle 实验明确；标准 oracle 的具体生成方式在所读正文中不足以独立确认，不能仅因“following PEBBLE”就宣称真人或oracle已证实。
3. BACON §2.3/Table1 已包含 `P(Y_i=k)` 的 multinomial 模型；本项目从该类分布取 `p_H[y_AI]` 不是新的概率建模原理。
4. Trust or Escalate / Localize-Then-Decide / CRC 的接受集合风险控制，不等于对任意单个 x 都有 calibrated correctness 保证；exchangeability 条件不能直接移植到自适应 on-policy stream。
5. RLTHF 的 fixed-pool annotation iterations 不等于 policy-induced rollout stages；random acquisition 不等于 static calibration。多篇论文的 budget sweep 也不是 escalation 实验。
6. **TrustRoboReward p.9 明确任务为 image-to-video prediction。App.E.2/Table7 是 Cosmos-Predict2.5-2B + DiffusionNFT；App.E.1/Table6 的 GRPO 是训练 reward judge。** 不能把它写成已完成机器人动作策略 PbRL。本文不使用 p.9 图文不一致的 Spearman 数值作为定量证据。
7. LAPP、Hybrid Preferences、RLTHF、Judging with Confidence 所读 PDF 与元数据可能分属不同版本；正式venue与所读方法版本分开标注，不把作者检索里的“未查到venue”当作否定全文中的出版页眉。

## 5. 哪些 novelty claim 已失效，哪些只能谨慎保留

| Claim | 判定 | 主要依据 |
|---|---|---|
| AI preference → RM/RL、frozen teacher、独立部署 | 已有，不应声称首次 | RL-VLM-F、LAPP |
| Human+AI、budget routing、confidence weighting、sample-wise filtering/flipping | 已有 | PrefVLM、ROVED、Hybrid Preferences、RLTHF、CFA |
| small human + frozen black-box external correction | 已有 | Bur25；UrbanAlign |
| context-wise human outcome / correctness estimation | 已有很近先例，不能声称首次 | BACON；DA-RAC |
| source-wise trust、trajectory-dependent preference noise | 已有 | TriTrust；FDN |
| policy-aware reward learning、跨阶段静态反馈失效 | 已有 | VARP；LAPP在线对照；PrefVLM Fig.8 |
| human–synthetic agreement 随policy训练变化 | **已有直接现象先例** | Met24b Fig.4；synthetic是reward oracle |
| online/drift-aware calibration | 不能泛称首次 | PromptShift-CRC；Calibrating the Evaluator（标签来源未明） |
| full judgment distribution 优于argmax/scalar | 已有动机与结果，不是新模块 | Judgment Distribution；Judging with Confidence |
| 完整目标组合已无人做 | **不能确认**；本轮未找到完整覆盖证据 | 查全、版本和实验仍受限 |

**最小可辩护增量（待实验）：** 在真实人类轨迹偏好上，量化同一冻结 AI 在不同策略产生的数据中的误差和校准迁移失效；用非oracle上下文与完整反馈分布进行外部纠正，并在相同人工判定总成本、严格未来/新seed测试下，证明在线更新比 static contextual、recent-only retraining、KL filtering 和简单直接human predictor 更能改善机器人 RM/SAC。价值来自可重复的失效机制与不可被强baseline解释的 policy收益，而非七个已有部件的列举。

建议论文措辞：

> We investigate how the human alignment of a fixed AI trajectory evaluator transfers across policy-generated distributions, and test whether budget-matched online contextual correction improves reward learning and robot policies beyond static calibration and disagreement filtering.

在证据齐备前使用 **policy-distribution-dependent agreement** 或 **agreement under policy-induced covariate shift**。不要先把 residual conditional drift 当成必然存在的自然规律。

## 6. Policy-Induced AI–Human Agreement Shift：直接先例与因果边界

### 6.1 直接先例的准确范围

Met24b，AAAI 2024，PDF p.5 Fig.4 caption：

> “The degree of label matching between the human and synthetic labellers per feedback round for each task.”

同页正文：

> “the degree of label matching depends on the task and the feedback round.”

p.3 定义 synthetic labels 来自 “the task’s hand engineered reward function”；p.4 说明第一轮后不同策略会暴露于不同数据。因此，**泛化的“human 与非human labeler 的 agreement 随训练阶段变化”已被研究**。论文中走势按任务不同，并不是所有任务一致单调下降。它没有训练 human-anchored AI correctness adapter，也没有控制所有阶段的 pair difficulty 后证明独立的条件机制漂移。

本轮没有确认一篇同时满足 **frozen LLM/VLM、真实human、难度/组成控制、等预算online correction、机器人动作PbRL** 的直接完整先例。这一限定比“首次policy shift”窄得多。

### 6.2 总体分布变化不等于条件函数变化

令 `a=1[y_AI=y_H]`，`X` 为完整评价上下文，`T` 为策略阶段：

$$
A_t=\mathbb E_{X\sim d^{\pi_t}}[\rho_t(X)],\qquad
\rho_t(x)=P(a=1\mid X=x,T=t).
$$

即使 `ρ_t(x)=ρ(x)` 始终不变，`d^{π_t}` 改变也足以让 `A_t` 改变。若冻结teacher、人类目标、输入与标注协议都固定，给定完整同一 `x`，stage 通常不该直接改变评价分布。可见的 residual stage effect 可能来自压缩特征遗漏、抽样机制、标注人群/准则变化，而非“teacher内部随policy退化”。

因此同时报告：

1. **总体效应**：自然on-policy流的 agreement/calibration 变化；保留真实难度组成变化，它可能正是policy造成的中介过程。
2. **标准化效应**：在task、pair difficulty、near-success、长度、OOD等共同支持区域匹配/加权后的变化。难度也可能是中介；控制它回答的是“超出难度组成的增量”，不是唯一的总因果效应。
3. **迁移失败**：早期训练的 calibrator 在未来阶段/新seed上是否失败，online 是否修复。即使无 residual stage effect，这仍可能是有限样本支持不足导致的有用问题，但不再称独立conditional drift。
4. **固定sentinel pairs**：相同输入跨时间重新评价且冻结版本/准则，用于排查API更新、时间效应和人类标准漂移；它们不应因“当前stage”标签而改变含义。

## 7. Phase 1 必须补充的设计、baseline 与指标

### 7.1 九项关键判断

| 问题 | 判断与具体修订 |
|---|---|
| policy-induced vs difficulty shift | 必须同时给总体与difficulty-standardized结果；固定task配比、early/late common-support匹配；不要用teacher自身confidence作为唯一difficulty定义 |
| stage ID leakage | teacher/human不见checkpoint标签；按trajectory连通分量、policy seed和时间切分；同轨迹的swap/paraphrase衍生样本必须同fold；stage-only、去stage、置乱stage作负控 |
| 200–400 pairs够不够 | 适合工程与大效应pilot，不足以保证多任务/多stage/interaction检测；见下节敏感性计算；不显著≠No-Go |
| 多人标注与soft preference | 目标先定义为指定人群随机annotator的偏好分布，另报majority；独立测试/ambiguous子集至少多位盲标，建议3人起并报告实际人次；tie与unable-to-judge分开 |
| full vector vs scalar | 必须比较label-only、top1/margin/entropy、完整q、q+context、context-only；多输入不保证胜出；其价值须跨seed留出成立 |
| structured state oracle leakage | reward、success flag、自动优劣排序禁入主teacher/adapter；精确goal distance等可作为privileged上界单列；Human/AI可见信息保持一致；同输入heuristic/direct predictor必须对照 |
| correction/reliability是否统一 | 优先统一3类 `p_H(A,B,Tie)`；派生ρ；纠正后不能用“原AI正确率”压低已成功纠正的样本，详见§8 |
| MetaWorld是否太简单 | 保留两任务做pilot，但加入near-success及任务内safety/efficiency trade-off；先做deterministic heuristic与human-only contextual predictor；若收益全由goal距离解释，无需扩RL规模 |
| open/local evaluator是否必要 | 正式论文需要至少一个冻结可复现backend；第1周定义接口、版本和缓存，第4周小子集复测，完整扩展放Go后；Laya/Metask/Nimble名称及能力尚无本轮学术证据背书 |

### 7.2 最小baseline阶梯

Phase 1 不实现所有大型论文，只做能排除替代解释的轻量版本，并清楚标为 **inspired baseline**，不冒充原论文复现：

1. majority/chance、raw AI、同输入deterministic heuristic、直接 `context→human preference`（无AI输入）。
2. label-only confusion mapping（Bur25类）；scalar logistic/isotonic；完整q的multinomial calibration。
3. static `q+context`（BACON式），邻域human-agreement（DA-RAC式），context-only geometric/consistency proxy。
4. stage-only、`q+context+stage`、no-stage、置乱stage；早→晚、新checkpoint、新policy seed留出。
5. **同capacity/同标签数**的 frozen static、online累计重训、online recent-window、periodic random-query、targeted-query。区分“更新数据收益”与“复杂drift算法收益”。
6. RM可用后加入 PrefVLM/ROVED/RIME式KL filtering/flipping与query；TriTrust source-wise、human-only RM、naive hybrid、raw confidence weighted RM，沿用docs/13的比较骨架。

### 7.3 预算、公平性与评价协议

将人工预算定义为 **实际独立判定次数**（同时报告分钟/成本），不是 unique pairs。初始化anchor、重复标注、校准更新、drift monitoring、escalation、人工prompt调优均计入训练预算。固定独立测试集标注成本另列，所有方法共用且不能用于拟合/选阈值；另报训练+评测总成本。

预算拆分建议作为pilot假设而非已优化参数：保留一部分随机audit/sentinel查询，余量用于targeted acquisition；记录每个样本被选择的概率。只用lowest-ρ样本更新calibrator会造成选择偏差；必须有随机audit覆盖，或在已知且非零propensity及足够effective sample size时加权。仅对已查询样本画ECE会高估系统可靠性。

静态/在线公平比较至少两种：

- **cost-matched可实施比较**：相同总B，static一次获得早期B；online按阶段花B。承认其label时点不同是处理因素。
- **mechanism comparison**：同一按时间到达的人类标签流，比较不同online模型/更新窗口；另列“仅B0初始标签的static”作为诊断，不能伪称与花满B的online等预算。

在线采用 **先预测、后查询/揭示标签、再更新** 的prequential协议；所有方法共享外生采样流时可比较feedback/RM机制，进入真实RL后再用匹配seed/预算的独立训练比较最终策略。新policy seed的所有轨迹整体留出；只在pair级随机切分不足以防止轨迹重用泄漏。

### 7.4 必报指标

- feedback：hard agreement、multiclass Brier/NLL、binary agreement Brier/NLL、class-wise calibration、ECE（预先固定bins，仅辅助）、risk–coverage/AURC。
- correction：**原错改对**与**原对改错**分别计数，net correction gain，tie/abstain覆盖率；ρ与corrected-label correctness分开评价。
- drift：task/stage分层、标准化差异及置信区间、cross-stage train/test矩阵、sentinel稳定性、OOD/common-support比例。
- human：多标注者一致性、vote distribution、重测稳定性、标注人次与时长；不能把“人类全一致”默认成ground truth。
- RM：held-out human preference NLL/Brier、ranking、跨阶段性能；训练/测试human anchor分离。
- policy：success、true environment return、预先定义的human utility/safety、learning AUC、query成本，多seed区间。环境reward仅作外部评价/特定对照，不替代human objective。

### 7.5 200–400 pairs 的功效敏感性

以下是**规划用近似下界，非真实实验效应估计，也不是最终混合模型功效保证**。假设两独立组、双侧α=0.05、power=0.80，参考agreement约0.75，N均分四stage、不扣训练/验证集、不计task/seed/annotator聚类。early-vs-late近似最小可检测差异：

$$
\mathrm{MDE}\approx(z_{0.975}+z_{0.80})
\sqrt{2\times0.75\times0.25/(N/4)}.
$$

| 总独立pairs N | 每stage | 约可检测差异 |
|---:|---:|---:|
| 200 | 50 | 24.3个百分点 |
| 400 | 100 | 17.2个百分点 |
| 800 | 200 | 12.1个百分点 |

两比例正态近似另算：0.80 vs 0.70 约需 **294/组**；0.80 vs 0.60 约需 **82/组**。这些是假设场景，不代表预期effect。若只聚焦early/late，可提高同预算功效；不能再同时声称充分刻画四stage交互。

重复使用同轨迹、同policy seed和多位标注者并不增加同等独立样本。设计效应 `1+(m−1)ICC` 仅是粗略提醒；正式样本量应根据pilot的方差/相关结构、预先选定的最小有用效应与主分析模型做cluster simulation，并预留有效独立测试pairs。第4周若区间同时包含“无效”和“有意义收益”，判为**证据不足**，而非统计No-Go；追加一次有上限的定向验证，不无限延长。

## 8. Change proposals：不直接覆盖执行基线

### P1：收缩现象首创与stage叙事

**当前方案**：发现policy-induced agreement shift → **具体问题**：现象有先例，marginal shift可由difficulty mixture解释 → **证据**：Met24b Fig.4，PrefVLM Fig.8，VARP Fig.3 → **建议修改**：同时测总体、标准化与calibrator迁移失败，stage只作为候选特征/诊断 → **修改后novelty**：冻结AI的真实human对齐在策略数据迁移下何时失败、何种纠正有效 → **新增工程量**：匹配/重加权、跨seed拆分、sentinel及负控约2–4人日（规划估算） → **4周影响**：纳入现有数据管线，必要时减少任务而不放松控制 → **12–16周影响**：不增加方法模块；删除无效stage分支 → **投稿影响**：减少宽泛首创风险，需更强机制证据。

### P2：统一概率目标，纠正后重新定义可靠性

**当前方案**：`(corrected label, ρ)`，用原AI agreement `ρ`加权corrected loss → **具体问题**：两个head可能矛盾，而且“原AI错误”不代表“纠正后不可靠” → **证据**：docs/13 §2.1损失定义与docs/15统一分布建议；BACON §2.3/Table1、Judging with Confidence概率目标 → **建议修改**：

$$
p_H=F_\phi(q,x),\quad
y_{AI}=\arg\max_k q_k,\quad
\rho_{raw}=p_H(y_{AI}),\quad
\hat y=\arg\max_k p_H(k),\quad
\rho_{corrected}=p_H(\hat y).
$$

`ρ_raw`用于回答“是否接受原AI”；`p_H`用于soft-target RM训练；若要给纠正后标签加权，须另验证 `ρ_corrected` 或epistemic/OOD不确定性，而不是机械沿用 `ρ_raw`。例如AI选A，`p_H=(0.05,0.90,0.05)`，修正B后乘0.05会恰好压掉最有价值的系统纠错。

对 A/B/Tie → 二元BT，预先约定 `target_A=p_H(A)+0.5 p_H(Tie)`；或采用显式tie模型，两种设计分开比较。`unable-to-judge`是缺失/弃权，不是Tie。soft-target CE本身已包含aleatoric ambiguity，额外entropy权重可能重复降权，应保留不加权soft-target baseline。低 `ρ_raw` 也不必一律问人：可纠正的系统错误应与高epistemic uncertainty分开。

→ **修改后novelty**：一致的、可校准外部纠正决策及其闭环效果，非双head结构 → **新增工程量**：1–2人日模型/损失/指标实现 → **4周影响**：减少并行head调参 → **12–16周影响**：增加raw weighting、soft correction、corrected confidence的必要消融 → **投稿影响**：修复方法逻辑，避免把恒等式当创新。

### P3：预算和样本量变成明确实验约束

**当前方案**：200–400 pairs pilot；固定Human percentage；static vs online → **具体问题**：独立pairs、人次、校准/监测成本混淆；小样本多重拆分可能失去功效 → **证据**：§7.5计算；Met24b每pair多票；BACON cross-fitting与sampling设计 → **建议修改**：budget ledger、共享盲测、预注册primary early/late contrast、随机audit支流、pilot后有上限增样 → **修改后novelty**：等真实人工成本的动态纠正收益 → **新增工程量**：约2–3人日工程，标注时间按实测另计 → **4周影响**：明确Go/证据不足/No-Go三分，不能承诺400对必得结论 → **12–16周影响**：Go后按正式power计划扩标；需增样时压缩次要ablation而非削弱测试 → **投稿影响**：使human-efficiency claim可审。

### P4：把替代解释与复现对照提前

**当前方案**：MetaWorld structured-state、Jev主backend，后续local模型 → **具体问题**：heuristic/oracle-like输入或直接human predictor可能已足够；API版本变化破坏frozen设定 → **证据**：LAPP structured logs；RL-VLM-F Fig.6难度；FDN失败模式；UrbanAlign/DA-RAC强轻量上游 → **建议修改**：同信息heuristic、无AI predictor、oracle-feature removal、cache/version固定；第4周local小样本复测 → **修改后novelty**：AI信号在受控信息集上的额外价值与跨backend稳定性 → **新增工程量**：约2–4人日；local推理成本依模型资源 → **4周影响**：不启动大模型训练，可与数据收集并行 → **12–16周影响**：至少一个local完整baseline；不自动新增视觉/真机 → **投稿影响**：支持可复现RA-L核心版，后续才谈跨环境升级。

这些工期是实现估算，部分可重叠，尚无实验运行数据支持精确排期。按docs/14保留第4周门槛和12–16周完整稿目标；若必须增标或修复环境，只能报告偏差，不能把路线表当交付保证。

## 9. 最终 Go / Weak-Go / No-Go 与保持、收缩、pivot

**本轮文献判断：Weak-Go / Proceed with caution。保持研究主线，收缩claim，不因相似关键词pivot。** 薄弱点可具体定位：Met24b已有现象，BACON/DA-RAC已有上游模型，PrefVLM/ROVED已有机器人预算闭环。剩余增量必须由严格的动态迁移与policy收益证明；“完整组合没有同名论文”不够。

| Gate | 可执行决定 |
|---|---|
| 第4周有跨seed、去泄漏的predictive/correction增益，且区间支持实际意义 | Go进入RM/SAC；漂移不要求一定下降，也不要求stage ID一定有用 |
| 无额外stage效应，但context bias可泛化且简单static仍不足 | Weak-Go，按docs/13转为Human-Aligned Feedback Correction；重新说明相对BACON/UrbanAlign的下游增量 |
| 无stage效应且static contextual已覆盖收益 | 收缩动态叙事；不要为保标题硬加online模块；先检验static correction是否仍有独立policy价值 |
| pilot区间太宽 | 证据不足；有限增样/集中primary contrast，不把p>0.05当证明不存在 |
| 在预设实际意义阈值、足够功效与区间支持下，同输入heuristic/direct-human predictor覆盖收益，或校准不能跨trajectory/seed泛化 | 当前方法分支No-Go；小pilot不显著仍归为证据不足；可更换evaluator或任务一次，不回退成Jev→reward旧题 |
| feedback/RM改善不传递policy，或收益仅来自额外human/API成本 | 不进入真机/第二环境高成本扩展；诊断RM/policy瓶颈并设有限止损 |

无已核实的完整覆盖论文，因此不能基于文献直接判整个方向No-Go；也没有本项目实验结果，因此不能给强Go或承诺可发表。

## 10. 对投稿定位的影响

| 目标 | 本轮后的定位 | 必须补足的证据 |
|---|---|---|
| RA-L / 相近机器人学习期刊 | 仍可作为12–16周核心稿目标，不是质量/录用保证 | 机器人动作策略完整闭环、真实human目标、强静态与KL/query对照、等预算、多seed、复现backend；两任务校准图仍只是PoC |
| CoRL / RSS | 不因“新现象名字”自动升级；需要清楚的机制或方法增量 | 区分组成迁移与校准失败、跨任务/seed/evaluator泛化、第二环境或有说服力的实机证据；收益不能只在一个困难分组出现 |
| T-RO / IJRR | 继续作为长线扩展，不提前堆工程 | 长时序/真实机器人、系统人类研究、广泛泛化与可靠性分析；不能仅把任务/seed数量加倍当长稿贡献 |

本轮没有重新核实2026投稿deadline、JCR分区或实验室奖励/资助制度；docs/14的这些行政信息不能由本次文献核验自动升级为已验证事实。

## 11. 尚未解决的问题

1. **最新版本完整性**：24个arXiv页面直连超时，作者追踪部分失败；所读PDF可能落后于最新修订。ROVED项目URL含ICRA字样不等于正式出版证据。
2. **真人数据来源**：PrefVLM未明确；ROVED标准oracle来源仍需代码/作者材料交叉确认；Demo2Reward需逐实验区分simulation标签与真人success标注；不能将“human”概念框架当真实实验。
3. **最新近邻保证适用范围**：DA-RAC的neighborhood signal能否外推到robot任务、PromptShift-CRC监督来源/假设能否在固定人类预算下满足，均未有本项目证据。
4. **Calibrating the Evaluator** 的 `binary_outcome`来源未明；不能认为它已证明独立human agreement校准。
5. **完整全文未取的外围候选**：`Adaptive Confidence-aware Preference-based RL with Noisy Feedback`、`PrefCLM`、`Active reward learning and iterative trajectory improvement from comparative language feedback`在检索中标记PDF不可用。本轮不对其未读方法作否定性结论，也不把它们计入31篇全文核验。
6. **产品证据**：system-one/Jev专项没有返回可确认的同类学术闭环先例；这不证明不存在。Jev、Laya、Metask-Jev、Nimble的官方接口、校准、版本冻结和复现条件仍需官方材料/实测。
7. **实验未知量**：真实effect size、轨迹/seed相关、多人agreement、teacher基础价值、online增益和downstream causal chain都还未测。
8. **统计目标**：多数票、随机人类偏好与任务成功并非同一个target；必须在数据收集前固定目标人群/准则，不能事后选择更有利的定义。

## 12. 操作记录与复核范围

本轮只新增本文和README目录链接；不改docs/13、docs/14。证据工具的原始输出、下载PDF和复核记录保存在本地 `.aris/traces/novelty-check/2026-09-23_run01/`，不将私人workspace原始数据或临时下载链接推送到GitHub。公共文档用持久DOI/arXiv链接引用论文。

方法复核使用 novelty-check 的同模型家族第二agent审读，为 **provisional**，不是独立跨模型认证。复核结论同为Weak-Go；其关于组成迁移、预算选择偏差、统一概率目标和pilot功效的意见已纳入本文，并补充了No-Go必须有充分功效/区间支持的条件。样本量部分使用 statistical-power 流程及 SciPy 正态近似敏感性计算；未把近似计算冒充真实聚类设计的simulation power。技能来源资料：Kassis et al., *Scientific Agent Skills: A Library of Procedural Knowledge for Research Agents*（2026，[arXiv:2609.00065](https://doi.org/10.48550/arXiv.2609.00065)；在线记录直连超时，Undermind补充定位亦返回No papers resolved，书目信息据技能文件，非novelty证据）。

## 附录 A. 实际提交的定向查询

Q1–Q10 均为 `search_papers(search_type="semantic", year_min=2024, year_max=2026, limit=10)`。结果按semantic相关性返回；不是穷尽全集。Deep Search另自主生成查询与citation追踪，其内部所有query字符串未通过MCP公开。

| ID | 主题 | 实际 sample_abstract |
|---|---|---|
| Q1 | Policy-induced AI–human agreement shift | As reinforcement-learning policies improve, the agreement between a fixed AI preference annotator and human trajectory judgments changes. We measure this policy-induced agreement shift and separate changes in pair difficulty from conditional annotator reliability. |
| Q2 | Human-calibrated judge under on-policy shift | A human-calibrated AI judge becomes miscalibrated on trajectories generated by an evolving on-policy reinforcement-learning agent. Contextual calibration predicts agreement with human preferences under distribution shift. |
| Q3 | Frozen black-box external correction | We keep a black-box evaluator frozen and learn an external mapping from its categorical judgments or probability outputs to human preferences using a small calibration set. The adapter corrects sample-specific biases without updating the evaluator. |
| Q4 | Context-dependent annotator reliability | Preference-based reinforcement learning models annotator reliability as a function of individual trajectory features and comparison context rather than a single source-level trust scalar. Human preference anchors estimate AI label correctness. |
| Q5 | Online recalibration during policy learning | We periodically recalibrate AI-generated preference feedback during policy learning using newly collected human labels. Online recalibration is evaluated against static calibration across policy stages. |
| Q6 | Human-budget correction/escalation | Under a fixed total human annotation budget, AI-generated trajectory preferences are corrected and uncertain comparisons escalated to humans. We jointly allocate calibration and active labeling queries for downstream reinforcement learning. |
| Q7 | Feature-dependent noise | Language-model trajectory preference noise is feature-dependent and changes with robot policy behavior. Noise-robust preference-based reinforcement learning corrects systematic errors rather than only filtering random label noise. |
| Q8 | System-one evaluators | Calibrated system-one decision models output structured categorical probability vectors for preference or reward evaluation. We test fast frozen decision evaluators for robot reinforcement learning and their alignment with human judgments. |
| Q9 | Cross-stage generalization | Reward or preference evaluators trained on early policy trajectories fail to generalize to later policy checkpoints. We evaluate cross-stage generalization and recalibration while controlling task composition and trajectory difficulty. |
| Q10 | Structured trajectory preference correction | Human feedback corrects AI preferences over structured robot trajectory summaries. A frozen language-model evaluator provides A, B, and tie probabilities, and a contextual adapter learns the human preference distribution. |

引用网络四次成功查询共享文本：

> Human calibration estimates context-dependent reliability of AI preference labels and corrects noisy feedback during evolving policy optimization, using budgeted active human queries and external black-box adapters.

组A：Gho25、Gho26、Jun24、Bur25、Hos26、Li26f、Sin25、Gum26；组B：Jia25、Wan24、Li25j、Li26d、Mir24、Xu25b、Shi26b、Wan26l。分别查citations/references，2024–2026，每次25条；citations按年降序，但这是semantic候选内排序，不是整个citation corpus时间穷举。

新Deep Search提交目标：

> As of 23 September 2026, we need to check whether the human agreement of a frozen black-box AI trajectory preference evaluator changes as a robot reinforcement-learning policy evolves, and whether small human preference budgets have already been used to learn sample-conditioned human preference corrections and AI–human agreement probabilities, periodically recalibrate them on new on-policy data, allocate human escalation, and improve reward models and downstream standalone robot policies. The evaluator's weights remain fixed; the trainable object is an external adapter using its probability vector and trajectory context. The scientific question is whether this agreement change is more than a change in pair difficulty or task composition, and whether online recalibration beats static contextual calibration at equal human annotation cost. Find the closest precedents including partial ones, particularly 2024–2026 preprints and updated versions, without requiring every component to be present in a single abstract. We need evidence to determine the smallest defensible increment over AI judge calibration, black-box human correction, feature-dependent preference noise, and policy-aware reward learning.

## 附录 B. 完整文献题名与持久链接

以下 cite keys 属于本次 Undermind workspace；在公共文档中使用显式引用链接，避免离开workspace后不可解析。

1. [Met24b] — Metcalf et al. **Can You Rely on Synthetic Labellers in Preference-Based Reinforcement Learning? It’s Complicated**. AAAI 2024.
2. [Gho25] — Ghosh et al. **Preference VLM: Leveraging VLMs for Scalable Preference-Based Reinforcement Learning**. 2025.
3. [Gho26] — Ghosh et al. **Reducing Oracle Feedback with Vision-Language Embeddings for Preference-Based RL**. 2026.
4. [Jun24] — Jung et al. **Trust or Escalate: LLM Judges with Provable Guarantees for Human Agreement**. 2024.
5. [Shi26b] — Shi et al. **BACON: Budgeted Human Calibration for Modeling and Evaluation with Multiple AI Judges**. 2026.
6. [Bur25] — van den Burg et al. **Aligning Black-box Language Models with Human Judgments**. 2025.
7. [Hos26] — Hosseini et al. **Trust, Don’t Trust, or Flip: Robust Preference-Based Reinforcement Learning with Multi-Expert Feedback**. 2026.
8. [Li26f] — Li et al. **Evaluating Feature Dependent Noise in Preference-based Reinforcement Learning**. 2026.
9. [Sin25] — Singh et al. **VARP: Reinforcement Learning from Vision-Language Model Feedback with Agent Regularized Preferences**. 2025.
10. [Gum26] — Gumbsch et al. **From Demonstrations to Rewards: Test-Time Prompt Optimization for VLM Reward Models**. 2026.
11. [Jia25] — Jian et al. **LAPP: Large Language Model Feedback for Preference-Driven Reinforcement Learning**. 2025.
12. [Wan24] — Wang et al. **RL-VLM-F: Reinforcement Learning from Vision Language Foundation Model Feedback**. ICML 2024.
13. [Li25j] — Li et al. **Judging with Confidence: Calibrating Autoraters to Preference Distributions**. 2025.
14. [Li26d] — Li. **Calibrate, Don’t Curate: Label-Efficient Estimation from Noisy LLM Judges**. 2026.
15. [Mir24] — Miranda et al. **Hybrid Preferences: Learning to Route Instances for Human vs. AI Feedback**. 2024/2025 version.
16. [Xu25b] — Xu et al. **RLTHF: Targeted Human Feedback for LLM Alignment**. 2025.
17. [Wan26l] — Wang et al. **TrustRoboReward: Preference-Ordered Isotonic Score Editing for Multi-Paradigm Robot Reward Models**. 2026.
18. [Wu26f] — Wu et al. **DA-RAC: Distance-Aware Calibration of LLM Judges for Trustworthy AI Auditing**. 2026.
19. [Zha26j] — Zhang et al. **UrbanAlign: Post-hoc Semantic Calibration for VLM-Human Preference Alignment**. 2026.
20. [Yua26] — Yuan et al. **PrefMoE: Robust Preference Modeling with Mixture-of-Experts Reward Learning**. 2026.
21. [Liu26d] — Liu. **Calibrating the Evaluator: Does Probability Calibration Mitigate Preference Coupling in LLM Agent Feedback Loops?** 2026.
22. [Che24f] — Chen et al. **Cost-Effective Proxy Reward Model Construction with On-Policy and Active Learning**. 2024.
23. [Hua26d] — Huang et al. **Reliability-Aware LLM Alignment from Inconsistent Human Feedback**. 2026.
24. [Che26e] — Chen et al. **Conformal Feedback Alignment: Quantifying Answer-Level Reliability for Robust LLM Alignment**. 2026.
25. [Wan25j] — Wang et al. **Improving LLM-as-a-Judge Inference with the Judgment Distribution**. 2025.
26. [Zho26b] — Zhou and Bıyık. **Subspace Inference Enables Efficient Active Reward Learning from Preferences**. 2026.
27. [Opo26] — Opoku and Banahene. **PromptShift-CRC: Drift-Aware Conformal Risk Control for Foundation Models Under Prompt and Domain Shift**. 2026.
28. [Li26i] — Li et al. **Localize-Then-Decide Guarantees for LLM Judgments**. 2026.
29. [Kim26e] — Kim et al. **Deployable Human Preference Alignment in Robotics: Learning Representative Rewards from Diverse Human Preferences**. 2026.
30. [Eha26] — Ehara. **Predicting Disagreement with Human Raters in LLM-as-a-Judge Difficulty Assessment without Using Generation-Time Probability Signals**. 2026.
31. [Che26k] — Cheng et al. **Multi-Expert Conformal Risk Control for Pairwise LLM Judging in Open-Ended Dialogue**. 2026.

[Met24b]: https://doi.org/10.1609/aaai.v38i9.28877
[Gho25]: https://doi.org/10.48550/arXiv.2502.01616
[Gho26]: https://doi.org/10.48550/arXiv.2603.28053
[Jun24]: https://doi.org/10.48550/arXiv.2407.18370
[Shi26b]: https://doi.org/10.48550/arXiv.2607.16239
[Bur25]: https://doi.org/10.48550/arXiv.2502.04997
[Hos26]: https://doi.org/10.48550/arXiv.2601.18751
[Li26f]: https://doi.org/10.48550/arXiv.2601.01904
[Sin25]: https://doi.org/10.48550/arXiv.2503.13817
[Gum26]: https://doi.org/10.48550/arXiv.2606.00083
[Jia25]: https://doi.org/10.48550/arXiv.2504.15472
[Wan24]: https://doi.org/10.48550/arXiv.2402.03681
[Li25j]: https://doi.org/10.48550/arXiv.2510.00263
[Li26d]: https://doi.org/10.48550/arXiv.2605.09702
[Mir24]: https://doi.org/10.48550/arXiv.2410.19133
[Xu25b]: https://doi.org/10.48550/arXiv.2502.13417
[Wan26l]: https://arxiv.org/abs/2608.08491
[Wu26f]: https://arxiv.org/abs/2608.14950
[Zha26j]: https://doi.org/10.48550/arXiv.2602.19442
[Yua26]: https://doi.org/10.48550/arXiv.2605.00384
[Liu26d]: https://doi.org/10.48550/arXiv.2606.31371
[Che24f]: https://doi.org/10.48550/arXiv.2407.02119
[Hua26d]: https://doi.org/10.48550/arXiv.2607.20515
[Che26e]: https://doi.org/10.48550/arXiv.2601.17329
[Wan25j]: https://doi.org/10.48550/arXiv.2503.03064
[Zho26b]: https://arxiv.org/abs/2609.04066
[Opo26]: https://doi.org/10.48550/arXiv.2606.15964
[Li26i]: https://arxiv.org/abs/2608.25824
[Kim26e]: https://doi.org/10.48550/arXiv.2607.12466
[Eha26]: https://doi.org/10.48550/arXiv.2605.12422
[Che26k]: https://arxiv.org/abs/2608.26529
