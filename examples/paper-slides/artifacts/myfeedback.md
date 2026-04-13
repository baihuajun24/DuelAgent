for v6 Chinese version

1. page 1, don't need to include Chinese name
A: Agree. Remove the Chinese subtitle/name line on page 1.

2. page 2, wrong figure to show (should've shown motivation tool use time occupies whole e2e time)
A: Agree. Current figure is method-oriented, not motivation. Better replace with a simple latency-breakdown / timeline figure showing tool wait dominates end-to-end time.

what does eviction mean? (can you explain from the paper)
A: In the paper, `eviction` means when the model emits a tool call, that request leaves the running batch / KV-cache residency. After the tool returns, it must be rescheduled and loaded back in, often with prompt/tool-output re-prefill. So the cost is not only tool wait, but also API + scheduling + re-entry overhead.
then we should add this concept in to bullet 2: re-prefill path when tool results come back.
A: Yes. Add it explicitly. Better bullet: `Eviction hurts throughput.` / `请求离开 batch；工具结果回来后还要重新调度，并常常走一遍 re-prefill path。`

3. page 2
Prior fixes are incomplete.
并行工具调用和 prefix cache 都只能解决一部分问题
doesn't give any info; what disadvantage of previous methods cause, how that relate to this study here?
A: Agree, current wording is too empty. The point should be: `parallel tool calls` only help when multiple tools are independent, but many agent steps are still sequential; `prefix cache` may save some repeated prefill, but it does not remove batch eviction / rescheduling overhead. That is exactly why this paper studies speculative tool execution: overlap tool wait with generation, and on engine-side avoid eviction + duplicate prefill when possible.

4. page 3 
I think current illustration figure is too large (takes a lot of space)
we could've used Figure 2 overview here to illustrate the point.

5. page 7 the bottom left formula can be removed, I think it serves the same purpose as page 8 formula, but page 8 formula looks better. and it should be made larger across left to right.

v7 feedbacks
keep only English subtitles/headers, remove Chinese parts
page 2 and page 4:
1) I think we can remove right part figure from page 2, it kind of deliver same message as page 4 figure
2) maybe we can combine page 2 and page 4, move page 3 as new page 2, (2+4) becomes new page 3
and we can start with not evidence from related work, but just call it evidence for tool stall (or better name if you can come up with). Then the next illustration figure should explain why tool time occupies such long time and the 3 problems
3) page 5 feels can be reduced as well, it doesn't have enough content for one page. maybe use 2+4+5 into new page 3

page 3: 
1) PASTE should include institutes
2) in slides notes (do we have it?) explain what composes tool init time in Chinese

page 6:
big rendering issue, black and short. I don't know what happened

page 8:
as I said before, try to use the screenshotted formula instead of printing out the formula in html
doesn't look good

Discussions:
in discussion, we can highlight what other related works do? like PASTE, or DuelSpec? what are the key takeaways?
