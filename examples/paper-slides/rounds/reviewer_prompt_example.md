# Task: Compare two presentation versions (neutral review) — FINAL ROUND

You are reviewing two presentation slide decks for the paper "Optimizing Agentic Language Model Inference via Speculative Tool Calls" (Nichols et al., arXiv 2512.15834). Both are intended for a ~25-minute group meeting talk.

This is the FINAL round of an 8-round adversarial competition. Both artifacts have been through multiple revision cycles. Score carefully.

## Step 1: Read the scoring guidelines

Read `./slide_guidelines.md` first. It defines:
- The use case, audience, and design preferences
- The weighted scoring rubric with dimension descriptions
- What 8+ and ≤5 look like for each dimension

Apply these guidelines as your evaluation standard.

## Step 2: Read and evaluate both versions

### Version A: PPTX
File: `./speculative_tool_calls_slides_v5.pptx`

Background: v5 PPTX. Corrected v4's over-compression. Raised font sizes (body ≥20pt, headings ≥28pt), rewrote speaker notes as 3-5 bullet cues with timing, added engine-side runtime intuition slide, takeaway captions on all figures, enlarged figure panels, trimmed experimental setup, intuition-first theory, one-idea-per-slide discipline, better 25min pacing, clearer appendix separation. Won round 7.

### Version B: HTML
File: `./ppt_v5.html`

Background: v5 HTML revision addressing round 7 feedback — raised typography to room-readable sizes, enlarged/simplified pipeline diagram, trimmed experimental setup, split strengths/limitations, shortened discussion cards, compressed summary, reworked backup slides for readability, audited figure legends/axes, increased caption sizes, reclaimed chrome space, improved pacing toward 16-20 slide target.

## Step 3: Write output — TWO separate files

### File 1: Score table only
Write to: `./slide_review_v8.md`

Contents:
1. Score comparison table using the weighted rubric from slide_guidelines.md
2. Weighted totals
3. Verdict: which version is better overall, and a brief (2-3 sentence) justification
4. Since this is the FINAL round, also add a 2-3 sentence note on how both artifacts compare to the very first versions — has quality genuinely improved?

That's it. No detailed feedback in this file.

### File 2: Improvement feedback for the LOSER only
Write to: `./feedback_for_loser_v8.md`

Contents:
- At the top, state which version this feedback is for (A or B)
- At least 8 specific, actionable improvement suggestions grounded in slide_guidelines.md
- CRITICAL RULE: This file must contain ZERO information about the winning version.

## Step 4: Notify the controller

After you have finished writing BOTH output files, run this command to notify the controller session:

```bash
tmux send-keys -t controller "Review round 8 (FINAL) complete. slide_review_v8.md and feedback_for_loser_v8.md are ready. Please proceed." Enter
```

This is mandatory — the controller is waiting for this signal.
