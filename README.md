# ug-taos-figma — Design image → editable Figma / clickable prototype

An [Agent Skill](https://agentskills.io) for **Claude Code** and **Codex CLI**: attach design/mockup/UX-UI images and get back an **editable Figma design** (real Auto-layout frames + editable text layers — not a flat image), or optionally a **clickable prototype deployed on [Lakebed](https://docs.lakebed.dev/)**.

## What it does

```
design PNGs ──► analyze (colors/px/text, retina-normalize)
            ──► ask ONLY what can't be inferred (defaults offered)
            ──► one plain HTML+CSS file per screen (pixel-faithful, clickable locally)
            ──► verify vs. original (capped screenshot loop)
            ──► Mode A (default): send to Figma via generate_figma_design → editable layers
                └─ Figma MCP not connected? SVG fallback (drag into Figma, text stays editable)
            ──► Mode B (on request): deploy as a Lakebed capsule → public URL
```

## Requirements

- **Mode A (Figma):** the official [Figma MCP server](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server) connected to your agent ([Claude Code setup](https://help.figma.com/hc/en-us/articles/39888612464151-Claude-Code-and-Figma-Set-up-the-MCP-server)) + a Figma account. Without it, the skill still delivers importable SVGs.
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
| `references/figma-handoff.md` | `generate_figma_design` usage + editable-SVG fallback rules |
| `references/lakebed-capsule.md` | Lakebed capsule structure, CLI, HTML→capsule porting rules |

## License

MIT
