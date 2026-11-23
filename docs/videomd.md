# VIDEO.md Breakdown Package

Breaking down a video produces two outputs:

- **Human-readable report** (`report.html` / `report.pdf`): what the video is about, why it works, how it was made, and how to remake it.
- **Machine-readable breakdown package** (`VIDEO.md` + `video/`): the same findings structured in a fixed directory layout for AI agents and downstream remake tools.

Every skill has a `SKILL.md`; every video gets a `VIDEO.md`. VIDEO.md is an open format—its specification, design rationale, integration guides, and validator live at [videomd/videomd](https://github.com/videomd/videomd). This page documents what `rea-video` writes into the package.

## Layered Reading

1. **Card** (`VIDEO.md` frontmatter, a few hundred tokens): what the video is about, how it was made, and confidence.
2. **Overview & Map** (`VIDEO.md` body, ~1–2k tokens): verdict, narrative arc, remake summary, uncertainties, and a file map (`path | ~tokens | contents · when to read`).
3. **Timeline** (`video/timeline.md`): one line per shot. Keys with `~` are inferred; keys without `~` are directly observed or measured.
4. **Shot files** (`video/shots/<id>.md`) and frame crops: opened only when verifying a specific shot.

For a 3-minute talking-head video, reading up to the overview costs ~2k tokens, or ~4k tokens with the timeline—compared to ~190k tokens for the full `breakdown.json`.

## Package Contents

| File | Contents |
|---|---|
| `VIDEO.md` | Frontmatter with `made_mix` (duration-weighted production breakdown), `confidence`, `readers` (`tools` / `vlm` / `agent` / `human`), `source.sha256`; body with verdict, summary, narrative arc, remake overview, required user assets, uncertainties, and map |
| `video/timeline.md` | `story` narrative role · `seen` visuals · `act` performance · `camera` camera work · `overlay` overlays · `made~` production method · `material~` asset source · `remake` remake route · `flag` needs review |
| `video/content.md` | Content & structure, narrative arc, transcript |
| `video/cast.md` | Cast, settings, props |
| `video/sound.md` | Voice, BGM, SFX, beats, and sources |
| `video/rhythm.md` | Pacing & keyframes: cuts/min, shot lengths, visual change curve (one digit per 0.5s), cut/transition timestamps & durations (`@N` frame numbers), and per-shot keyframes (timestamp + type: first frame, action start/peak, turn, element enter/exit, framing shift, expression peak, mid-transition, etc.) |
| `video/look.md` | Framing, camera setup, lighting, editing, and gear for live-action shots (generated when live-action shots exist) |
| `video/packaging.md` | Timing and styling of subtitles, styled captions, stickers, and PiP |
| `video/remake.md` | Remake plan: generation units, shooting schedule, code compositing, required real assets, performance & mood |
| `video/shots/<id>.md` | Per-shot `inferred` (production method, probability, runner-up, reviewer, real assets), `flags`, `keyframes`, `frames`; body with observed visuals, inferences + evidence, and prompts |
| `breakdown.json`, `shots/`, `stems/`, `sfx/`, `packaging/`, `frames/` (when `code_render` shots exist), `storyboard.jpg`, `audio.png` | Full raw data and media assets listed in the map |

Topic files with no content are omitted.

## Commands

```bash
rea-video decompose ref.mp4 -o out/ref      # Generates VIDEO.md package alongside breakdown
rea-video videomd out/ref                   # Regenerates package from breakdown.json
rea-video videomd out/ref --check           # Validates package against VIDEO.md spec
```

```python
import videomd
pkg = videomd.read("out/ref")   # {"card", "map", "timeline", "shots"}
```
