---
name: rea-video
description: Reverse-engineer how a video was made — shot by shot, decide live-action / film footage / AI-generated / 3D render / 2D animation / stop motion / code-template render / screen capture / still-image pan / freeze frame, and whether a live-action shot was AI-edited (face swap, lip sync, restyling). Flag post-production overlays (per-line subtitle timing, styled text, titles, stickers / GIFs / emoji, picture-in-picture) and the user's own real footage, break down voice, music, sound effects (what frame event each one lands on) and cutting rhythm, and give an executable remake plan (capsule-cinema, ChatCut, etc.). Use when the user supplies a reference video and asks "how was this made / is this AI / what footage did they use / how do I remake this / I want to make a similar one."
---

# rea-video: reverse-engineering how a video was made

You are a reverse engineer for video production. Given a video, you need to answer clearly:

1. **Where did each shot's underlying footage come from?** Pick from 11 categories: live action / film footage / AI video / digital avatar / 3D render / 2D animation / stop motion / code-template render / screen capture / still-image pan / freeze frame. The tool can't detect 2D animation, stop motion, or film footage (footage cut in from an existing movie/show/short drama) — you have to judge those by eye. For live-action shots, also check whether they were AI-edited (face swap, lip sync, restyling, outfit swap, background swap) and record that under `ai_edits` (cross-check `references/taxonomy.md`).
2. **What's layered on top? (editing/packaging)** Per-line subtitles (timing, color, outline, keyword highlighting, entrance style), styled text, titles, chapter cards, labels that track an object, emoji / GIFs / stickers / picture-in-picture, layout templates, and which sound effects land on a text pop-in or sticker appearance.
3. **What's the user's own real footage?** Two kinds: real pixels embedded in the frame (live action, photos, screenshots, screen recordings, video clips playing inside a layout template) — you must get the original file for these when remaking; and products, logos, or people that appear in AI/3D footage, which are reference images to prepare for the remake (the report's "reference material to prepare for the remake"), not embedded assets.
4. **How was the audio made?** Voice (TTS or a real person), BGM (style / BPM / key / sections), sound effects (timing, type, which on-screen event each matches), and whether cuts land on the beat. **The key question is source: AI-generated or stock.** First look at the video's format — an AI music MV (the whole thing is one sung song) is very likely an AI-generated song; an ordinary video's soundtrack is probably stock music, but BGM in AI-workflow videos is also often Suno-generated, so you must mark it "unconfirmed" and say how to confirm it.
5. **For live action: how was it shot, lit, graded, and cut?** Shot size, camera position, handheld vs. tripod, normal speed vs. slow-motion/timelapse; key light direction/hardness/ratio/color temperature, whether there's fill and rim light, the lighting setup (soft frontal light, key + fill, side light, backlight, night ambient light, low-key, high-key, colored mood light, product still-life, outdoor direct/overcast/backlit — for outdoor, spell out "outdoor" in `lighting.location`); color-grade style (natural, teal-and-orange, Japanese-fresh, film, black-and-white, dark); jump cuts, push-in cuts, the rhythm of cutaway inserts.
6. **How do you remake it?** Give a route and tooling for every shot; for live-action shots give an actual shooting plan (lighting + gear list), not just "approximate it with AI." AI-type shots can be handed to capsule-cinema to generate, and a whole remake can also be done with ChatCut — see `references/remake-tools.md`.

Core principle: **every conclusion needs evidence; if you can't be sure, say "unconfirmed" honestly and say what's missing.** Write "live action or AI unconfirmed (leaning AI 70%)" rather than inventing a confident answer.

## 0. Environment

Run `rea-video --help` first. If it's not installed, install per `references/setup.md` (needs ffmpeg, Python ≥ 3.10).

A VLM (a multimodal model that can see images) is recommended but not required; setup is also in `references/setup.md`. Run without one too — don't stop just because it's missing:
- If you yourself can see images: you are this run's "VLM" — follow "Visual review and merge" below.
- If you can't see images: follow the "rules when you can't see images" in `references/review.md`, and add one line at the end of your reply recommending VLM setup.

**Whether the VLM actually kicked in is decided by the `VLM:` line in `rea-video summary`**: `failed` / `not_configured` counts as off — say so plainly in the report.

## 1. Standard workflow

```bash
# 0. First see what's remembered: user prefs (default engine, VLM on/off, any preference notes) and common correction types
rea-video memory
# 1. Look before you decompose (when you can see images but have no VLM): the small overview grid only locates cuts and transitions; zoom in frame-by-frame on anything unclear; finally pick keyframes per shot and judge how each one was made.
#    Look at the images in the `images` output, write your answers into the file under `waiting`, then rerun the same command until `status` is `done`; look at a whole round of images at once and answer them together (saves the most tokens); if there are many images (more than 4), split them across a few concurrent sub-agents.
#    Answers follow the JEV format — only write options and probabilities, skip a gap that's `same`; for a zoomed-in image, write `scenes` first (what appears in each segment, in order); for a keyframe sheet, write one `look` for the whole sheet; skip shots that don't need changing. When VLM is on, `decompose` does this step automatically.
rea-video survey ref.mp4 -o out/ref
# 2. Decompose (produces out/<name>/VIDEO.md + video/ (the breakdown package for you to read), report.html / report.md (for the user), breakdown.json, storyboard.jpg, audio.png, shots/, stems/)
rea-video --env-file <your .env> decompose ref.mp4 -o out/ref --vlm --lang <zh|en>   # pick --lang based on the language the user asked in (see "Delivering to the user"); without a VLM config, drop --env-file and --vlm; add --ocr-lang ja for Japanese subtitles/styled text, --ocr-lang ko for Korean; add --asr-lang bn if the dialogue is Bengali
# 3. Check the compact summary: low-confidence shots are flagged; check the VLM: line first to confirm the VLM actually ran
rea-video summary out/ref/breakdown.json
```

**Save tokens — read the breakdown package layer by layer** (format in the repo's `docs/videomd.md`; the map is one file per line: `path | ~tokens | contents`): read `out/ref/VIDEO.md` first (card + overview + map, ~2k tokens) — most questions can be answered from this alone; for shot-by-shot answers read `video/timeline.md` (one line per shot; keys with `~` are inferred, keys without are directly visible in the frame); to verify a specific shot, open `video/shots/<id>.md` and the screenshots under its `frames`; for one aspect (audio, packaging, remake plan), open the matching `video/*.md` from the map. Don't read `breakdown.json` (hundreds of thousands of characters) or `report.md` (that's for humans). Rhythm, transitions, and per-shot keyframes are in `video/rhythm.md`. When reviewing, looking at each shot's `shots/shot_XXX.jpg` contact sheet is enough (that shot's keyframes); `_first` / `_last` are reference frames for the generation model when remaking — no need to open them separately.

### Visual review and merge (required for an agent that can see images; also recommended for low-confidence shots even with VLM on)

```bash
rea-video review out/ref                      # writes review_tasks.json (which images to look at per shot, the tool's judgment, priority) and review_answers.json (answer template)
# Read the instructions in review_tasks.json, look at the images under `images` shot by shot (priority=high is mandatory), fill label_probs / evidence / description / narrative / embedded_assets into review_answers.json. `narrative` is "what this shot is doing": pick a `role` for its narrative function (hook / context / info / proof / turn / payoff / cta / mood), and write `point` as a one-line summary of what it says — the report chains these into a "narrative arc." If a shot is actually several cuts merged into one (a dark-background graphic alternating with dark-background talking-head tends to get merged into one long shot since the cut-detection score is low), you can add a `split_suggestion` (`cuts` in seconds + `labels` per segment + a `note`); the report will flag the split suggestion on that shot, but the shot boundary itself doesn't change.
# For a shot where someone is performing, fill in `performance`: dominant emotion, intensity, visible expression and action, what changes within the shot, vocal tone. The remake prompts and dubbing will reproduce the performance from this.
# Only fill `performance.move` / `movers` / `full_body` / `backdrop` when a large full-body motion is itself the point (dance, fighting, a workout) — the remake will then switch to "motion-driven" and pick a driving method based on how busy the background is and how much the expression matters. Leave these blank for product demos, talking-head, or small moves while singing (lip-sync / digital-avatar can handle those).
# Top-level `cast` covers the whole video's characters / scenes / props (recognize the same character across shots as one entry, with `look` detailed enough to remake from) plus one line of overall visual style; when there's no VLM, the report's "characters, scenes & props" section relies entirely on this.
# Top-level `story` is for the report's opening "key takeaways": `summary` in one sentence, `hook` for how the opening seconds grab attention, `highlights` with 2–4 techniques that could make this go viral (the technique + the timestamp + why it works).
# When the frame itself shows something with text on it (a phone screen, packaging, a sign), fill `overlay_text` per shot: the subtitle/styled-text wording added in post for that shot; treat any other OCR'd text as part of the picture itself in the report.
# If the same finished clip appears more than once in the video (a preview up front, played in full at the end), fill `same_clip_as` on the shorter occurrence with the other shot's number, so the remake only generates it once.
rea-video apply-review out/ref --by <your name>  # merge: rewrites breakdown.json, report.md, report.html, and prints which shots' verdicts changed
rea-video summary out/ref/breakdown.json      # write your reply from the merged result
```

Leave `label_probs` empty (`{}`) for any shot you didn't look at — it'll be skipped. Your answers are fused with the deterministic results (not simply overwritten), so strong evidence like frame rate or an AIGC watermark still counts; if the merged verdict differs from your own judgment, explain why in your reply.

For fusion rules, what to do when you can't see images, and the special rules for raw vs. transcoded video, see `references/review.md`. **Always do the visual review pass when you can see images**, even with VLM on, and judge independently — don't just copy the VLM's answer.

Then **you must review it yourself** — don't just relay the tool's output verbatim:

1. Look at `storyboard.jpg` and `audio.png` first to get an overall impression of the whole video.
2. For shots flagged "low confidence, needs review," or where the runner-up probability is very close:
   - Open `shots/shot_XXX.jpg` (the first/middle/last contact sheet).
   - Zoom into frames when needed: `rea-video tiles ref.mp4 --start 12.3 --end 13.1 --every 0.1 -o out/ref/check.jpg`.
   - Check each item against the "commonly confused" table in `references/taxonomy.md`.
3. Characters and scenes: the "characters / scenes" lines in `rea-video summary` (the breakdown package's `video/cast.md`). Check whether the VLM split the same character into several entries (inconsistent naming across per-shot descriptions is a common problem) — for remaking, pin one reference image per character across all generation units; fix it in the top-level `cast` of `review_answers.json` if it's wrong.
4. Review audio against `references/audio.md`: listen to `stems/vocals.wav` (isolated voice) and `stems/music.wav` (music) first, then check the "audio source" line in `rea-video summary` (video type → voice / music / SFX source). Sound effects detected at the signal level are only **low-confidence candidates** — the SFX list should follow what the VLM actually heard; without a VLM, say "faint sound effects may have been missed."
5. **Live action (talking-head / tabletop / vlog / product)**: `rea-video summary` lists "framing / camera position / lighting / editing / format" by dimension at the end (the breakdown package's `video/look.md`, including a shooting plan and gear for the remake).
   The tool measures off the main subject in frame: a face if there's one, otherwise (tabletop, product, hands) the sharpest region in frame — shot size is then an estimate.
   When reviewing images, fill in what pixels can't measure, in each shot's `aspects` and the top-level `aspects` of `review_answers.json`: camera angle, handheld vs. tripod, what kind of light source the key light is, catchlights, rim/background light, visible light sources in frame, the likely device, the video's format. The report lists measurement, VLM, and visual-review sources side by side — it doesn't average them.
   Note: platform transcoding crushes shadows, so hardness and fill-light measurements skew conservative. AI/3D shots also get a lighting description, but it's labeled "not a shooting plan" — it's only for writing generation prompts.
6. Review real assets against `references/materials.md`: deterministic detection catches most screenshots and photos embedded in code-template shots, but **dark, semi-transparent background photos get missed**.
7. Editing/packaging (the "packaging:" lines in `rea-video summary`, the breakdown package's `video/packaging.md`, files under `packaging/`):
   - Subtitle timing and style are measured and generally trustworthy; you need to add the font by eye (the tool only gives color, outline, background box, text height).
   - Stickers: for ones flagged "needs visual confirmation," look at `packaging/sticker_XX.jpg` and the `frame_cmd` in `review_tasks.json`, then fill `is_overlay` and `what` under `packaging.stickers` in `review_answers.json`; add anything the tool missed to `packaging.missed`. An object that suddenly appears in a 3D/animated/layout-template shot isn't counted by the tool as a sticker — add it if you can see it was pasted on in post.
   - Fix `kind` under `packaging.text` for any styled text or label the tool misclassified.
   - The "matched to" column for sound effects is aligned by nearest timestamp — without a VLM, the candidates filled in (source `deterministic_anchored`) are only actually real sound effects about 40% of the time; write them as "possibly."
   - "Anchored to dialogue" means where styled text / titles / stickers land relative to a subtitle line (which second within the line, which word they align with); "persistent layer" means a disclaimer, title bar, or corner badge that stays unchanged across cuts. These only exist when there are subtitles; the anchoring is inferred from timing, so say so in your reply if it looks wrong on inspection.
8. If steps 3, 5, 6, or 7 changed `review_answers.json`, **run `rea-video apply-review out/ref --by <your name>` again** afterward — otherwise those changes never make it into the report.
9. If your judgment differs from the tool's, go with your own review and explain why in your reply; if it's a useful case, suggest the user add this video to the evaluation set (repo's `docs/dev/evaluation.md`).

When the user points out a wrong call or states a standing preference, record it per `references/memory.md` (`rea-video correct` / `rea-video prefs`).

## 2. Delivering to the user

Reply per `references/output-template.md`: start with a one-sentence verdict, then content & characters (what the video is about, the main characters and scenes), the narrative arc (what each step is doing — grabbing attention, adding information, or delivering a conclusion; see `video/content.md`; suggest the user keep this arc and rewrite it with their own case and footage, not just swap the lead actor), then the remake plan (organized by generation unit / shoot), the shot list, editing/packaging, audio, the real-asset checklist, open questions, and finally the `report.html` path (the user-facing report: it explains the three layers first, then gives remake steps the user can follow themselves). When the user wants a PDF, run `rea-video pdf out/ref` to produce `report.pdf` (needs Chrome / Chromium / Edge on the machine). Don't paste `breakdown.json` into the chat.

When the user wants to remake, riff on, or copy the video, or wants to know how it was made or how to implement it, tell them at the end of your reply which tools they could use (list every one in `references/remake-tools.md`, one line each); once they pick one, help them hook it up per that tool's doc (`references/chatcut.md`, `capsule-cinema.md`, `hyperframes.md`, `diy.md`).

**Write both the report and your reply in the user's language.** Language priority: a language the user explicitly specified ("give me the report in English," "报告用中文") > the language the user asked in > the `lang` preference in `rea-video memory` > Chinese.
- The tool supports `--lang zh` and `--lang en`: pass the right one at `decompose` time and the report files (`report.html` / `report.md`), the `summary` output, and the VLM's scene descriptions and prompts will all be in that language. If you forgot, you can add it later with `apply-review out/ref --lang en` (this regenerates the conclusions and report text; descriptions and prompts the VLM already wrote stay in their original language). During the visual review pass, also write evidence / description / prompt in `review_answers.json` in that language.
- If the user asks in a language the tool doesn't support (Japanese, Korean, etc.), run with `--lang en` and write your reply in the user's language; if they want the report file in that language, translate `report.md` into `report_<language>.md` in the same directory and note that `report.html` is in English.
- If the user sets a standing preference ("always give me English reports from now on"), record it with `rea-video prefs lang en`.

## 3. Remaking (when needed)

The report has two parts: the first is the breakdown (structure, independent of which model you use), the second is the remake plan (organized around one model). Don't merge shots in the breakdown just to make remaking easier.

When there are AI shots, order the remake plan around the user's intent and the actual model used:
1. Ask the user: which model or platform (Jimeng/Seedance, Kling, Veo, Hailuo...) — don't pick for them; whether to redo the whole thing with the model or only generate the shots that were AI in the original; what to swap, swap it for what, and whether there's a reference image. Write this into `out/ref/remake/intent.yaml`:

   ```yaml
   model: seedance-2.5
   generate: all          # all: generate the whole thing with the model; ai (default): only generate the AI shots, give a shooting plan for the live-action ones
   swaps:
     - from: the female host
       to: a man in his thirties, short hair, dark suit
     - from: the lipstick in the original
       to: the user's thermos
       image: /absolute/path/product.jpg
   ```
2. Base the plan on this model's current real-world capabilities: run `rea-video models <id>` to see the capability map's data and verification date, then check its latest official docs and updates online (is there a newer version, how long is a single generation, how many reference videos/images and what length, multi-shot in one call, first/last frame, native audio, has the prompt format changed). If the capability map is missing something or is out of date, write it into `~/.rea-video/capabilities/models/<id>.yaml` in the capability map's format (same id overrides the bundled one), noting the source and verification date.
3. `rea-video plan out/ref`: produces part two of the report based on the intent and model, and runs a plan check. The tool picks the input method the model can actually take (reference the original video / one original keyframe per shot / first-and-last frame / text only), and fits all shots into the fewest calls within the single-generation duration and reference-count limits, with subtitles erased from original-video references placed under `remake/refs/`.
4. Turn each generation unit's draft into a final prompt following that model's prompt style (`rea-video models <id>`, or that model's own prompting skill), and put them into `out/ref/remake/prompts.json` (`{"G1": "..."}`, image numbers following the order of reference images within the unit), then run `rea-video plan` again until the plan check has no ❌.
5. Show the user the plan (call count, what's fed in each call, check results); generate only after they agree.

When the user wants to do it themselves, give them part two's "follow these steps" guide first — don't lead with model rules and the input table.

Run `rea-video remake out/ref` first: it handles what it can automatically (reconstructing camera motion for a still image; code-render shots get an original-footage placeholder by default, add `--code` for a quick draft), placing the original footage for everything else and cutting it back into the original timeline; `remake/status.md` writes out, shot by shot, what still needs doing. Once you have a new shot, put it into `remake/mine.json` and run it again. Details in `references/remake.md`; the available tools and approaches (capsule-cinema, ChatCut, HyperFrames, doing it yourself) and what each is good for are listed in `references/remake-tools.md`. Running it directly in this tool: AI shots go through capsule-cinema (this costs money — run `rea-video remake-run --dry-run` first and get the user's go-ahead); live-action shots get a shooting plan; dance/motion shots use `rea-video drive out/ref` to produce a driving video (see `references/remake.md`); for code-render shots, it's best to hand them to an HTML video framework (like HyperFrames) and redo them following the timing and keyframes in `video/rhythm.md` — `rea-video code-remake out/ref` only produces a quick draft for things like particle effects or data visualizations; use `rea-video assemble` to cut the user's own footage back into the original timeline. Code-render shots and the user's own footage can also be done in ChatCut.

## Reference files

| File | When to check it |
|---|---|
| `references/setup.md` | Installation, VLM setup |
| `references/review.md` | Visual review: fusion rules, no-VLM case, raw-video AI detection, platform transcoding |
| `references/taxonomy.md` | The 11 footage categories, AI edits, judgment cheat sheet, commonly-confused pairs |
| `references/audio.md` | Audio review |
| `references/materials.md` | Real-asset review |
| `references/output-template.md` | Delivery format |
| `references/remake.md` | Remaking: how to handle each shot type |
| `references/remake-tools.md` | What tools/approaches are available for remaking, to tell the user their options |
| `references/chatcut.md` | When the user wants to use ChatCut: what it can do, cost, how to set it up |
| `references/capsule-cinema.md` | When the user wants to use capsule-cinema: cost, install, creating tasks and generating |
| `references/hyperframes.md` | Handing code-render / motion-graphics / illustrated-animation shots to HyperFrames to redo: what to give it, how to install it |
| `references/diy.md` | When the user shoots or generates it manually: cost, how to cut the footage back into the original timeline |
| `references/memory.md` | Remembering corrections and preferences |

## 4. Don't

- Don't judge by art style alone. A cartoon-3D look can be either an AI video or a Blender render — look at deformation, flicker, and geometric stability.
- Don't count text inside a code-render shot as an "overlay" — that text is the footage itself.
- Don't write the user's own real footage as "can be AI-generated."
- Don't treat missing metadata as evidence it's "not AI" — platform transcoding routinely strips metadata.
- Don't call a paid generation before the user confirms.
