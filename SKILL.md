---
name: ug-hd-taos-skill
description: Use when the user attaches design/mockup/UX-UI/screenshot images and wants them turned into an editable Figma design (professional SVG), a clickable HTML prototype, or deployed online via Lakebed. Triggers - design to Figma, mockup to SVG, image to Figma, แปลงภาพเป็น Figma, ทำเป็น SVG, ทำเป็น prototype, แปลงภาพเป็นเว็บ, deploy design.
---

# Design image → professional Figma-ready SVG (or clickable prototype)

Turn design images into pixel-faithful screens with two outputs:
- **Mode A (default): one professional SVG per screen** that a UX/UI designer drags straight into Figma and edits — token-based colors, 8pt grid, named layers, shared components identical across screens. Rules in `references/figma-handoff.md`.
- **Mode B (only when the user asks for a live URL / deploy):** clickable HTML prototype deployed as a Lakebed capsule. Rules in `references/lakebed-capsule.md`.

Nothing is hard-coded to a constant size: every dimension derives from the **target width the user chooses** (step 3).

## Pipeline

### 1. Ingest the images
Read every attached image. Establish: how many screens, which is which (list them), and **target device** — infer mobile vs desktop from aspect ratio/UI idioms.

**Normalize retina scale first:** design exports are often 2× (macOS screenshots especially). If the image is ~2× a standard device width (e.g. 2880 ≈ 2×1440, 780 ≈ 2×390), halve every measured px. All later steps work in CSS px; note the **measured design width** (e.g. mobile 390, desktop 1440).

### 2. Analyze each screen — and build the token table
- **Layout** — structure (header/sidebar/grid/stack), alignment, spacing.
- **Components** — buttons, inputs, cards, lists, nav, tabs, modals, charts/data-viz (redraw charts as vector shapes, not images).
- **Shared components** — identify regions that repeat across screens (sidebar, top nav, footer, tab bar). These MUST end up byte-identical in every screen (step 4).
- **Color tokens** — collect every distinct color into ONE palette table with semantic names (`primary`, `primary-hover`, `surface`, `surface-alt`, `text-primary`, `text-muted`, `border`, `success`, `danger` …). Every fill in every screen must come from this table — no ad-hoc hexes. This is what lets the designer convert colors to **Figma Variables** in one pass.
- **Text** — read every visible label/heading verbatim; note font family, and the type scale (sizes/weights used).

### 3. Ask the user — width is ALWAYS asked
**Mandatory question (never skip):** "อยากได้ Output กว้างเท่าไร?" — offer the measured design width as the suggested default (e.g. "วัดจากภาพได้ 1440px — เอาตามนี้ไหม หรือระบุค่าอื่น เช่น 1920 / 1280 / 375?"). All coordinates in step 4 are computed against the chosen width (scale factor = chosen ÷ measured).

**Everything else stays default-first:** infer what you reasonably can, state assumptions in one short vetoable list. Ask only when genuinely blocked: ambiguous colors, unrecognizable font, unreadable text, uninferable navigation, real photos vs placeholders.

Format: ONE message, numbered questions, each with your suggested default. Write in the user's language.

### 4. Build the Figma handoff set
Create a work folder under cwd (kebab-case from the design's purpose) with the standard handoff layout:

```
<project>/
  tokens.svg          # palette + type scale sheet
  components.svg      # UI-kit sheet: every reusable component, labeled
  screens/
    01-login.svg      # numbered in flow order
    02-dashboard.svg
```

Follow `references/figma-handoff.md` exactly — the short version:

- **Artboard = chosen width:** `viewBox="0 0 <W> <H>"` with `width`/`height` attributes set to the same values so it drops into Figma at 1:1.
- **Scale everything** from measured px by the step-3 factor (uniform zoom — a "scaled 1440" layout, not a re-flowed one; that's the deliberate design). Then snap **sizes and gaps** to the 8pt grid (multiples of 8; 4 for fine detail) — positions come from the running sum of snapped sizes/gaps, NOT independently snapped, so boxes never drift apart or overlap. Text baselines and icon offsets inside a component are free. **Font sizes:** scale by the factor, round to the nearest whole px, and fix the result once in the type scale — every screen uses those exact sizes. Artboard height = measured height × factor, snapped.
- **Figma-convention layer names:** every logical block is a group whose id reads like a Figma layer — **PascalCase, English, slash-namespaced**: `<g id="Sidebar">`, `<g id="Card/Revenue">`, `<g id="Button/Primary">`, `<g id="Icon/Search">`. Shallow, ordered top-to-bottom. (Internal `<defs>` ids stay kebab `cmp-*` and are always wrapped: `<g id="Sidebar"><use href="#cmp-sidebar"/></g>` so the designer sees the clean name.)
- **Shared components once:** author each repeating region (sidebar, nav) as a single `<defs><g id="cmp-sidebar">…</g></defs>` block and place with `<use href="#cmp-sidebar"/>`; paste the SAME defs block verbatim into every screen file. Identical by construction — never redraw it per screen. **Canonical state = every item inactive**; ANY per-screen difference (active menu item, page title in a shared nav) is a named overlay group on top of the `<use>` — see the reference for overlay rules.
- **Repeating elements inside a screen (table rows, list items, card grids):** author the FIRST item fully, then clone its geometry verbatim per item (y offset = item height × n) changing only the content — identical heights and column positions, so the designer can turn one row into a Figma component. Tables follow the **column-grid + text-fitting rules** in the reference — text must never cross into the next column.
- **Colors only from the token table.** Same role = same hex everywhere, so the designer can bulk-convert via Selection Colors → Variables.
- **Text stays text:** `<text>` elements with real font-family/size/weight — never outline to paths.
- **`tokens.svg`:** one swatch row per token (rect + name + hex) plus the type scale with Figma-style names (`Heading/H1 · 24/32 · SemiBold`) — the designer's one-glance reference for creating Figma Variables/styles.
- **`components.svg`:** the UI-kit sheet — one labeled specimen of every shared component, repeating template (Table/Row, Card), button/form control per visible state, and an icon grid. Same markup as the screens (see reference). This is where the designer componentizes once.

### 5. Verify against the original — capped loop
- **With preview/screenshot tools:** serve the folder with a static server (`npx -y serve .`), **resize the viewport to the chosen width × artboard height** before screenshotting (default presets crop a 1920 artboard), compare each screen to the source image. **Cap: 3 passes total.** Fix only clearly-wrong things (color, misalignment, missing element); re-screenshot only changed screens. Two classes of delta are EXPECTED — do not "fix" them back: 8pt-grid snap offsets (≤4px) and text metrics from a locally-missing font (flag the font instead). Stop early when visually faithful — don't chase sub-pixels.
- **Without screenshot tools or if screenshots time out:** ONE code-vs-image audit — re-read each SVG against the original, checking token usage, coordinates, text verbatim. Don't retry-loop.

ponytail: 3-pass cap is a deliberate token ceiling; raise only if the user asks for tighter fidelity.

### 6. Deliver
- **Default:** the handoff folder + assumptions list + a short **designer guide in the user's language** (the 5-step import checklist template is in the reference): install listed fonts → drag SVGs into Figma → Variables via Selection Colors → componentize from `components.svg` → text styles from `tokens.svg`.
- **Bonus, only if the Figma MCP is connected and authed** (skip silently otherwise): also push screens via `generate_figma_design` for Auto-layout frames — see the MCP section of `references/figma-handoff.md`.

## Mode B — Lakebed deploy (only when asked)
When the user wants a shareable live URL: build one plain HTML+CSS file per screen from the same analysis (same tokens as CSS variables, same chosen width, `<a href>` navigation), then port to a Preact capsule and deploy per `references/lakebed-capsule.md`. Key rules: `npx -y lakebed new` in a new folder; cd into the app folder for every command; CLI fails → scaffold by hand from the reference; verify the deployed URL serves before reporting; auth/domain prompts are interactive → hand those commands to the user.

## Common mistakes
| Mistake | Fix |
|---|---|
| Hard-coding 390/1440 instead of asking | Width is a mandatory step-3 question, measured width offered as default |
| Ad-hoc hex colors per element | Single token table; every fill references it |
| Redrawing the sidebar per screen (drifting sizes) | One `<defs>` component block, `<use>`, pasted verbatim into each file |
| Outlining text to paths | `<text>` always — designer must be able to retype |
| Off-grid magic numbers (13px gaps) | Snap to 8pt grid (4 for fine detail) |
| Rasterizing or `foreignObject` in SVG | Real shapes only — Figma ignores/flattens the rest |
| Table text overflowing/overlapping columns | Column grid defined once; budget text width, truncate with … |
| Each table row drawn freehand (drifting geometry) | Clone row 1's geometry verbatim, change only content |
| Choosing Lakebed mode uninvited | SVG/Figma is the default; deploy only when the user asks for a URL |

## Notes
- Figma MCP not connected is NOT an error state — SVG delivery is the standard path, MCP is a bonus.
- Wants both outputs? Deliver SVGs first, then Mode B reuses the analysis and tokens.
