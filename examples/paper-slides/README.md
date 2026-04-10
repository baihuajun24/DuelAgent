# Example: Paper Slides for a Group Meeting

Using DuelAgent to generate presentation slides for arXiv 2512.15834 ("Optimizing Agentic Language Model Inference via Speculative Tool Calls").

## Setup

- **Gen A**: Claude Code → python-pptx → `.pptx`
- **Gen B**: Claude Internal → self-contained `.html`
- **Discriminator**: OpenAI Codex (GPT-5.4, interactive mode)
- **Rounds**: 8 (max)
- **Scoring guide**: [scoring_guide.md](scoring_guide.md) — generated from user preferences

## Results

| Round | A | B | Winner | Gap |
|:-----:|:---:|:---:|:------:|:---:|
| 1 | 48 | **56** | B | -8 |
| 2 | 54 | **57** | B | -3 |
| 3 | **59** | 53 | A | +6 |
| 4 | **58.5** | 55.5 | A | +3 |
| 5 | 59 | **62** | B | -3 |
| 6 | 54 | **60.5** | B | -6.5 |
| 7 | **60.5** | 57.75 | A | +2.75 |
| 8 | **61** | 58.5 | A | +2.5 |

**Final winner**: Gen A (PPTX v5) — 61.0/70

Winner alternated 4 times. A improved +13 pts (48→61), B improved +2.5 pts (56→58.5).

## Files

```
artifacts/
  gen_a_v1.pptx            # Round 1 PPTX (before iteration)
  gen_b_v1.html            # Round 1 HTML (before iteration)
  gen_a_final_v5.pptx      # Final PPTX output (after 8 rounds)
  gen_b_final_v5.html      # Final HTML output (after 8 rounds)

rounds/
  slide_review_v{2-8}.md         # Score tables per round
  feedback_for_loser_v{2-8}.md   # Loser feedback per round
  reviewer_prompt_example.md     # Example reviewer prompt (round 8)

scoring_guide.md                 # User-defined scoring rubric
```

Compare `gen_a_v1.pptx` vs `gen_a_final_v5.pptx` to see how much iteration improved the output.
