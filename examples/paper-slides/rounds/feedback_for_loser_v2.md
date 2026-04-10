# Improvement Feedback for Version A (PPTX)

This feedback is for **Version A: PPTX slide deck**
File: `./speculative_tool_calls_slides_v2.pptx`

## Improvement Suggestions

1. **Improve visual hierarchy and information density balance.** Several slides (e.g., Slide 8 — engine-side runtime equations, Slide 10 — experimental setup) pack many bullet points at similar font sizes with little visual differentiation. Use larger/bolder text for the single key takeaway per slide and push details into smaller supporting text or speaker notes.

2. **Add a dedicated Background / Pipeline slide.** The deck jumps from Motivation (Slide 2) straight into Speculative Decoding vs. Tool Calling (Slide 4). A slide explicitly showing the standard Prefill → Decode → Tool Call → Re-prefill → Decode cycle as a visual timeline would ground the audience before introducing the solution.

3. **Introduce visual design elements beyond plain bullet lists.** Most body slides use the same layout: header bar + bullet list. Consider step cards, numbered process flows, side-by-side comparison boxes, or callout/takeaway boxes for variety and to signal different types of content (process, comparison, data, theory).

4. **Add a progress indicator.** Across 16 slides for a 25-minute talk, a subtle progress bar or section labels ("Motivation → Method → Theory → Results → Discussion") would help the audience orient themselves in the narrative arc.

5. **Strengthen the formula slides with more whitespace and annotation.** Slides 6 and 8 present runtime equations, but the formulas compete with surrounding text. Give formulas a dedicated framed box with labeled terms beneath, and reduce surrounding bullets to one or two.

6. **Include appendix or backup slides.** There is no appendix material. Adding 1–2 backup slides (e.g., proof sketch for the <2× bound, detailed API spec, or full runtime derivation) would prepare the presenter for deeper audience questions without crowding the main deck.

7. **Enhance the Discussion slide (Slide 14).** The current three-column layout (Strengths / Limitations / Questions) is dense. Consider splitting Strengths+Limitations and Discussion Questions into two separate slides so the audience can focus on each.

8. **Make color usage more intentional.** The deck uses green takeaway boxes and blue headers, but the accent palette is limited. Assign specific colors to recurring categories (e.g., client-side = blue, engine-side = orange, theory = grey) and use them consistently in headers, callout borders, and figure labels.

9. **Polish figure captions to be self-contained.** Several figure labels (e.g., "Fig 2: Speedup distribution across α, g/G, and T") summarize but do not state the conclusion. Rewrite captions to include the one-sentence takeaway so the figure is understandable even without the speaker's narration.

10. **Add timing guidance in speaker notes.** While all 16 slides have Chinese notes, none mention target duration. Adding "(~2 min)" markers helps the presenter pace a 25-minute slot and know which slides to compress if running late.
