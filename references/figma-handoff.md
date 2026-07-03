# Figma handoff reference

How to get generated screens into Figma as **editable layers** (not flat images).

## Primary path: Figma MCP `generate_figma_design`

Figma's official MCP server includes `generate_figma_design`: it sends UI (HTML/CSS) into Figma as fully editable layers.

What the conversion does (per Figma docs):
- HTML elements → Figma frames/shapes
- CSS flexbox/grid → **Auto-layout** groups
- Text nodes → editable text layers with correct font, size, weight, color
- Images → extracted and embedded as image fills

Because of this mapping, the quality of the Figma output = the quality of the HTML:
- Use flexbox/grid for ALL layout (absolute positioning converts poorly).
- Name major blocks with meaningful classes — they become layer names the designer sees.
- Load real fonts (Google Fonts `<link>`) so text layers carry the right typeface.
- Keep nesting shallow; every wrapper div becomes a layer the designer has to wade through.

Operational notes:
- **Read the tool's parameter schema at runtime** (ToolSearch → tool description). Input shape may evolve; don't hard-code assumptions from this doc.
- New files are created in the user's **team drafts**; writing into an existing file requires edit permission on that file.
- The tool respects the user's Figma **seat type** — if calls fail with permission errors, the user's seat/plan is the first suspect. Report the error verbatim and stop; don't retry-loop.
- Setup guides (user runs these, not you):
  - Claude Code: https://help.figma.com/hc/en-us/articles/39888612464151
  - General MCP guide: https://help.figma.com/hc/en-us/articles/32132100833559
  - Tool docs: https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/

## Fallback path: SVG export (no MCP needed)

If the Figma MCP isn't connected, produce one `.svg` per screen. Figma imports SVG natively (drag-drop) as editable vectors.

**Method:** hand-translate each screen's HTML geometry into SVG — walk the rendered layout top-down, emitting one shape per box (position/size from the CSS values you wrote). Charts become explicit `<rect>`/`<path>`/`<circle>` marks. Do NOT rasterize the screen and do NOT wrap HTML in `foreignObject`.

Rules for editable-quality SVG:
- Exact artboard matching the design width: mobile `viewBox="0 0 390 844"`, desktop e.g. `viewBox="0 0 1440 1024"`.
- **Text as `<text>` elements** with `font-family`/`font-size`/`fill` — NEVER outline text to paths (designer must be able to retype it). Note: the font must be available in Figma for text to import as text.
- Group each logical block with `<g id="header">`, `<g id="stat-card">` — ids become layer names.
- Shapes as `<rect>`/`<circle>`/`<path>` with exact fills, radii (`rx`), and strokes.
- No `<foreignObject>`, no embedded HTML — Figma ignores it.
- Gradients via `<linearGradient>`; shadows via `filter` are hit-or-miss in Figma — prefer flat fills and let the designer add effects.

Limitations to tell the user: SVG import gives editable vectors + text, but no Auto-layout (static positions) — good starting point, less structured than the MCP path.

## Choosing

| Situation | Path |
|---|---|
| Figma MCP connected & authed | `generate_figma_design` (Auto-layout, best) |
| MCP missing/unauthorized | SVG fallback now + tell user how to connect MCP for next time |
| User explicitly wants files only | SVG fallback |
