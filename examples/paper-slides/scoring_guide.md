# Slide Generation Guidelines & Scoring Rubric

> These guidelines define what "good slides" look like for our use case. Both generators and the reviewer/discriminator should follow them.

## Use Case

- **Format**: Group meeting / lab talk, ~25 minutes
- **Audience**: Peers who know the field — no need to explain basics like transformers or KV-cache from scratch
- **Goal**: Present one paper clearly, spark discussion

## Design Preferences

### Language
- Mixed freely — use whichever language is clearest for each part
- Chinese in speaker notes is fine and encouraged for the presenter's comfort
- Slide titles and key terms can stay in English for consistency with the paper

### Visual Style: Clean & Minimal
- Lots of whitespace, restrained color palette (2-3 accent colors max)
- Let figures and key numbers speak — don't decorate for decoration's sake
- Avoid visual clutter: no gratuitous icons, gradient backgrounds, or drop shadows
- Consistent layout across slides — same header style, same margin, same font sizes

### Text Density: Sparse — talk > read
- Max 3-4 bullet points per content slide
- One core idea per slide
- Big font — body text ≥ 20pt, headings ≥ 28pt
- If a slide has a figure, text should be minimal (caption + 1-2 bullets max)
- Details belong in speaker notes, not on the slide

### Figures
- **Must use original paper figures** — embed actual PNGs from the PDF
- Figures should be large and central, not squeezed into a corner
- Every figure needs a caption that states the **takeaway**, not just the title
- Example: "Client-side saves 6-21% — sweet spot at T ≈ G" not "Figure 3: Time saved vs tool latency"

### Math
- **Intuition first, formula as support**
- Lead with the insight in plain language, then show the equation
- Key formulas get a clean framed box with variable definitions nearby
- Skip full derivations on slides — put proof sketches in appendix if needed
- Never show a formula without explaining what it means in words

### Speaker Notes
- **Bullet-point cues** — not a full script, not empty
- 3-5 short bullets per slide: what to say, key transition to next slide
- Include timing hints for pacing (e.g., "~2 min", "skip if short on time")
- Chinese is fine in notes

### Slide Count & Pacing
- **16-20 slides** for 25 minutes (~1.5 min each)
- Include 1-2 appendix/backup slides for Q&A (clearly marked)
- Title slide + thank you slide don't count toward content budget

## Scoring Rubric (for Discriminator)

Score each version 1-10 on the following dimensions. Apply the guidelines above as the standard.

| Dimension | Weight | What 8+ looks like | What ≤ 5 looks like |
|-----------|:------:|--------------------|--------------------|
| **Content completeness** | 1.0 | Covers problem, method (client + engine), key theory result, experiments, limitations. Nothing major missing. | Skips a core contribution or has factual errors. |
| **Visual clarity** | 1.5 | Clean, minimal layout. Consistent styling. Whitespace used well. Easy to read from back of room. | Cluttered, inconsistent fonts/colors, walls of text, hard to parse. |
| **Figure quality** | 1.5 | Original paper figures, large, with takeaway captions. Properly placed next to relevant content. | Missing figures, placeholder text, tiny images, or no captions. |
| **Narrative flow** | 1.0 | Motivation → insight → method → theory → results → discussion. Smooth transitions, logical order. | Jumps between topics, no clear story arc, audience gets lost. |
| **Text economy** | 1.0 | Sparse text, one idea per slide, details in notes. Big readable font. | Dense bullet lists, multiple ideas crammed per slide, small font. |
| **Speaker support** | 0.5 | Bullet-point cues in notes, timing hints, Chinese annotations where helpful. | No notes, or notes are just copy of slide text. |
| **Portability** | 0.5 | File works on common platforms, figures embedded, no broken dependencies. | Broken images, platform-specific issues, missing fonts. |

### Scoring Formula

**Weighted total = Σ (score × weight)** out of a max of **70** (7 dimensions × 10 × weight).

Actual max with weights above: 10 × (1.0 + 1.5 + 1.5 + 1.0 + 1.0 + 0.5 + 0.5) = **70**.

### Reviewer Instructions

When reviewing, the discriminator should:
1. Read these guidelines first
2. Evaluate each version independently against the guidelines (not against each other)
3. Apply scores based on the rubric above
4. Write feedback for the loser using only absolute quality suggestions — never reference the winner
