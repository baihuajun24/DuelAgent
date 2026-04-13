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
