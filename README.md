# DuelGAN：内容生成擂台 — 赢者不动，输者迭代

[English version](README_en.md)

> **DuelGAN: A Winner-Stays Adversarial Framework for AI Content Generation**
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
# DuelGAN 伪代码

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

## 踩过的坑 & 学到的事

**"改进"不等于"更好"。** A 在第 6 轮从 59 跌到 54——按反馈大刀阔斧压缩内容，结果矫枉过正，叙事断裂、演讲者备注被砍空。好消息是下一轮的反馈拉回了正轨（54→60.5）。对抗迭代有自我纠错能力，但需要足够的轮次来恢复。

**GAN 类比好用但别当真。** 这不是真正的零和博弈——没有参数更新，没有对抗分布匹配。更准确的说法是"基于挑战者的偏好搜索"。GAN 类比帮助设计直觉，但不应声称理论保证。

**评审格式偏差是真实存在的。** .pptx 和 .html 的渲染差异会影响判别器打分。更公平的做法是先统一渲染成 PDF 截图再评审——这是下一步要改的。

**回调通知必须和任务写在同一条消息里。** 我们踩了一个坑：把 "完成后通知 Controller" 作为单独的 follow-up 消息发给 Agent，结果 Agent 把它当成了新任务。完成通知指令必须嵌入原始任务 prompt——这是 agent 编排的通用原则，不限于特定通信方式。

**判别器无记忆 = 高方差。** 每轮清空上下文避免了锚定偏差，但也引入了评分不一致。同一份作品在不同轮次可能得到不同分数。未来考虑评审集成（多个 judge 取平均）。

**赢者冻结可能掩盖隐藏缺陷。** 如果判别器连续漏评某个问题，赢者可以带着缺陷冻结多轮。需要一个"冠军保质期"机制——比如连赢 3 轮后强制重新评审。

---

*多个 agent 会话，3 个 AI agent，一个评分标准。代价是 token，换来的是两份独立进化、质量收敛的作品。特别适合 PPT、技术文档、设计稿、营销文案——那些没有标准答案、需要主观判断的生成任务。*
