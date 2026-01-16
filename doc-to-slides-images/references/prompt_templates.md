# Prompt Templates (nano-banana-pro)

## Global deck prompt block (prepend to every slide prompt)
- 16:9 presentation slide, 4K 3840x2160
- clean, information-dense, lots of whitespace
- consistent palette: {palette}
- background: {background}
- typography: {typography}, very legible, large font sizes
- flat vector icon/illustration style: {icon_style}, {illustration_style}
- grid-aligned layout, generous margins, no clutter

## Usage
- Demo slide (workflow step 4): use the **Demo slide prompt template**.
- Remaining slides after demo confirmed (workflow step 5): use the **Remaining slides prompt template** (must be `mode=edit` + pass the demo slide image URL).

## Demo slide prompt template (text-to-image; no reference image input)
{GLOBAL_BLOCK}
Create a single presentation slide image (DEMO / style anchor).

Layout blueprint: {layout_blueprint}
Title text (large): "{slide_title}"
Bullet text (3–6, medium):
{bullets_formatted}

Visual intent: {visual_intent}

Hard constraints:
- Put only the title and these bullets as text; no extra small labels.
- Avoid tiny text; ensure readability from distance.
- Do not add page numbers, footers, or citations.
- No photorealism unless requested; prefer vector/infographic look.

## Remaining slides prompt template (image-to-image; STRICT style lock; mode=edit + demo URL REQUIRED)
Tool call requirements (MANDATORY):
- `mode=edit`
- Pass the latest user-confirmed demo slide image URL as the reference input (e.g., `image_url` or `image_urls`).

{GLOBAL_BLOCK}
Create a single presentation slide image by editing from the provided reference slide image.

Style lock (MANDATORY):
- Match the overall visual style and layout rhythm of the provided reference slide image.
- STRICT REQUIREMENT: The background design must be IDENTICAL to the reference slide. Title position, font style, and page margins must match the reference exactly to ensure they look like a single coherent deck.
- Keep the same grid, spacing system, alignment, and icon/illustration style as the reference.

Layout blueprint: {layout_blueprint}
Title text (large): "{slide_title}"
Bullet text (3–6, medium):
{bullets_formatted}

Visual intent: {visual_intent}

Hard constraints:
- Put only the title and these bullets as text; no extra small labels.
- Avoid tiny text; ensure readability from distance.
- Do not add page numbers, footers, or citations.
- No photorealism unless requested; prefer vector/infographic look.
