# Figma-ready SVG standard (primary output)

The deliverable is SVG a professional UX/UI designer would accept as a starting file: drag into Figma, everything named, colors tokenized, shared components identical, resizable without surgery.

## Document setup — dynamic, not constant

- Artboard from the user's **chosen width** (mandatory step-3 question): `viewBox="0 0 <W> <H>"` AND `width="<W>" height="<H>"` attributes — Figma then imports at exactly 1:1.
- Scale = measured px × (chosen ÷ measured width), uniform zoom. **Snap sizes and gaps** to the 8pt grid (multiples of 8; 4 for fine detail); **positions are the running sum of snapped sizes/gaps** — never snap x/y independently, that makes adjacent boxes gap or overlap. Text baselines and offsets inside a component are free. Whole numbers only. Font sizes: × factor, round to whole px, fixed once in the type scale. Artboard height = measured height × factor, snapped.
- Because everything lives under one `viewBox` in named groups with no clip-path tricks, the imported frame scales proportionally when the designer resizes it.

## Layer naming (what the designer sees)

- Every logical block: `<g id="sidebar">`, `<g id="card-revenue">`, `<g id="btn-primary">` — kebab-case, semantic, unique per file. Figma turns ids into layer names.
- Shallow hierarchy ordered top-to-bottom as in the design. No anonymous wrapper groups.

## Color tokens → Figma Variables

- ONE palette table (built in pipeline step 2): semantic names → hex (`primary #2563EB`, `surface #FFFFFF`, `text-primary #0F172A`, …). **Sample from flat interior regions only** — never anti-aliased edges — and merge near-identical readings (±2–3 per channel) into one canonical token. A real UI palette is ~8–20 tokens, not hundreds.
- Every `fill`/`stroke` in every screen uses a hex FROM the table — never a one-off. Same role = same hex byte-for-byte.
- Why: Figma's **Selection Colors** lists each distinct hex once across the selection; consistent hexes let the designer convert the whole file to Figma Variables in one pass. (SVG can't carry actual Figma Variables — consistency is the bridge. CSS custom properties in `<style>` are NOT reliably honored by Figma's importer; use literal hexes.)
- Deliver `tokens.svg`: one row per token — `<rect>` swatch + `<text>` name + hex — plus the type scale (one `<text>` sample per size/weight). This is the sheet the designer builds Variables/text styles from.

## Shared components — identical by construction

For any region repeating across screens (sidebar, top nav, tab bar, footer):

```svg
<defs>
  <g id="cmp-sidebar"><!-- drawn ONCE: 240×900, all token colors --></g>
</defs>
<use href="#cmp-sidebar" x="0" y="0"/>
```

- Author the component ONE time — **canonical state: every item inactive** (you'll have to synthesize the inactive look of items only ever shown active; copy the styling of their siblings). Paste the same `<defs>` block **verbatim** into every screen file, place with `<use>`.
- Never redraw or "adjust" it per screen — identical size/spacing everywhere is the requirement. **Any per-screen difference is an overlay**, not a fork: active menu item, page title/breadcrumb in a shared nav, badge counts. Overlay = a named group on top of the `<use>` containing an **opaque** highlight rect plus a restyled copy of the label/icon (`<g id="sidebar-active-products">`); the opaque rect hides the inactive copy underneath, so Figma shows one visible label. If a strip of the region is entirely per-screen (e.g. breadcrumb bar), exclude that strip from the component instead.
- Figma expands `<use>` into a group on import, so every screen carries a same-named, same-sized copy the designer can turn into a real Figma component.

## Shapes & text

- Text as `<text>` with `font-family` (real name, e.g. `Inter`), `font-size`, `font-weight`, `fill` from tokens — NEVER outline text to paths (designer must retype). Font must exist in Figma for text to import as text; note any font the user must install.
- Shapes as `<rect>` (radius via `rx`), `<circle>`, `<path>`. Charts = explicit `<rect>`/`<path>`/`<circle>` marks, hand-translated from the design — never an embedded image.
- Icons: inline paths inside a named group (`<g id="icon-search">`), square boxes = measured size × factor, snapped to the grid (e.g. a 24px icon at 1.333× → 32×32).
- Gradients via `<linearGradient>` in defs. Shadows via `filter` are hit-or-miss in Figma — prefer flat fills and note "add effect in Figma" instead.
- Banned: raster embeds of the screen, `<foreignObject>`, external `<image>` links for UI chrome (user-provided real photos may use `<image href>`).

## Production method

Walk each screen's analyzed layout top-down, emitting one shape per box using the scaled, grid-snapped values. Don't rasterize; don't approximate with one giant path.

## Optional bonus: Figma MCP `generate_figma_design`

Only when the Figma MCP server is connected AND authed (skip silently otherwise — SVG is the standard path):
- The tool sends HTML/UI into Figma as editable Auto-layout layers. Read its parameter schema at runtime (ToolSearch "figma"); if the server is flagged unauthenticated or the tool is absent, don't burn a failing call.
- New files land in the user's team drafts; existing files need edit permission; respects seat type — on permission errors report verbatim and stop, don't retry-loop.
- Setup guides (user runs these): Claude Code https://help.figma.com/hc/en-us/articles/39888612464151 · general https://help.figma.com/hc/en-us/articles/32132100833559 · tool docs https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/
