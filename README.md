# DuelAgent：内容生成擂台 — 赢者不动，输者迭代

[English version](README_en.md)

> **DuelAgent: A Winner-Stays Adversarial Framework for AI Content Generation**
>
> 灵感来自 GAN（生成对抗网络）：两个生成器竞争，一个判别器打分，输者改进，赢者不动，循环迭代直到收敛。特别适合**难以自动验证、需要主观评分**的生成任务——PPT、技术文档、设计稿、营销文案等。本仓库以论文组会 PPT 为例展示完整流程。

## 架构

```
                    ┌─────────────────────────────┐
                    │      🎮 Controller（编排者）   │
                    │                             │
                    │  问偏好 → scoring_guide.md   │
                    │  跟踪版本 · 路由反馈 · 推进轮次│
                    └──────┬──────────────┬───────┘
                           │              │
                  ┌────────▼───┐    ┌─────▼────────┐
                  │ Generator A │    │ Generator B  │
                  │ (独立 session) │    │ (独立 session) │
                  │ ⚠️ 看不到 B  │    │ ⚠️ 看不到 A  │
                  └────────┬───┘    └─────┬────────┘
                           │              │
                           ▼              ▼
                    ┌─────────────────────────────┐
                    │       🧑‍⚖️ Discriminator       │
                    │    每轮无状态 · 按标准打分     │
                    └──────────────┬──────────────┘
                                  │
                    ┌─────────────▼──────────────┐
                    │         评审结果              │
                    │  🏆 Winner → 冻结，不动       │
                    │  📝 Loser  → 收到反馈，迭代   │
                    └─────────────┬──────────────┘
                                  │
                                  ▼
                         下一轮：Loser_v{n+1}
                              vs Winner_v{n}
                           （循环直到收敛）
```

**示例场景**：给一篇论文做组会 PPT。Gen A 用 python-pptx 生成 .pptx，Gen B 生成自包含 .html 模拟幻灯片。两种格式各有优势，互相无法抄袭，天然防止模式坍缩。

## 核心规则

**信息隔离** — Generator 永远不知道对手的存在。反馈中零信息泄露，防止模仿导致"模式坍缩"。

**赢者不动，输者迭代** — 每轮只有输者收到反馈并改进，赢者冻结。下一轮用输者的新版本 vs 赢者的旧版本。

**判别器无记忆** — 每轮清空上下文，从零开始评审，避免锚定偏差和分数膨胀。

**评分标准由用户定义** — 开始前 Controller 问用户偏好问题，生成 `scoring_guide.md`（用途、风格、图表、密度、权重等）。判别器每轮按此标准打分。

**事件驱动** — Agent 完成任务后通知 Controller（通过 tmux 回调或等效的 agent 间通信机制），无需轮询，流程自动推进。

## 流程

```
0. 初始化：Controller 问用户偏好 → 生成 scoring_guide.md
1. Controller 写评审 prompt → 清空 Discriminator → 启动评审
2. Discriminator 按 scoring_guide 打分 → 写打分表 + 输者反馈 → 回调 Controller
3. Controller 读取结果 → 只把反馈发给输者
4. 输者改进 → 产出新版本 → 回调 Controller
5. 回到第 1 步（赢者不动，版本号不变）
```

```python
# DuelAgent 伪代码

scoring_guide = controller.ask_user_preferences()  # Round -1
A = gen_a.create(source_material, scoring_guide)    # Round 0
B = gen_b.create(source_material, scoring_guide)

for round in range(1, max_rounds + 1):
    discriminator.clear_context()                   # 无记忆
    scores, loser_feedback = discriminator.review(
        A, B, scoring_guide                         # 按用户标准打分
    )
    winner, loser = identify(scores)

    if human.satisfied(winner.artifact):
        break
    if abs(scores.A - scores.B) < threshold:        # 收敛，继续迭代无意义
        break

    loser.revise(loser_feedback)                    # 只有输者改
    # winner.artifact 不变，版本号不动
```

## 实战战绩

用这个框架给 arXiv 2512.15834（Speculative Tool Calls）做组会 PPT，两个 Agent 对抗 8 轮：

| 轮次 | 对阵 | 赢家 | 比分 | 输者动作 |
|:----:|------|:----:|:----:|---------|
| 1 | A_v1 vs B_v1 | B | 56 vs 48 | A 补真实图表、精简文字 |
| 2 | A_v2 vs B_v2 | B | 57 vs 54 | A 加步骤卡片、配色分区 |
| 3 | A_v3 vs B_v2 | **A** | 59 vs 53 | B 加导航、配色、时间标注 |
| 4 | A_v3 vs B_v3 | **A** | 58.5 vs 55.5 | B 精简密度、放大字体 |
| 5 | A_v3 vs B_v4 | **B** | 62 vs 59 | A 拆分背景页、图表主导 |
| 6 | A_v4 vs B_v4 | **B** | 60.5 vs 54 | A 矫枉过正后恢复节奏 |
| 7 | A_v5 vs B_v4 | **A** | 60.5 vs 57.75 | B 放大字体、拆分页面 |
| **8** | **A_v5 vs B_v5** | **A** | **61 vs 58.5** | **最终轮，竞赛结束** |

**最终结果**：A（PPTX v5）以 61/70 胜出。8 轮对抗中赢家交替 4 次，A 从首轮 48 分爬升到 61，B 从 56 到 58.5。双方都有实质性进步——评审原话："两份作品都从粗糙的初稿演变成了像样的组会演示。"

## 为什么有效

| GAN 概念 | 对应机制 |
|----------|---------|
| 两个 Generator | 两个独立 AI agent，各自探索不同方向 |
| Discriminator | 无状态评审，按用户定义的标准打分 |
| Loss signal | 打分差距 + 定向改进反馈 |
| 防止模式坍缩 | 信息隔离 + 不同输出格式 |
| 收敛判定 | 分数接近 / 人类满意 / 达到最大轮数 |

## 停止条件

- 达到人类设定的最大轮数
- 人类查看当前赢家的产出，觉得满意了
- 双方所有维度 ≥ 8/10，分差 ≤ 1，继续迭代无意义

## 技术栈

- **Controller**: Claude Code（编排 agent 会话）
- **Generator A**: Claude Code → python-pptx
- **Generator B**: Claude Code → HTML/CSS
- **Discriminator**: OpenAI Codex（GPT-5.4，interactive mode）
- **Agent 间通信**: tmux send-keys（或任何 headless agent 编排方案；事件回调，非轮询）
- **状态管理**: Markdown 文件（controller_state.md, scoring_guide.md）

---

*多个 agent 会话，3 个 AI agent，一个评分标准。代价是 token，换来的是两份独立进化、质量收敛的作品。特别适合 PPT、技术文档、设计稿、营销文案——那些没有标准答案、需要主观判断的生成任务。*

## 灵感来源

本项目的设计借鉴了以下 LLM / ML 领域的概念和工作：

- **GAN（Generative Adversarial Networks）** — 生成器 vs 判别器的对抗训练范式 (Goodfellow et al., 2014)
- **Self-Refine** — 单 agent 自我迭代改进 (Madaan et al., 2023)
- **Constitutional AI** — 基于规则的自我纠正 (Bai et al., 2022)
- **LLM-as-a-Judge** — 用 LLM 做自动评审 (Zheng et al., 2023)
- **Chatbot Arena / LMSYS** — 基于 Elo 的 LLM 对战排名 (Zheng et al., 2023)
- **Multi-Agent Debate** — 多 agent 辩论提升推理质量 (Du et al., 2023)
- **Mixture of Agents (MoA)** — 跨模型迭代精炼 (Wang et al., 2024)
- **GADC** — GAN 启发的多 agent 自主软件工程框架 (2025)
- **Tournament Selection** — 进化计算中的锦标赛选择策略
- **Self-Play** — 自博弈提升策略质量 (Silver et al., 2017; SPIN, Chen et al., 2024)
