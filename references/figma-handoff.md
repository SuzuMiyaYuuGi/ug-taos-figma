# Figma-ready SVG standard (primary output)

The deliverable is SVG a professional UX/UI designer would accept as a starting file: drag into Figma, everything named, colors tokenized, shared components identical, resizable without surgery.

## Document setup — dynamic, not constant

- Artboard from the user's **chosen width** (mandatory step-3 question): `viewBox="0 0 <W> <H>"` AND `width="<W>" height="<H>"` attributes — Figma then imports at exactly 1:1.
- Scale = measured px × (chosen ÷ measured width), uniform zoom. **Snap sizes and gaps** to the 8pt grid (multiples of 8; 4 for fine detail); **positions are the running sum of snapped sizes/gaps** — never snap x/y independently, that makes adjacent boxes gap or overlap. Text baselines and offsets inside a component are free. Whole numbers only. Font sizes: × factor, round to whole px, fixed once in the type scale. Artboard height = measured height × factor, snapped.
- Because everything lives under one `viewBox` in named groups with no clip-path tricks, the imported frame scales proportionally when the designer resizes it.

## Layer naming (what the designer sees)

Figma turns group ids into layer names, so ids follow **Figma naming conventions**, not code conventions:
- **PascalCase, English** (even when UI text is Thai): `<g id="Sidebar">`, `<g id="TopNav">`.
- **Slash namespaces** for parts/variants — Figma nests these visually: `Button/Primary`, `Button/Secondary`, `Icon/Search`, `Table/HeaderRow`, `Table/Row-01`, `Card/Revenue`.
- Referenced `<defs>` ids stay kebab `cmp-*` (safe in `href="#cmp-sidebar"`); ALWAYS wrap the `<use>` in a named group so the ugly internal id never becomes a visible layer: `<g id="Sidebar"><use href="#cmp-sidebar"/></g>`.
- Shallow hierarchy ordered top-to-bottom as in the design. No anonymous wrapper groups — an unnamed `<g>` becomes a meaningless "Group" layer.

## Color tokens → Figma Variables

- ONE palette table (built in pipeline step 2): semantic names → hex (`primary #2563EB`, `surface #FFFFFF`, `text-primary #0F172A`, …). **Sample from flat interior regions only** — never anti-aliased edges — and merge near-identical readings (±2–3 per channel) into one canonical token. A real UI palette is ~8–20 tokens, not hundreds.
- Every `fill`/`stroke` in every screen uses a hex FROM the table — never a one-off. Same role = same hex byte-for-byte.
- Why: Figma's **Selection Colors** lists each distinct hex once across the selection; consistent hexes let the designer convert the whole file to Figma Variables in one pass. (SVG can't carry actual Figma Variables — consistency is the bridge. CSS custom properties in `<style>` are NOT reliably honored by Figma's importer; use literal hexes.)
- Deliver `tokens.svg`: one row per token — `<rect>` swatch + `<text>` name + hex — plus the type scale, each sample captioned with a Figma-style text-style name: `Heading/H1 · 24/32 · SemiBold`, `Body/Regular · 16/24`, `Caption · 12/16`. This is the sheet the designer builds Variables/text styles from.

## Components sheet (components.svg)

A one-page UI kit the designer componentizes once, then swaps into screens. Layout: neutral background, title, specimens on the 8pt grid with generous gaps.

- **One labeled specimen of every reusable piece:** each shared `cmp-*` component (Sidebar, TopNav), each repeating template (Table/Row, Card/Product, ListItem), each button/form control, plus an icon grid (every icon at output size).
- **Same markup as the screens** — the specimen IS the `<defs>` component placed once (`<g id="Sidebar"><use …/></g>`), not a redraw. Component sheet and screens can never drift apart.
- **States: only what the designs show.** A hover/disabled/active state visible anywhere in the source images gets a specimen (`Button/Primary/Hover`); never invent states — note "add states in Figma" instead.
- **Caption under each specimen** (name + WxH, e.g. `Sidebar · 240×900`) as small muted `<text>` OUTSIDE the component group, so captions don't import into the component itself.

## Designer guide (print at delivery, in the user's language)

1. Install fonts first: <list — or "none needed">, then drag `tokens.svg`, `components.svg`, and `screens/*` into Figma (fonts must be present for text to import as editable text).
2. Colors → Variables: select all on any screen → Selection Colors shows each token hex exactly once → create Variables named per `tokens.svg`.
3. Components: on the components page, right-click each specimen → Create component. Screen copies are same-named groups — swap as needed.
4. Text styles: create from the captioned samples in `tokens.svg`.
5. Layout intent: 8pt grid throughout; spacing/sizes are multiples of 8 (4 for fine detail).

## Shared components — identical by construction

For any region repeating across screens (sidebar, top nav, tab bar, footer):

```svg
<defs>
  <g id="cmp-sidebar"><!-- drawn ONCE: 240×900, all token colors --></g>
</defs>
<g id="Sidebar"><use href="#cmp-sidebar" x="0" y="0"/></g>
```

- Author the component ONE time — **canonical state: every item inactive** (you'll have to synthesize the inactive look of items only ever shown active; copy the styling of their siblings). Paste the same `<defs>` block **verbatim** into every screen file, place with `<use>`.
- Never redraw or "adjust" it per screen — identical size/spacing everywhere is the requirement. **Any per-screen difference is an overlay**, not a fork: active menu item, page title/breadcrumb in a shared nav, badge counts. Overlay = a named group on top of the `<use>` containing an **opaque** highlight rect plus a restyled copy of the label/icon (`<g id="Sidebar/Active-Products">`); the opaque rect hides the inactive copy underneath, so Figma shows one visible label. If a strip of the region is entirely per-screen (e.g. breadcrumb bar), exclude that strip from the component instead.
- Figma expands `<use>` into a group on import, so every screen carries a same-named, same-sized copy the designer can turn into a real Figma component.

## Tables & repeating lists — the column grid

SVG `<text>` neither wraps nor clips. If you don't budget text width yourself, long values run under the next column — the #1 way generated tables become unusable. Rules:

**1. Define the column grid ONCE per table** (before drawing anything):

```
columns: [
  { name: "product", x: 24,  width: 320, align: "start"  },
  { name: "status",  x: 360, width: 120, align: "start"  },
  { name: "price",   x: 496, width: 96,  align: "end"    },
]
rowHeight: 56   headerHeight: 48   cellPadX: 16
```

Every header cell and every row cell reads its x/width/alignment from this grid — never eyeball a cell individually. Column widths come from the design, snapped; they must sum (with gaps) to the table width.

**2. Text fitting — budget, then truncate.** Approximate text width ≈ 0.6 × font-size × character count. Budget per cell = column width − 2 × cellPadX. Over budget → truncate the STRING with `…` before writing the `<text>` element. Never rely on the next column's background to hide overflow, never squeeze with `textLength` (Figma ignores it, glyphs distort).

**3. Alignment via anchor, not spaces:** left → `text-anchor="start"` at `x + cellPadX`; right (numbers) → `text-anchor="end"` at `x + width − cellPadX`; center → `text-anchor="middle"` at `x + width/2`.

**4. One row = one group, geometry cloned:** structure as `<g id="Table/Products">` → `<g id="Table/HeaderRow">` + `<g id="Table/Row-01">`, `<g id="Table/Row-02">`… Author row 1 completely (cells, badge pills, divider line INSIDE the row group so the row moves as one unit); every other row copies that geometry verbatim at `y = headerHeight + rowHeight × n`, changing only content. Identical rows = designer selects row-01, makes it a Figma component, swaps text.

The same template-clone discipline applies to any repeating list: menu items, card grids, notification rows — first item is the template, the rest are geometry clones with new content.

## Shapes & text

- Text as `<text>` with `font-family` (real name, e.g. `Inter`), `font-size`, `font-weight`, `fill` from tokens — NEVER outline text to paths (designer must retype). Font must exist in Figma for text to import as text; note any font the user must install.
- Shapes as `<rect>` (radius via `rx`), `<circle>`, `<path>`. Charts = explicit `<rect>`/`<path>`/`<circle>` marks, hand-translated from the design — never an embedded image.
- Icons: inline paths inside a named group (`<g id="Icon/Search">`), square boxes = measured size × factor, snapped to the grid (e.g. a 24px icon at 1.333× → 32×32).
- Gradients via `<linearGradient>` in defs. Shadows via `filter` are hit-or-miss in Figma — prefer flat fills and note "add effect in Figma" instead.
- Banned: raster embeds of the screen, `<foreignObject>`, external `<image>` links for UI chrome (user-provided real photos may use `<image href>`).

## Production method

Walk each screen's analyzed layout top-down, emitting one shape per box using the scaled, grid-snapped values. Don't rasterize; don't approximate with one giant path.

## Optional bonus: Figma MCP `generate_figma_design`

Only when the Figma MCP server is connected AND authed (skip silently otherwise — SVG is the standard path):
- The tool sends HTML/UI into Figma as editable Auto-layout layers. Read its parameter schema at runtime (ToolSearch "figma"); if the server is flagged unauthenticated or the tool is absent, don't burn a failing call.
- New files land in the user's team drafts; existing files need edit permission; respects seat type — on permission errors report verbatim and stop, don't retry-loop.
- Setup guides (user runs these): Claude Code https://help.figma.com/hc/en-us/articles/39888612464151 · general https://help.figma.com/hc/en-us/articles/32132100833559 · tool docs https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/
