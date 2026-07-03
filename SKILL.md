---
name: ug-hd-taos-skill
description: Use when the user attaches design/mockup/UX-UI/screenshot images and wants them turned into an editable Figma design, clickable HTML prototype, or deployed online via Lakebed. Triggers - design to Figma, mockup to HTML, image to Figma, แปลงภาพเป็น Figma, ทำเป็น prototype, แปลงภาพเป็นเว็บ, deploy design.
---

# Design image → editable Figma design (or clickable prototype)

Turn design images into pixel-faithful screens, then hand off to one of two outputs:
- **Figma mode (default):** editable Figma layers so a UX/UI designer can revise the design.
- **Lakebed mode (only when the user asks for a live URL / deploy):** clickable prototype deployed as a Lakebed capsule.

Both modes share the same pipeline (steps 1–5). The intermediate artifact is always **one self-contained plain HTML file per screen** — it renders in any browser and is what gets sent to Figma or ported to Lakebed.

## Shared pipeline

### 1. Ingest the images
Read every attached image. Establish: how many screens, which is which (list them), and **target device** — infer mobile vs desktop from aspect ratio/UI idioms.

**Normalize retina scale first:** design exports are often 2× (macOS screenshots especially). If the image is ~2× a standard device width (e.g. 2880 ≈ 2×1440, 780 ≈ 2×390), halve every measured px. All later steps work in CSS px; note the chosen **design width** (e.g. mobile 390, desktop 1440).

### 2. Analyze each screen
- **Layout** — structure (header/sidebar/grid/stack), alignment, spacing.
- **Components** — buttons, inputs, cards, lists, nav, tabs, modals, charts/data-viz (redraw charts as inline SVG shapes, not images).
- **Exact values** — hex colors, px sizes (font, padding, radius, width/height), font family.
- **Text** — read every visible label/heading verbatim.
- **Assets** — icons and images (asset rule in step 4).

### 3. Ask ONLY what's genuinely unclear — don't block on the inferable
**Default-first rule:** if you can infer a reasonable answer (e.g. Login → Dashboard is the obvious flow), proceed with it and state your assumptions in one short list the user can veto. Ask only when genuinely blocked:
- ambiguous colors, unrecognizable font, unreadable/cut-off text
- navigation wiring you truly can't infer
- real photos vs placeholders

Format: ONE message, numbered questions, each with your suggested default so the user can just say "ตามนั้น". Write in the user's language. If nothing is unclear, skip this step and say so.

### 4. Build each screen as a plain HTML file (fidelity rules)
Create a work folder under the current working directory (kebab-case from the design's purpose, e.g. `fitness-app-design/`), one `<screen>.html` per screen (entry screen = `index.html`).

- **Plain HTML + a `<style>` block** (real CSS — flexbox/grid). No frameworks, no build step. Use exact values from the image: hex colors, px sizes, radii, spacing.
- **Semantic flat structure, well-named:** use flexbox/grid for layout (Figma maps them to Auto-layout) and give major blocks clear names (`class="header"`, `class="stat-card"`) — these become the designer's layer names. Avoid deep pointless nesting.
- **Fonts:** identify the typeface and load it via a Google Fonts `<link>` when available; otherwise closest system stack, and flag the gap.
- **Fixed canvas at the design width (both device types):** mobile → centered phone frame (`width: 390px; margin: 0 auto; min-height: 100vh`) on a neutral background; desktop → container fixed to the design width from step 1 (`width: 1440px` or as measured), not fluid — fidelity beats responsiveness for a design handoff.
- **Icons → inline SVG.** Real photos → user-provided URLs. Placeholders → simple inline SVG shape or `https://placehold.co/WxH`.
- **Wire navigation** with plain `<a href="dashboard.html">` links per the flow from step 3 — the folder is itself a clickable prototype when opened in a browser.

### 5. Verify against the original — capped loop
- **With preview/screenshot tools:** preview tools need a server, not `file://` — serve the folder with any static server (e.g. `npx -y serve .` in the work folder) and **set the screenshot viewport to the design width** so the comparison is apples-to-apples. Screenshot each screen, compare to the source image. **Cap: 3 passes total.** Fix only clearly-wrong things (color, misalignment, missing element); re-screenshot only changed screens. Stop early when visually faithful — don't chase sub-pixels.
- **Without screenshot tools or if screenshots time out:** do ONE code-vs-image audit instead — re-read each screen's HTML/CSS against the original image, checking colors/sizes/text verbatim. Don't retry-loop screenshots.

ponytail: 3-pass cap is a deliberate token ceiling; raise only if the user asks for tighter fidelity.

## Output mode A — Figma (default)

Goal: every screen lands in Figma as **editable layers** (Auto-layout frames, editable text, image fills) — not a flat image. Read `references/figma-handoff.md` before this step.

1. Load the Figma MCP tools (ToolSearch for "figma"). The key tool is **`generate_figma_design`** — read its actual parameter schema at runtime; input shape may change. If the server is already flagged unauthenticated in this session, or the tool is absent from whatever "figma" server you find, go straight to the fallback — don't burn a failing call.
2. Send each screen's HTML. New files land in the user's Figma team drafts; sending into an existing file needs edit permission there.
3. **If the Figma MCP is not connected/authenticated:** don't fake it — tell the user to connect it (claude.ai connector settings, or `claude mcp` in an interactive session, per the setup guide linked in the reference), and meanwhile deliver the **SVG fallback**: export each screen as a faithful `.svg` (real `<text>` elements, named groups) that the designer can drag straight into Figma as editable vectors.
4. Report: Figma file link (or SVG file paths), plus the assumptions list from step 3 of the pipeline.

## Output mode B — Lakebed deploy (only when asked)

When the user wants a shareable live URL: port the HTML screens into a Lakebed capsule (Preact + Tailwind arbitrary values, one route per screen) and deploy. Full structure, CLI commands, failure fallbacks, and porting rules are in `references/lakebed-capsule.md`. Key rules: scaffold with `npx -y lakebed new` in a new folder under cwd; cd into the app folder for every command; CLI fails → scaffold by hand from the reference; verify the deployed URL serves before reporting it; auth/domain prompts are interactive → hand those commands to the user.

## Common mistakes
| Mistake | Fix |
|---|---|
| Sending Figma a flat screenshot/image | Use `generate_figma_design` with the HTML so layers stay editable |
| Deep div soup / unnamed blocks | Flat semantic structure with meaningful class names → clean layer names |
| Mobile design rendered edge-to-edge on desktop | 390px centered phone frame |
| Blocking on questions the image already answers | Default-first: assume, list assumptions, let user veto |
| Retry-looping a failing MCP/CLI/screenshot | One fallback (SVG / manual scaffold / code audit), then move on |
| Choosing Lakebed mode uninvited | Figma is the default; deploy only when the user asks for a URL |

## Notes
- The HTML work folder is a deliverable in itself — user can open `index.html` locally and click through.
- Wants both? Do Figma mode first, then Lakebed mode reuses the same screens.
