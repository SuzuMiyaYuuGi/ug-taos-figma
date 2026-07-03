# ug-taos-figma — Design image → professional Figma-ready SVG / clickable prototype

An [Agent Skill](https://agentskills.io) for **Claude Code** and **Codex CLI**: attach design/mockup/UX-UI images and get back **professional SVG screens that drag straight into Figma** — named layers, token-based colors ready to become Figma Variables, shared components identical across screens, everything sized to the width YOU choose. Optionally deploys a **clickable prototype on [Lakebed](https://docs.lakebed.dev/)** instead.

## What it does

```
design PNGs ──► analyze (retina-normalize, color-token table, shared components)
            ──► ask: target output width (always) + only what can't be inferred
            ──► one professional SVG per screen + tokens.svg palette sheet
                • 8pt grid, scaled to your chosen width (no hard-coded sizes)
                • <g id> named layers, <text> stays editable text
                • sidebar/nav authored once (<defs>+<use>) → identical on every screen
                • consistent hexes → Selection Colors → Figma Variables in one pass
            ──► verify vs. original (capped screenshot loop)
            ──► deliver SVGs (drag into Figma)
                └─ bonus: Figma MCP connected? also pushes via generate_figma_design
            ──► Mode B (on request): deploy as a Lakebed capsule → public URL
```

## Requirements

- **Mode A (SVG → Figma):** nothing — SVG delivery works out of the box. The official [Figma MCP server](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server) ([Claude Code setup](https://help.figma.com/hc/en-us/articles/39888612464151-Claude-Code-and-Figma-Set-up-the-MCP-server)) is an optional bonus for Auto-layout push.
- **Mode B (Lakebed):** Node.js (`npx lakebed` is fetched on demand).

## Install

The folder name becomes the slash command, so install into a folder named `ug-hd-taos-skill`.

### Claude Code

```bash
git clone https://github.com/SuzuMiyaYuuGi/ug-taos-figma.git ~/.claude/skills/ug-hd-taos-skill
```

Windows (PowerShell):

```powershell
git clone https://github.com/SuzuMiyaYuuGi/ug-taos-figma.git "$env:USERPROFILE\.claude\skills\ug-hd-taos-skill"
```

### Codex CLI

```bash
git clone https://github.com/SuzuMiyaYuuGi/ug-taos-figma.git ~/.agents/skills/ug-hd-taos-skill
```

Windows (PowerShell):

```powershell
git clone https://github.com/SuzuMiyaYuuGi/ug-taos-figma.git "$env:USERPROFILE\.agents\skills\ug-hd-taos-skill"
```

### Both tools, one copy (Windows)

Clone once, junction everywhere:

```powershell
git clone https://github.com/SuzuMiyaYuuGi/ug-taos-figma.git C:\src\ug-taos-figma
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\ug-hd-taos-skill" -Target C:\src\ug-taos-figma
New-Item -ItemType Junction -Path "$env:USERPROFILE\.agents\skills\ug-hd-taos-skill" -Target C:\src\ug-taos-figma
```

(macOS/Linux: same idea with `ln -s`.)

Restart Claude Code / Codex after installing so the skill list refreshes.

## Use

Attach your design image(s) — one image per screen — and either:

- type `/ug-hd-taos-skill`, or
- just ask naturally: *"แปลงเป็น Figma ให้หน่อย"*, *"turn this mockup into Figma"*, *"ทำเป็น prototype คลิกได้"* — the skill auto-triggers on design-to-Figma/HTML requests.

Want a shareable live URL instead of (or after) Figma? Ask for it — that switches to Lakebed deploy mode.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The skill: shared pipeline + Figma mode + Lakebed mode |
| `references/figma-handoff.md` | Figma-ready SVG standard (grid, tokens, shared components) + optional MCP push |
| `references/lakebed-capsule.md` | Lakebed capsule structure, CLI, HTML→capsule porting rules |

## License

MIT
