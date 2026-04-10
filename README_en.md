# DuelGAN: A Winner-Stays Adversarial Framework for AI Content Generation

[中文版](README.md)

> A GAN-inspired adversarial framework where two LLM agents compete, one discriminator scores, and only the loser iterates. Best suited for **subjective deliverables that resist automated verification** — slides, documents, design drafts, copywriting — where quality requires human-like judgment. This repo demonstrates the full workflow with a paper presentation case study.

## Architecture Overview

```
                    ┌─────────────────────────────┐
                    │      🎮 Controller            │
                    │                             │
                    │  Ask preferences → rubric   │
                    │  Track versions · Route feedback │
                    └──────┬──────────────┬───────┘
                           │              │
                  ┌────────▼───┐    ┌─────▼────────┐
                  │ Generator A │    │ Generator B  │
                  │ (isolated)  │    │ (isolated)   │
                  │ ⚠️ Can't see │    │ ⚠️ Can't see │
                  │    B's work │    │    A's work  │
                  └────────┬───┘    └─────┬────────┘
                           │              │
                           ▼              ▼
                    ┌─────────────────────────────┐
                    │       🧑‍⚖️ Discriminator       │
                    │  Stateless · Scores by rubric │
                    └──────────────┬──────────────┘
                                  │
                    ┌─────────────▼──────────────┐
                    │          Result              │
                    │  🏆 Winner → frozen          │
                    │  📝 Loser  → gets feedback   │
                    └─────────────┬──────────────┘
                                  │
                                  ▼
                         Next round: Loser_v{n+1}
                              vs Winner_v{n}
                          (loop until converged)
```

## Agents

### Generator A — PPTX Agent
- **Binary**: `claude` (standard)
- **tmux session**: `gen-a` (example; any headless agent orchestration works)
- **Output format**: `.pptx` via `python-pptx`
- **Strengths**: Editable in PowerPoint/Keynote, speaker notes, programmatic layout control
- **Working dir**: `<project_root>/`

### Generator B — HTML Agent
- **Binary**: `claude`
- **tmux session**: `gen-b` (example; any headless agent orchestration works)
- **Output format**: Self-contained `.html` simulating 1280×720 slides
- **Strengths**: Figures embedded inline, browser-viewable, printable to PDF via Cmd+P
- **Working dir**: `<project_root>/`

### Discriminator D — Review Agent
- **Binary**: `codex` (interactive mode)
- **tmux session**: `codex-critique` (example; any headless agent orchestration works)
- **Setup**: Enter `codex` first to start interactive mode, then send review prompts via inter-agent messaging (e.g., `tmux send-keys`)
- **Output**: Two files per round — score table + loser feedback (see Information Isolation)
- **Key property**: **Stateless** — context cleared before every round. D never sees its own previous reviews. Prevents anchoring bias.

## Protocol

### Round -1: Preference Elicitation → `scoring_guide.md`

Before any generation or review begins, the controller asks the human a series of preference questions to build a scoring guide. These questions are **task-agnostic** — the same elicitation pattern works whether you're generating slides, documents, code, or anything else.

**For slides/presentations**, example questions:

1. **Use case** — group meeting, conference talk, tutorial, defense?
2. **Language** — English slides + Chinese notes, all English, mixed freely?
3. **Figure policy** — must use paper figures, recreated diagrams, or mix?
4. **Visual style** — clean & minimal, information-dense, visual-heavy, academic standard?
5. **Text density** — sparse (talk > read), moderate, dense (standalone readable)?
6. **Math depth** — intuition first, show full math, avoid formulas?
7. **Speaker notes** — detailed script, bullet-point cues, minimal?
8. **Slide count** — fewer/deeper, standard pace, more/lighter?

**For other content types**, the controller adapts the questions to the domain (e.g., code style, documentation depth, tone of voice for copywriting).

Based on answers, the controller writes `scoring_guide.md` containing:
- Design preferences derived from user choices
- Weighted scoring rubric with dimension descriptions and anchor examples (what 8+ and ≤5 look like)
- Reviewer instructions

This file is **fixed for the entire competition** — both generators and the reviewer reference it every round. The rubric lives in one place, never duplicated in prompts.

### Round 0: Initialization
1. Human provides paper source material to both generators
2. Both generators read `slide_guidelines.md` for design preferences
3. Generator A produces `slides_v1.pptx`
4. Generator B produces `ppt_v1.html`
5. Both outputs placed in `<project_root>/`

### Round N: Competition Loop (Winner-Stays, Loser-Challenges)

The winner's artifact is frozen — only the loser revises. Each round compares the loser's **new** version against the winner's **old** (unchanged) version.

**Version tracking**: The controller tracks `A_version` and `B_version` independently. They do NOT advance in lockstep.

```
1. CLEAR discriminator context
   └─ Human/controller clears reviewer session (fresh start, no memory)

2. WRITE review prompt → reviewer_prompt_v{N}.md
   Contents:
   - Reference to scoring_guide.md (reviewer MUST read this first)
   - Path to A's current artifact (speculative_tool_calls_slides_v{A_version}.pptx)
   - Path to B's current artifact (ppt_v{B_version}.html)
   - Background on version history (what was changed), but D does NOT read old versions
   - Scoring rubric is IN scoring_guide.md — do not duplicate, just point to it
   - Output file paths for round N
   - CRITICAL: no reference to any previous review
   - Callback instruction: after writing both files, D must notify the controller
     (e.g., `tmux send-keys -t controller "Review round {N} complete..." Enter`)

3. SEND prompt to D (already in interactive mode)
   # Example using tmux (any headless agent messaging works):
   # ⚠️ GOTCHA: text and submit must be separate send-keys calls
   tmux send-keys -t codex-critique \
     "Read ./reviewer_prompt_v{N}.md and follow its instructions. \
     Read the scoring guidelines, both presentation files, then write the two output files."
   tmux send-keys -t codex-critique Enter

4. WAIT for D callback
   D notifies the controller session when done (e.g., via tmux send-keys).
   No polling needed — the controller receives the notification directly.

5. IDENTIFY the loser (lower total score)

6. ROUTE feedback — ONLY to the loser
   - Send feedback_for_loser_v{N}.md to losing generator
   - MUST include callback instruction: when done, notify controller
   - Example: tmux send-keys -t {gen-a|gen-b} "..." Enter
   - Loser produces its next version (v{X+1}) then calls back
   - Winner does NOTHING — its artifact is frozen until it loses a round

7. WAIT for loser callback
   Loser notifies the controller session when revision is done.

8. UPDATE version tracker
   - Increment only the loser's version counter
   - Winner's version stays the same

8. INCREMENT N, GOTO step 1
```

**Example progression**:
```
Round 1: A_v1 vs B_v1 → B wins → A revises → A_v2
Round 2: A_v2 vs B_v1 → B wins → A revises → A_v3
Round 3: A_v3 vs B_v1 → A wins → B revises → B_v2
Round 4: A_v3 vs B_v2 → ...
```

### Scoring Rubric

Defined in `slide_guidelines.md` (generated during Round -1 preference elicitation). The rubric includes:
- 7 weighted dimensions (visual clarity and figure quality weighted higher)
- Anchor descriptions for what 8+ and ≤5 look like per dimension
- Weighted total out of 70

The rubric is **not duplicated** in reviewer prompts — the reviewer reads `slide_guidelines.md` directly each round.

### Stopping Conditions
- **Score convergence**: Both generators within 1 point on all dimensions
- **Score threshold**: Both ≥ 8/10 on all dimensions
- **Human satisfaction**: Human previews both and picks a winner
- **Max rounds**: Hard cap (e.g., 5 rounds) to prevent infinite loop

## Principles

### 1. Information Isolation (Anti-Mode-Collapse)

The most critical design choice: **generators never learn about each other**.

D writes two files per round:

| File | Reader | Contains info about competitor? |
|------|--------|------|
| `slide_review_v{N}.md` | Human only | ✅ Yes (scores both, verdict) |
| `feedback_for_loser_v{N}.md` | Losing generator only | ❌ **ZERO** |

**No winner feedback file.** The winner's artifact is frozen and does not change until it loses a future round.

**Rule**: Feedback files contain ZERO information about the other version. No mentions of the competitor's name, format, approach, structure, or qualities. All suggestions are phrased as absolute improvements ("add a summary slide") never relative comparisons ("add a summary slide like the other version has").

**Why**: Without this, the losing generator would learn the winner's strategy and mimic it — the GAN equivalent of **mode collapse** (both generators converging to the same output). By isolating feedback, each generator improves along its own axis (PPTX becomes a better PPTX, HTML becomes a better HTML), preserving diversity and giving the human two genuinely different high-quality artifacts.

### 2. Discriminator Statelessness

- Context cleared before every round — no anchoring to previous scores
- D never reads its own past reviews
- Each round is a fully independent evaluation
- Prevents: score inflation, implicit favoritism from prior rounds, confirmation bias

### 3. Format Diversity

Two different output formats by design:
- Prevents trivial copying — A can't just replicate B's approach
- Each format has structural advantages (PPTX: speaker notes, native editing; HTML: inline figures, browser rendering)
- The human gets two usable, complementary artifacts
- Tests genuine understanding of the paper vs template matching

### GAN Analogy

| GAN Concept | Our System |
|-------------|-----------|
| Generator | Claude Code agents (A and B) producing slides |
| Discriminator | OpenAI Codex / GPT-5.4 scoring and critiquing |
| Loss signal | Score gap + isolated improvement feedback (loser only) |
| Training step | Loser revises based on D's feedback; winner frozen |
| Mode collapse prevention | Information isolation + different output formats |
| Convergence | Both generators produce high-quality, competitive output |

## Reviewer Prompt Template

Each round, `reviewer_prompt_v{N}.md` follows this structure:

```markdown
# Task: Compare two presentation versions (neutral review)

## Step 1: Read the scoring guidelines
Read `/path/to/slide_guidelines.md` first. It defines:
- The use case, audience, and design preferences
- The weighted scoring rubric with dimension descriptions
- What 8+ and ≤5 look like for each dimension

Apply these guidelines as your evaluation standard.

## Step 2: Read and evaluate both versions

### Version A: PPTX
File: [path to latest A]
Background: [version history, what changed — D does NOT read old versions]

### Version B: HTML
File: [path to latest B]
Background: [version history, what changed]

## Step 3: Write output — TWO separate files

### File 1: Score table only
Write to: slide_review_v{N}.md
- Score comparison table using the weighted rubric from slide_guidelines.md
- Weighted totals
- Verdict (2-3 sentences)
- No detailed feedback

### File 2: Improvement feedback for the LOSER only
Write to: feedback_for_loser_v{N}.md
- State which version (A or B)
- ≥ 8 specific, actionable improvements grounded in slide_guidelines.md
- CRITICAL: ZERO info about the winner
```

## Round History

| Round | A version | B version | Comparison | Result |
|-------|-----------|-----------|------------|--------|
| 0 | v1 | v1 | A_v1 vs B_v1 | Initial generation |
| 1 | v1 | v1 | A_v1 vs B_v1 | B won (56 vs 48). A's figures scored 1/10. |
| 1→ | v2 | v1 | — | A revised: embedded PNGs, reduced text, summary slide |
| 2 | v2 | v2 | A_v2 vs B_v2 | B won (57 vs 54). B's visual polish ahead. |
| 2→ | v3 | v2 | — | A revised: step cards, color-coded sections, appendix, timing notes |
| 3 | v3 | v2 | A_v3 vs B_v2 | A won (59 vs 53). Tables turned — A's visual design now ahead. |
| 3→ | v3 | v3 | — | B revised: color-coded sections, navigation, timing notes, step cards, callout variants |
| 4 | v3 | v3 | A_v3 vs B_v3 | A won (58.5 vs 55.5 weighted). A ahead on visual clarity & text economy. |
| 4→ | v3 | v4 | — | B revised: reduced density, larger fonts, figure-dominant results, simplified palette |
| 5 | v3 | v4 | A_v3 vs B_v4 | B won (62.0 vs 59.0 weighted). B overtook on visual clarity & text economy. |
| 5→ | v4 | v4 | — | A revised: split background, compressed comparison, figure-dominant results, pruned appendix |
| 6 | v4 | v4 | A_v4 vs B_v4 | B won (60.5 vs 54.0 weighted). A over-compressed, lost on flow & speaker support. |
| 6→ | v5 | v4 | — | A revised: restored pacing, font sizes, speaker notes, engine-side intuition slide |
| 7 | v5 | v4 | A_v5 vs B_v4 | A won (60.5 vs 57.75 weighted). A recovered on visual clarity & text economy. |
| 7→ | v5 | v5 | — | B revised: raised typography, enlarged pipeline diagram, trimmed setup, split slides, compressed summary |
| **8** | **v5** | **v5** | **A_v5 vs B_v5** | **A won (61.0 vs 58.5 weighted). FINAL ROUND.** |

**COMPETITION COMPLETE.** Final winner: A (PPTX, v5) at 61.0/70. Both artifacts improved substantially from round 1 (A: 48→61, B: 56→58.5).

## File Naming Convention

```
<project_root>/
├── good-examples/2512.15834/                 # Paper source + figures (incl. .png conversions)
│
├── slide_guidelines.md                       # Preferences + weighted rubric (fixed for entire run)
│
├── speculative_tool_calls_slides_v{X}.pptx   # Gen A, current version
├── ppt_v{Y}.html                             # Gen B, current version
│
├── reviewer_prompt_v{N}.md                   # Prompt sent to D each round
├── slide_review_v{N}.md                      # D's scores + verdict only (human reads)
├── feedback_for_loser_v{N}.md                # D's feedback for loser (no info about winner)
│
├── generate_slides_v{X}.py                   # Gen A's generation script
│
├── archive/                                  # Old versions and stale files
│   ├── v1/                                   # Round 0-1 artifacts
│   └── stale/                                # Aborted/invalid review outputs
│
└── multi-agent-GAN.md                        # This file
```

## Quick Commands Reference

The example below uses tmux as the inter-agent communication layer. Any headless agent orchestration that supports sending messages to agent sessions will work.

```bash
# --- Session setup (tmux example) ---
tmux new-session -d -s gen-a
tmux new-session -d -s gen-b
tmux new-session -d -s codex-critique
# Then start codex in codex-critique:
tmux send-keys -t codex-critique "codex" Enter

# --- Send to generators (claude) ---
tmux send-keys -t gen-a "{prompt}" Enter
tmux send-keys -t gen-b "{prompt}" Enter

# --- Discriminator: reset context + send review prompt ---
# D is already in codex interactive mode
#
# ⚠️ GOTCHA 1: /clear ends the session and starts a fresh one.
# Any send-keys during the transition will be LOST.
# Must wait ~5s for the new session to initialize.
tmux send-keys -t codex-critique "/clear" Enter
sleep 5   # wait for new session
#
# ⚠️ GOTCHA 2: text + Enter in one send-keys only TYPES into the buffer.
# Must send a SEPARATE Enter to actually submit.
# If prompt doesn't start processing, resend — transition may need more time.
tmux send-keys -t codex-critique \
  "Read ./reviewer_prompt_v{N}.md and follow instructions. \
  Read the scoring guidelines and both presentations, then write the two output files."
sleep 1
tmux send-keys -t codex-critique Enter

# --- Check if review is done ---
ls -la ./slide_review_v{N}.md
ls -la ./feedback_for_loser_v{N}.md

# --- Feed loser its isolated feedback (winner does nothing) ---
# ⚠️ MUST include callback instruction IN THE SAME MESSAGE as the task
# Sending callback as a separate follow-up will be treated as a new prompt!
tmux send-keys -t {gen-a|gen-b} "Read ./feedback_for_loser_v{N}.md and implement all suggestions. Save as v{X+1}. When completely done, run: tmux send-keys -t controller 'Gen-{a|b} round {N} revision complete. {artifact} is ready. Please proceed.' Enter" Enter
```
