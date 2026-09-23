# 14. Publication & Timeline Roadmap

更新时间：2026-09-23  
状态：**当前投稿与时间规划执行版**

---

## 0. 总体目标

当前课题不采用“一开始就按顶刊/顶会工作量铺满”的方式，而采用分阶段升级：

\[
\boxed{
4周\ Go/No\text{-}Go
\rightarrow
12\text{-}16周\ 完整稿
\rightarrow
结果足够强再追加2\text{-}3个月升级
}
\]

当前研究主线：

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

最关键假设：

\[
\boxed{
P(AI=Human\mid d^{\pi_0})
\neq
P(AI=Human\mid d^{\pi_T})
}
\]

即优先验证 **Policy-Induced AI–Human Agreement Shift** 是否真实存在。

---

# 1. 时间总表

| 阶段 | 建议时间 | 核心任务 | 决策 |
|---|---:|---|---|
| Phase A：工程与数据链 | 第1周 | MetaWorld、SAC、trajectory pair、AI API、Human标注链路 | 继续 |
| Phase B：核心现象 PoC | 第2–4周 | agreement / calibration / policy-stage shift | **第一次 Go/No-Go** |
| Phase C：Feedback Adapter | 第5–7周 | static / contextual / policy-aware / correction | **第二次 Go/No-Go** |
| Phase D：RM + RL 闭环 | 第8–10周 | Better Feedback → Better RM → Better Policy | **第三次 Go/No-Go** |
| Phase E：Human Budget + Online Recalibration | 第11–13周 | 固定人工预算、static vs online | 形成主贡献 |
| Phase F：Robustness + Ablation | 第14–16周 | OOD、systematic bias、noise、paraphrase、reward hacking | 完整稿 |
| Phase G：升级版 | 额外2–3个月 | 第二环境、真机、多 evaluator、更强分析 | 决定是否冲更高档 |

---

# 2. 第 1–4 周：先回答“这个方向值不值得继续”

## 2.1 第 1 周：跑通工程底座

完成：

- MetaWorld Drawer Open；
- MetaWorld Button Press；
- SAC baseline；
- random / early / mid / late policy checkpoint；
- structured trajectory serializer；
- trajectory pair generator；
- AI evaluator query + cache；
- Human pairwise annotation 表。

这一周不追求论文结果，只要求数据链完整可复现。

---

## 2.2 第 2–3 周：第一批 Human–AI Preference 数据

目标：

\[
N=200\sim400
\]

trajectory pairs。

覆盖：

- random；
- early；
- mid；
- late；
- easy pairs；
- ambiguous pairs；
- success vs near-success；
- failure vs failure。

每个 pair 保存：

\[
(\tau_A,\tau_B,y_H,y_{AI},c_{AI},task,stage,difficulty,OOD)
\]

必须得到：

- Human–AI agreement；
- agreement by policy stage；
- raw confidence vs actual agreement；
- ECE；
- Brier；
- NLL；
- reliability diagram；
- systematic error cases。

---

## 2.3 第 4 周：第一次硬性 Go/No-Go

### 强 Go

满足以下至少 2–3 项：

- AI preference 明显优于随机；
- raw confidence 存在明显 miscalibration；
- agreement 随 task/context 变化；
- early / mid / late policy 上 agreement 有漂移；
- static calibration 在 later-stage data 上退化。

### 弱 Go

如果 policy-stage shift 不明显，但发现：

- systematic context-dependent bias；
- correction 可以明显改善 Human preference prediction；

则保留项目，但弱化 “policy-shift” 叙事，转向：

> **Human-Aligned Feedback Correction for Frozen AI Teachers**

### No-Go

如果：

- AI 与 Human 接近随机；
- confidence 没有可利用信息；
- Human calibration / correction 无法泛化；

则停止继续投入完整 RL 实验。

---

# 3. 第 5–7 周：完成 Feedback Adapter

比较：

1. Raw AI；
2. Global calibration；
3. Context-aware calibration；
4. Policy-stage-aware calibration；
5. Correction + Reliability。

建议方法：

\[
F_\phi(
trajectory,
AI\ feedback,
confidence,
difficulty,
OOD,
policy\ stage
)
\rightarrow
(\hat y_H,\rho)
\]

其中：

\[
\hat y_H
=
\text{Human-aligned corrected preference}
\]

\[
\rho
=
P(y_{AI}=y_H\mid context,\pi_t)
\]

目标：

> 证明不仅“知道 AI 靠不靠谱”，而且可以学习 AI 的 systematic bias，并在不同 policy stage 下保持较好的 Human alignment。

---

# 4. 第 8–10 周：完成 Reward Model + SAC 因果链

核心必须证明：

\[
\boxed{
Better\ Feedback
\rightarrow
Better\ Reward\ Model
\rightarrow
Better\ Policy
}
\]

比较至少包括：

1. Environment sparse reward；
2. Human-only PbRL；
3. AI-only PbRL；
4. Human + AI naïve pooling；
5. Raw AI confidence weighting；
6. PrefVLM/RIME-style disagreement filtering；
7. Source-wise trust；
8. Static Human calibration；
9. Policy-Conditional Reliability；
10. Ours: Correction + Reliability。

正式实验：

- 至少 3 seeds；
- 主结果建议 5 seeds；
- 报告 Success Rate；
- True Return；
- Sample Efficiency；
- AUC；
- mean ± CI。

如果 calibration metric 变好，但最终 policy 没有稳定提升：

> 不进入高成本扩展阶段。

---

# 5. 第 11–13 周：Human Budget + Online Recalibration

Human budget：

\[
B_H\in\{0,5\%,10\%,20\%,50\%,100\%\}
\]

比较 query strategies：

- Random；
- Lowest raw AI confidence；
- RM↔AI disagreement；
- Highest OOD；
- Lowest calibrated reliability；
- correction uncertainty。

核心图：

\[
Human\ Budget
\rightarrow
Policy\ Success
\]

同时比较：

\[
Static\ Adapter
\]

与：

\[
Online\ / Periodic\ Recalibration
\]

目标是证明：

> 在固定 Human budget 下，当前方法能把人工反馈用在更值得标注的 trajectory pairs 上，并在 policy distribution 改变后保持 alignment。

---

# 6. 第 14–16 周：Robustness + Ablation + 成稿

## 6.1 Robustness

必须考虑：

- random label noise；
- systematic bias；
- OOD；
- paraphrase；
- reward hacking；
- evaluator version / backend replacement。

尤其重点做 systematic bias：

- 偏好更快但不安全的 trajectory；
- 偏好更短 trajectory；
- 对 near-success 过度乐观；
- 忽略 contact/safety；
- late-stage fine-grained preference 判断错误。

## 6.2 Ablation

至少包括：

- no Human anchor；
- no confidence；
- no trajectory context；
- no policy stage；
- no correction head；
- no reliability head；
- no online recalibration；
- no escalation；
- source-wise vs sample-wise trust；
- static vs online；
- raw vs corrected preference。

## 6.3 第 16 周目标

形成一版：

> **可以正式投稿的完整论文稿**

而不是继续无止境加模块。

---

# 7. 目标档位一：3–4 个月完整稿

如果最终完成：

- MetaWorld 4–6 tasks；
- strong baselines；
- Human calibration/correction；
- RM + SAC；
- Human budget；
- policy-stage shift；
- robustness；
- 5 seeds 主实验；

则形成一篇完整机器人学习论文。

### 优先考虑

**RA-L / 同级 JCR Q1 路线**

按照实验室当前奖励制度：

- RA-L 被列为 A 档；
- A 档参考奖励 10000 元；
- 普通 JCR Q1 属于 A- 档；
- A- 参考奖励 8000 元。

这个档位更适合作为当前课题的**第一阶段正式投稿目标**。

---

# 8. 目标档位二：5–7 个月升级版

只有当下面现象足够强时再升级：

\[
Policy\text{-}Induced\ AI\text{-}Human\ Agreement\ Shift
\]

明确存在，并且：

\[
Static
<
Policy\text{-}Conditional
<
Online\ Recalibration
\]

在最终 policy 上也成立。

此时追加：

- ManiSkill / LIBERO；
- 第二类任务；
- 多 evaluator backend；
- 更完整 Human budget study；
- 真机 pilot；
- 更强 OOD / reward hacking test。

### 可考虑

**CoRL / RSS 强度路线**

按照实验室当前制度：

- RSS、CoRL 被列为 A+ 档；
- A+ 参考奖励 12000 元；
- 顶会需要满足实验室关于突出展示、奖项或导师组认定等要求。

这一步不是默认执行，而是由第 12–16 周结果决定。

---

# 9. 目标档位三：6–10 个月以上的长线版本

如果希望进一步冲更高档，需要增加的不应只是“更多任务”，而应形成：

\[
新问题定义
+
强方法
+
多环境
+
真机
+
跨 evaluator 泛化
+
更系统的理论/分析
\]

建议额外加入：

- real robot；
- cross-task transfer；
- cross-evaluator transfer；
- online reliability drift theory / analysis；
- stronger human study；
- long-horizon manipulation；
- failure taxonomy；
- API teacher → local open evaluator transfer baseline。

### 可考虑

**T-RO / IJRR 长稿路线**

按照实验室制度：

- T-RO、IJRR 属于 A++ 档；
- A++ 参考奖励 15000 元；
- 与 Automatica、TAC、TCybernetics、IEEE/CAA JAS 同档。

当前不建议第一天就按 A++ 工作量启动。

更合理的方式：

\[
RA\text{-}L/Q1\ level\ core
\rightarrow
结果强
\rightarrow
再扩展成长稿
\]

---

# 10. 当前投稿决策树

\[
\boxed{
第4周：现象是否成立？
}
\]

如果否：

\[
Stop/Pivot
\]

如果成立：

\[
\boxed{
第8\text{-}10周：Feedback improvement 是否传递到 Policy？
}
\]

如果否：

\[
弱化方法或停止
\]

如果成立：

\[
\boxed{
第12\text{-}16周：形成 RA\text{-}L/Q1 级完整稿
}
\]

如果结果普通但完整：

\[
Submit\ RA\text{-}L/Q1
\]

如果核心现象非常强：

\[
+\ 第二环境
+\ 真机
+\ 多 evaluator
\rightarrow
CoRL/RSS\ 强度
\]

如果进一步形成系统性新问题 + 强泛化 + 真机 + 理论：

\[
T\text{-}RO/IJRR\ 长线版本
\]

---

# 11. 当前最推荐的实际时间目标

## Month 1

目标：

> **证明课题值得做。**

不是写论文。

## Month 2

目标：

> **完成 Feedback Adapter + RM + 初版 RL 闭环。**

## Month 3

目标：

> **完成 Human budget / online calibration / strong baselines。**

## Month 4

目标：

> **完成 robustness、ablation 和第一版完整稿。**

因此：

\[
\boxed{
12\text{-}16周
}
\]

是当前最合理的“完整第一稿”目标。

---

# 12. 资源与节奏控制

不要在 PoC 阶段投入：

- 真机；
- 大规模视觉模型训练；
- 大规模 VLA；
- 多机器人平台；
- 复杂理论证明。

优先投入：

- 可靠 trajectory 数据；
- Human preference；
- strong baselines；
- 多 seed；
- calibration / correction；
- policy-stage generalization。

---

# 13. 与实验室奖励制度对齐

实验室当前档位中：

| 目标 | 实验室档位 | 参考奖励 |
|---|---:|---:|
| T-RO / IJRR | A++ | 15000 元 |
| RSS / CoRL | A+ | 12000 元 |
| RA-L | A | 10000 元 |
| 普通 JCR Q1 / ICRA / IROS 等 Full Paper | A- | 8000 元 |
| JCR Q2 / CCF B / CAA B | B | 5000 元 |

同时注意：

> 实验室执行细则明确：不再资助开源期刊论文和普通国际会议论文，项目特殊安排除外；SCI 分区以文章发表年度分区为准。

因此真正投稿前必须再次确认：

- 当年 JCR / 中科院分区；
- 是否符合实验室资助规则；
- 署名路径；
- 导师组认定；
- 是否存在 OA / APC 问题。

---

# 14. 当前默认目标

现阶段默认目标不是直接冲 A++。

当前默认路线：

\[
\boxed{
4周\ 验证
\rightarrow
12\text{-}16周\ 做出 RA\text{-}L/Q1 级完整稿
}
\]

如果结果特别强：

\[
\boxed{
额外2\text{-}3个月
\rightarrow
第二环境 + 真机 + 多 evaluator
\rightarrow
CoRL/RSS\ 强度
}
\]

如果后续进一步形成：

- 强泛化；
- 真机；
- 明确新问题；
- 系统方法；
- 理论支持；

再考虑：

\[
\boxed{
T\text{-}RO/IJRR
}
\]

---

# 15. 当前行动优先级

现在只做以下事情：

1. MetaWorld 两任务跑通；
2. 保存四个 policy stage；
3. 生成第一批 trajectory pairs；
4. 完成 200–400 对 Human + AI preference；
5. 验证 AI–Human agreement by stage；
6. 验证 raw confidence calibration；
7. 比较 static vs stage-aware calibration；
8. 第 4 周作第一次 Go/No-Go；
9. Go 后再投入 RM + SAC；
10. 第 12–16 周必须形成第一版完整稿。

> **不要因为目标期刊更高，就在研究现象还没成立前提前堆真机、视觉和大模型。**
