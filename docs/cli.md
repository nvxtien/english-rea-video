# CLI Reference

Normally you don't need to type these commands manually: hand the repository and video to an agent and it will run them following [SKILL.md](../skills/rea-video/SKILL.md). This reference is for running commands directly or seeing what the agent does under the hood.

## Installation

Requires Python ≥ 3.10 and ffmpeg (macOS: `brew install ffmpeg`; Ubuntu: `apt install ffmpeg`).

```bash
git clone <this-repo> && cd rea-video
pip install -e ".[ocr]"          # numpy / opencv / pillow / pyyaml / onnxruntime + optional OCR
rea-video --help
bash scripts/install_skill.sh    # Optional: installs CLI and links skill to ~/.claude/skills, ~/.codex/skills, ~/.agents/skills
```

When installing `[ocr]` manually, the OCR package also pulls in `opencv-python`, which conflicts with `opencv-python-headless` in the same `cv2` directory. Run once after installing:
`pip uninstall -y opencv-python && pip install --force-reinstall --no-deps "opencv-python-headless>=4.8,<5"` (`install_skill.sh` handles this automatically).

Claude Code can also install the skill as a plugin (still requires `pip install`; step 1 of the skill checks for it):

```text
/plugin marketplace add JuneYaooo/rea-video
/plugin install rea-video@rea-video
```

The first breakdown automatically downloads the ~67 MB vocal-separation model (`~/.cache/rea-video/models`). A 30-second video without VLM takes ~1 minute.

Once installed, simply tell Claude Code / Codex / zcode: *"Break down this video xxx.mp4 and check if it's AI-made"*. The agent follows [SKILL.md](../skills/rea-video/SKILL.md): check memory → decompose → visual review → deliver using the report template.

## Quick Start

```bash
# 1. Deterministic forensics only (offline, free)
rea-video decompose ref.mp4 -o out/ref

# 2. Add a VLM (recommended, optional) for higher accuracy on live / AI / 3D
#    Any OpenAI-compatible multimodal model works; comma-separate fallbacks
export REA_VLM_BASE_URL=https://.../v1 REA_VLM_API_KEY=... REA_VLM_MODEL=gemini-2.5-pro,gpt-4o
rea-video decompose ref.mp4 -o out/ref --vlm
#    Or reuse capsule-cinema's config: rea-video --env-file ~/code/github/capsule-cinema/.env decompose ref.mp4 -o out/ref --vlm
#    If all VLM calls fail, execution continues and the report / `rea-video summary` marks VLM as disabled

open out/ref/report.html

# 3. No VLM API, but your agent can see images (zcode, Claude Code, etc.): let the agent act as VLM
rea-video survey ref.mp4 -o out/ref     # Overview grid first, then zoomed detail: sets cuts, transitions & keyframes (repeat until done)
rea-video decompose ref.mp4 -o out/ref  # Splits shots using survey cuts
rea-video review out/ref                # Generates per-shot review tasks and answer template
rea-video apply-review out/ref --by zcode  # Merges agent answers into verdict and report

# 4. User corrections and preferences are remembered (stored in ~/.rea-video; override with REA_VIDEO_HOME)
rea-video correct out/ref --shots 3 --label still_motion --note "This is stop-motion"
rea-video prefs vlm on && rea-video prefs --note "Keep reports concise"
rea-video memory

# Extract audio stems only: stems/original.wav, vocals.wav, music.wav, effects.wav
rea-video stems ref.mp4 -o out/ref
```

Audio stem separation runs locally on CPU using two models, downloaded on first use to `$REA_VIDEO_MODELS` (default `~/.cache/rea-video/models`):

- **Vocals & BGM:** UVR MDX-Net Inst HQ 3 (~67 MB).
- **Speaker diarization** (requires `rea-video[asr]`): pyannote segmentation 3.0 + 3D-Speaker CAM++ (~35 MB total). Outputs `stems/voice_S1.wav`, etc. when 2+ speakers are present and tags lines by speaker.
- **SFX:** BandIt v2 multilingual (~149 MB; Watcharasupat, Wu, Orife, [arXiv 2407.07275](https://arxiv.org/abs/2407.07275), CC BY-SA 4.0), splitting audio into dialogue, music, and effects stems. SFX detection and preview clips come cleanly from the effects stem without vocals or music (~1.5× audio duration; falls back to mixed audio if model download fails).

Thread counts for stem separation and OCR are auto-selected (performance cores on Apple Silicon, physical cores elsewhere, capped by container limits and max 8); override with `REA_VIDEO_THREADS`.

Example output (mixed media):

```
Verdict: Mixed: Code / template render 59% + AI-generated video 41%; Overlays: subtitles, titles
#1 00:00.000 1.38s Code / template render → R01 Remotion rebuild
#2 00:01.375 2.50s AI-generated video (24fps source, no sensor noise...) → R07 Keyframe generation (capsule-cinema)
...
```

## Remaking With capsule-cinema

```bash
CC=~/code/github/capsule-cinema   # Or set CAPSULE_CINEMA_ROOT / `rea-video prefs capsule_root $CC` once
rea-video plan out/ref --model seedance-2.5                                      # Generates remake plan (Part 2) from remake/intent.yaml & model
rea-video remake-plan out/ref/breakdown.json --capsule-root $CC                  # Converts generation units into jobs
rea-video remake-run out/ref/remake/jobs.json                                    # Runs capsule-cinema generation (paid; test with --dry-run first)
rea-video assemble out/ref/breakdown.json out/ref/remake/results.json            # Stitches onto original timeline + side-by-side comparison; --overlays restores captions
rea-video capsule-draft out/ref/breakdown.json --capsule-root $CC --capsule-name my_remake
```

- **Ken Burns & still frames:** Rebuilt locally using measured zoom/pan parameters without calling video models.
- **Code render & screen capture:** Report provides text tracks and layout specs to rebuild in Remotion.
- **Subtitles, styled captions, stickers:** Excluded from generation prompts; reference frames/videos are cleaned of text before generation, then re-applied via `assemble --overlays`.
- **`capsule-draft`:** Injects breakdown data via capsule-cinema's `tool_factory` to produce a reusable recipe draft without modifying capsule-cinema's code.

See [capsule-cinema.md](capsule-cinema.md).

## Commands

| Command | Purpose |
|---|---|
| `decompose VIDEO [--vlm] [--no-ocr] [--ocr-lang zh\|ja\|ko] [--lang zh\|en] [--no-packaging] [--max-vlm-shots N] [--no-asr] [--asr-lang auto\|bn] [--no-speakers]` | Full breakdown: outputs `breakdown.json`, `report.html/md`, `VIDEO.md` + `video/`, `storyboard.jpg`, per-shot keyframes, `packaging/` (SRT, styled ASS, timeline, sticker crops), and `sfx/` preview clips. Use `--lang en` for English output (`rea-video prefs lang en` sets default); `--no-packaging` skips overlay analysis (~30–50s/min faster); `--ocr-lang ja\|ko` switches OCR model; `--max-vlm-shots N` caps VLM calls (default 60); `--no-asr` skips local SenseVoice ASR; `--asr-lang bn` switches to Bengali ASR. |
| `survey VIDEO [-o OUT_DIR] [--vlm] [--no-detail] [--workers N]` | Watch-before-splitting: round 1 builds an overview contact sheet to spot cuts, transitions, and high-change regions; subsequent rounds zoom into transitions/uncertain boundaries (up to 3 rounds), then sample keyframes per shot. Saves `survey/timeline.json` for `decompose`. Without `--vlm`, prints image paths and answer file for the agent to review. |
| `review OUT_DIR` | Generates per-shot visual review tasks (`review_tasks.json`) and answer template (`review_answers.json`). |
| `apply-review OUT_DIR [ANSWERS] [--by NAME] [--lang zh\|en]` | Fuses agent visual review answers with deterministic results and regenerates `breakdown.json` and reports. |
| `correct OUT_DIR --shots N --label L [--real yes/no] [--note]` | Records user corrections, updates verdict/report, and saves to memory for future runs. |
| `prefs [KEY VALUE] [--note TEXT]` | Manage user preferences (default engine, VLM toggle/config, reviewer name, report language `zh\|en`, agent notes). |
| `memory [--export-labels YAML]` | Inspect stored preferences and frequent corrections; export corrections as eval labels. |
| `probe VIDEO` | Inspect metadata clues only (capture device, AIGC implicit tags, C2PA, editor signatures). |
| `summary breakdown.json` | Print compact verdict and flag low-confidence shots needing review. |
| `videomd OUT_DIR` | Regenerate `VIDEO.md` + `video/` package from `breakdown.json` (see [videomd.md](videomd.md)). |
| `videomd OUT_DIR --check` | Validate package against the `VIDEO.md` specification. |
| `pdf OUT_DIR` | Print `report.html` to `report.pdf` via local Chrome / Chromium / Edge (`REA_CHROME` overrides path). |
| `tiles VIDEO --start --end --every` | Extract and tile frames over a time range for close inspection. |
| `frames OUT_DIR [--every 0.25] [--points 4000] [--all]` | Segment code-rendered shots into scenes and extract foreground layers, sampled points, and `scenes.json`. |
| `compare REF REMAKE [--scenes frames/scenes.json] [-o DIR]` | Frame-by-frame comparison of foreground layers between remake and original, reporting timing offset, position, spread, and color deltas. |
| `drive OUT_DIR [--shots 3-7] [--depth] [--no-clean] [--prefer 10] [--max 15]` | Prepare driving clips for dance/action shots (cleaned of subtitles/stickers, split at motion pauses near beats). `--depth` also exports depth videos. |
| `drive OUT_DIR --join [RESULTS]` | Join generated driving clips back onto the original timeline and render a comparison video. |
| `eval labels.yaml` | Evaluate classification accuracy and cut-detection F1 on labeled samples. |
| `capsule breakdown.json` | Export video analysis JSON for capsule-cinema. |
| `models [ID]` | List generative models in the capability map; pass an ID to print prompting & keyframe specs. |
| `plan OUT_DIR [--model ID] [--generate ai\|all]` | Generate remake plan (Part 2) from `remake/intent.yaml` and validate model prompt formats. |
| `remake-plan` / `assemble` | Convert generation units into jobs and assemble generated clips onto the timeline. |
| `remake-run JOBS [--dry-run] [--only G2\|2,5\|shot_002] [--capsule-root]` | Execute remake jobs via capsule-cinema. |
| `capsule-draft breakdown.json` | Generate a reusable capsule-cinema recipe draft. |

## Workflow Architecture (Roadmap)

```
Ingest ─► Decompose ─► Video IR ─► Plan ─► Reconstruct ─► Verify ─► Remix
```

| Stage | Status |
|---|---|
| Ingest, Decompose, Video IR (`breakdown.json`) | Ready (core of this repository) |
| Plan | Ready: assigns per-shot remake routes and generates capsule-cinema jobs for AI shots |
| Reconstruct | Partial: AI shots via capsule-cinema, Ken Burns locally; code layer outputs text/layout specs (Remotion project written by agent) |
| Verify | Partial: `assemble` stitches clips and exports side-by-side comparison; automated scoring pending |
| Remix | Planned |

1. **Ingest:** Probe metadata, extract frames, separate audio stems.
2. **Decompose:** Split labor between deterministic CV (cuts, OCR tracks, segmentation, camera motion, beats, word alignment) and VLM (intent, performance, style).
3. **Video IR:** Timeline → tracks → segments, each tagged with render mode (`code` / `gen` / `footage` / `audio`), parameters, evidence, and unknowns.
4. **Plan:** Consult [Capabilities Map](../rea_video/capabilities/README.md) to route each unit to primitives and models, outputting a Shot Recipe with cost estimates.
5. **Reconstruct:** Render code layers (Remotion), generate AI layers, produce voice/music/SFX, and composite.
6. **Verify:** Check cut alignment, OCR text match, layout IoU, perceptual similarity, subject consistency, motion trajectory, beat sync, and lip-sync.
7. **Remix:** Re-render only affected layers when swapping cast, products, copy, language, or aspect ratio.

## Two Modes

- **fidelity:** Target exact alignment across structure, pacing, camera moves, text, and motion graphics.
- **adapt:** Preserve *why* the original works (hook, reveal, pacing) while rewriting content for a new topic.

Modes can be mixed per layer—for example, `fidelity` for pacing and camera motion, `adapt` for characters and copy.
