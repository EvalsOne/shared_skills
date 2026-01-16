---
name: doc-to-slides-images
description: Generate NotebookLM-like slide images (PNG) from user-provided documents, web articles/URLs, or pasted text. Use when the user asks to create slides, a slide deck as images, or NotebookLM-style slides. Produces 8–12 slides by default (max 20), 16:9 4K. Style is routed by document type with a clean default and user overrides. Render via an image-generation tool call (default nano-banana-pro; user-specified model/tool wins; no local API calls).
---

# Doc → Slides Images (PNG)

## Output contract
- 8–12 slides by default; cap at 20 unless user explicitly requests more.
- 16:9, 4K (3840×2160).
- Output: `slide_plan.json` + slide images (PNG) in order.
- Slide 1 must be a cover; the last slide must be an ending slide.
- No page numbers, no footers/headers, no logos.
- If you include key numeric facts, you may add a short inline source in the same bullet (e.g., `(..., Source: ...)`).

## Workflow
0) Read references (MANDATORY before any rendering)
- Read these files carefully before deciding layouts or writing any `image_prompt`:
  - `references/slide_json_schema.json`
  - `references/slide_blueprints.md`
  - `references/prompt_templates.md`
  - `references/style_router.md`
- Use the default image tool/model `nano-banana-pro` unless the user explicitly specifies another model/tool.

1) Ingest & normalize input (document / URL / pasted)
- Extract readable text, remove boilerplate, deduplicate.
- If long: chunk → summarize per chunk → merge to unified outline.
- Capture metadata: title, doc type guess, audience, tone.
- Default language follows user request; otherwise follow source content language.
- Assume the deck is meant to be readable and shareable without a live presenter (self-explanatory slides).

2) Decide style + write STYLE INSTRUCTIONS (global)
- Use default “NotebookLM clean deck” unless router strongly suggests otherwise.
- If user specifies style, user wins.
- Read `references/style_router.md` when choosing style.
- Create a `STYLE INSTRUCTIONS` code block that captures the global design decisions (background + hex, palette, typography, icon/illustration style, and layout principles). Treat it as the single source of truth for consistency.

3) Build slide plan JSON
- Follow `references/slide_json_schema.json`.
- Choose 8–12 slides unless content is very long; never exceed 20 by default.
- For each slide, choose a `layout_blueprint` from `references/slide_blueprints.md`.
- Build per-slide `image_prompt` using `references/prompt_templates.md`.
- Slide titles must read like human narrative sentences:
  - Avoid `Title: Subtitle` colon patterns.
  - Avoid clichés and “AI slop” phrasing; use direct, confident, active language.
- Include enough context on-slide so each slide can be understood standalone (within the legibility constraints).
- **Cover + ending slides must look different**:
  - Cover slide: use `TITLE` blueprint (big title, minimal text).
  - Ending slide: use `SUMMARY` or `CTA` blueprint (distinct layout from content slides).
- Ending slide must NOT be generic (“Thank you”, “Any questions?”). Use a designed closing statement, meaningful quote (if available), or a visual summary that anchors the narrative.
- In each slide’s `notes`, include (as text) these 4 sections to help QA and iteration:
  - `NARRATIVE GOAL` (why this slide exists)
  - `KEY CONTENT` (exact on-slide text; include short sources for key numeric facts)
  - `VISUAL` (what to draw)
  - `LAYOUT` (hierarchy and composition; align to the chosen blueprint)

4) Render a demo slide first (MANDATORY gate)
- Explicitly output the full `slide_plan.json` and the `STYLE INSTRUCTIONS` block for user review.
- Pick one representative **content** slide from `slide_plan.json` (NOT the cover or ending slide).
- Render **only this one slide** first (using the selected image tool/model) and show it to the user for confirmation.
- If the user requests any style/layout changes, **regenerate the demo slide** and ask for confirmation again.
- Treat the **latest user-confirmed demo slide** as the single source of truth: `FINAL_DEMO_SLIDE` (older demo renders are obsolete).
- When a new demo slide version is confirmed, **overwrite/replace** `FINAL_DEMO_SLIDE` with that latest PNG output (the exact image you will pass as the reference input).
- Do not render the remaining slides until the user explicitly confirms the plan + style + `FINAL_DEMO_SLIDE`.

5) After user confirms: batch render the remaining slides (MANDATORY consistency)
- For every remaining slide:
  - Call the image generation tool/model (default: `nano-banana-pro`; if the user specified another model/tool, use that).
  - Use the slide’s `image_prompt`.
  - **Always use image-to-image edit mode** (e.g., `mode=edit`) and **pass `FINAL_DEMO_SLIDE` as the reference image input** to the tool call (if the demo slide was regenerated, use the regenerated one).
  - Explicitly require the generated slide to keep the **same base style and background** as the demo slide.
  - Require: 16:9, 4K PNG output.
- If you cannot attach `FINAL_DEMO_SLIDE` as the reference image input (missing/expired/not available), stop and re-render/re-confirm the demo slide first.
- You may render the remaining slides in parallel.
- Do NOT use local scripts to call Gemini/Vertex/HTTP APIs.

6) QA & regenerate if needed
- If text illegible, regenerate with stricter prompt:
  - larger font, fewer words, higher contrast, more whitespace, vector look.
- Keep consistent palette, typography feel, iconography across the deck.
- If any slide drifts in background/style, regenerate it with the demo slide reference image and stricter “match reference” constraints.

## Prompting rules (must follow)
- Prefer vector/infographic presentation style unless user asks photorealistic.
- Only include requested on-slide text: title + bullets; no extra labels.
- Keep bullets short (≤12 words), 3–6 bullets.
- Enforce safe margins and grid alignment.
- No footer/header/page number.
- If sources are needed for key facts, include them inline inside the relevant bullet text (not as a footer or citation marker).
- If an inline source would make the slide less readable, keep the source in `notes` only (and keep on-slide text clean).
- Avoid colon-title patterns and generic endings (“Thank you”, “Any questions?”).
- After demo approval (including after any re-generation), always include “match the reference image’s base style + background” constraints and provide `FINAL_DEMO_SLIDE` as the reference image input for **every** subsequent slide generation call.

## Bundled resources
- Layouts: `references/slide_blueprints.md`
- Style routing: `references/style_router.md`
- Prompt templates: `references/prompt_templates.md`
- JSON schema: `references/slide_json_schema.json`
