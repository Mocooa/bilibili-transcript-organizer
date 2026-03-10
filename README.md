# bilibili-transcript-organizer

[中文说明](./README.zh-CN.md)

Turn Bilibili videos, playlists, multi-part series, favorites, or subtitle files into structured knowledge outputs.

This repository contains a Claude Code skill focused on **knowledge organization**, not content rewriting. It keeps source attribution, supports multi-video synthesis, and can output Markdown notes plus Obsidian `.canvas` maps.

<p align="center">
  <img src="./assets/preview-notes.svg" alt="Structured notes preview" width="48%" />
  <img src="./assets/preview-canvas.svg" alt="Canvas map preview" width="48%" />
</p>

## Preview

Example outputs live in [`examples/sample-notes.md`](./examples/sample-notes.md) and [`examples/sample-topic-map.canvas`](./examples/sample-topic-map.canvas).

`notes` preview:

```md
## Core Takeaways

- The series frames the issue as a structural problem, not an isolated event.
- Energy, diplomacy, and domestic politics interact across multiple episodes.
- Later videos refine earlier claims instead of simply repeating them.

## Topic Threads

### Historical context
Explain the long-term background needed to interpret the current event.

### Cross-video evolution
Track which claims stay stable and which ones change as the series develops.
```

`.canvas` preview:

```json
{
  "nodes": [
    { "id": "topic", "type": "text", "text": "Core Topic" },
    { "id": "factor-a", "type": "text", "text": "Historical Context" },
    { "id": "factor-b", "type": "text", "text": "Policy Impact" }
  ],
  "edges": [
    { "id": "e1", "fromNode": "topic", "toNode": "factor-a" },
    { "id": "e2", "fromNode": "topic", "toNode": "factor-b" }
  ]
}
```

## Real Workflow

This is the intended end-to-end flow for a typical multi-video series:

### Input

```text
Organize this Bilibili series into full notes first, then recommend glossary or visual output if useful:
https://space.bilibili.com/<user-id>/lists/<season-id>?type=season
```

### What the skill does

1. Uses `bili` CLI to fetch video metadata and subtitles
2. Builds one shared knowledge structure across the full series
3. Generates a base `notes` document first
4. Recommends extra outputs such as `glossary`, `timeline`, or `visual` when the material supports them

### Typical output set

- One composite `notes` document for the series
- Source references preserved as BV links, part markers, or timestamps
- Optional `visual` output as an Obsidian `.canvas` topic map
- Optional `glossary` when the material is concept-heavy

### Example result shape

- [`examples/sample-notes.md`](./examples/sample-notes.md)
- [`examples/sample-topic-map.canvas`](./examples/sample-topic-map.canvas)

## Why This Exists

Bilibili has a lot of valuable long-form analysis, lectures, and issue-based series. The raw viewing experience is good for consumption, but weak for:

- long-term note-taking
- cross-video synthesis
- concept extraction
- timeline building
- reusable knowledge storage

This skill is designed to solve that gap.

## Features

- Always generates a base `notes` output first
- Supports additional modes: `brief`, `glossary`, `timeline`, `compare`, `index`, `qa`, `cards`, `visual`
- Uses `bili` CLI for Bilibili resource fetching
- Supports fallback input from local subtitle files
- Handles single videos, multi-part videos, playlists, favorites, and series
- Supports one composite note or multiple themed notes for multi-video input
- Includes `.canvas` visual output for Obsidian
- Explicitly avoids rewriting, style imitation, and laundering-style output

## Best For

- analysis videos
- history / politics / economics / technology explainers
- issue-based series
- lecture-style content
- knowledge management workflows in Obsidian or Markdown

## Not For

- entertainment clips
- pure subtitle downloading
- marketing rewrite generation
- imitation of the creator's writing style

## Quick Start

### 1. Preferred install: `skills.sh`

`skills.sh` is the recommended install path for general users:

```bash
npx skills add Mocooa/bilibili-transcript-organizer
```

If you want to install from a full GitHub URL instead:

```bash
npx skills add https://github.com/Mocooa/bilibili-transcript-organizer
```

### 2. Local development or manual install

If you want to inspect or modify the skill locally:

```bash
git clone https://github.com/Mocooa/bilibili-transcript-organizer.git
cd bilibili-transcript-organizer
claude skill install .
```

If your environment does not support that command, copy this directory into your local skill directory manually.

### 3. Install `bili` CLI if you want direct Bilibili fetching

Check first:

```bash
command -v bili
```

If not installed, use one of:

```bash
uv tool install bilibili-cli
# or
pipx install bilibili-cli
# or
python3 -m pip install --user bilibili-cli
```

Verify:

```bash
bili --help
```

If you do not want to install the CLI, you can still use local `.srt`, `.txt`, or `.md` subtitle files as input.

## Output Modes

`notes` is always the base output. Other modes are additional views on top of it.

| Mode | Purpose |
|------|---------|
| `notes` | base structured note |
| `brief` | one-page summary |
| `glossary` | term sheet |
| `timeline` | event or argument sequence |
| `compare` | multi-video / multi-view comparison |
| `index` | series navigation page |
| `qa` | question-answer index |
| `cards` | atomic knowledge cards |
| `visual` | Obsidian `.canvas` map |

`visual` v1 only targets Obsidian `.canvas`, not Excalidraw.

## Example Prompts

```text
Organize this Bilibili video: BV1ABcsztEcY
```

```text
Compare these 3 Bilibili videos and focus on where their arguments diverge
```

```text
Turn this playlist into a glossary and list key terms with where they first appear
```

```text
Turn this series into a visual mode output and build a canvas topic map
```

```text
I have exported subtitle files. Turn them into atomic knowledge cards
```

## Multi-Video Behavior

For multi-video input, the skill does not blindly split by count.

It first checks theme complexity, then chooses between:

- one composite `notes`
- multiple themed `notes`

If both are reasonable, it should ask the user which structure they prefer.

There is no hard limit on number of documents or note length. Output structure should follow content structure and user preference.

## Repository Structure

```text
bilibili-transcript-organizer/
├── SKILL.md
├── README.md
├── README.zh-CN.md
├── LICENSE
├── assets/
│   ├── preview-canvas.svg
│   └── preview-notes.svg
├── examples/
│   ├── sample-notes.md
│   └── sample-topic-map.canvas
└── references/
    ├── modes.md
    ├── templates.md
    └── canvas.md
```

## Files

- `SKILL.md`: main skill instructions
- `README.zh-CN.md`: Chinese README
- `assets/preview-notes.svg`: README visual preview for structured notes
- `assets/preview-canvas.svg`: README visual preview for `.canvas` maps
- `examples/sample-notes.md`: public sample of a structured note output
- `examples/sample-topic-map.canvas`: public sample of a visual output
- `references/modes.md`: mode semantics, fallback rules, delivery pattern
- `references/templates.md`: Markdown output templates
- `references/canvas.md`: `.canvas` output rules

## FAQ

**What is the recommended install method?**  
Use `skills.sh` via `npx skills add Mocooa/bilibili-transcript-organizer`. Clone/manual install is mainly for local development or customization.

**Do I need Obsidian?**  
No. Markdown outputs work anywhere. `.canvas` is mainly for Obsidian Canvas.

**What if `bili: command not found` appears?**  
Install `bilibili-cli`, or switch to local subtitle-file input.

**Will this rewrite videos into "original articles"?**  
No. This skill is for knowledge organization and indexing, not rewriting.

**What happens if I request `compare` for a single video?**  
The skill should still generate `notes` first, then downgrade gracefully and recommend more suitable modes such as `timeline`, `glossary`, or `visual`.

**How does it decide one note vs multiple notes for a series?**  
Based on theme complexity and, when needed, explicit user preference.

## License

MIT
