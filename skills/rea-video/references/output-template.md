# Delivering to the user

## Principles

- Keep your reply split into the same two parts as the report — don't blend them:
  - **The breakdown**: a one-line verdict (how this video was made), two or three lines on content and cast (what it's about, the main characters/scenes/products), the shot structure, editing/packaging, audio, real assets, and open questions. Write it shot by shot — don't merge shots by production method.
  - **The remake plan**: organized around whichever model the user picked (this is part two of the report, produced by `rea-video plan`). If they haven't picked one yet and there are AI shots, ask which model first — don't pick one for them and produce a plan anyway.
- Lead the remake plan with "follow these steps": part two of the report already opens with a step-by-step sequence organized by footage type (the breakdown package's `video/remake.md` "follow these steps") — relay it as-is, keep it short: one sentence per step the user can act on directly, leaving measurement numbers in the report's own section. If a shot's type was misjudged, say so and give the steps in the correct order for the right type. State which model and capabilities the plan is built around (the first line of report part two).
- When there are AI-generated shots, add a reminder: the prompts in the report use a generic structure — before generating, revise them against that model's own official prompting guide or the matching prompting skill.
- Organize the remake plan the way it would actually be produced, not shot-by-shot in editing order:
  - AI shots → **generation units**: several consecutive shots generated in one call, which model, reference material (`@image` for product/person, `@video` for an original clip), the merged multi-shot prompt, and how units connect to each other. If the default engine doesn't support multiple shots per call, copy the `engine_note` verbatim, noting how many calls a different model would save.
  - Live-action shots → **shoot blocks**: which shots share the same scene and camera position and should be shot together; the lighting plan for each scene; wide shots before close-ups. The shot list itself still gives detail shot by shot.
  - Shots with someone performing → spell out how each shot's emotion and performance get reproduced (the report's "Performance & emotion": expression, action, tone, and whether it's first-frame-sets-the-expression, dub-first-then-drive, performance transfer, or an actor performing to match).
  - Code-render → however many code-composite pieces, with each shot's visual elements and motion spelled out (the report's "code composite" section's per-shot motion timeline: element position, appearance time, direction, speed, color value, with element screenshots under `shots/`); still-image pans → rebuilt locally.
- When a prompt is missing (no VLM was run), fill it in yourself by eye: in `rea-video review`'s answers, fill `replication.image_prompt / video_prompt / video_prompt_en` per shot — `apply-review` then carries these into the generation units automatically.
- Give the user the report file path (`report.html`). When there are code-render shots and the user wants to remake it, tell them the remake needs the whole report directory (the element screenshots and keyframe strips under `shots/`, the subtitle files under `packaging/`) — handing over only the HTML or PDF will be missing individual screenshots. Don't paste the whole of `breakdown.json` into the chat.

## Language

The report's language follows `--lang` (zh / en); your reply's language follows the user — see SKILL.md's "Delivering to the user" for how each is decided. The template below is in English; use the same structure regardless of language: one-line verdict → content and cast → remake plan → shot list → packaging → audio → material you must supply → open questions → report path.

## Reply template

**One-line verdict**: Mixed production — underlying footage is 62% AI-generated video, 30% code template, 8% live action; subtitles throughout are composited in code; 2 real screenshots are embedded; a female TTS voice over 110 BPM pop BGM, with cuts landing on the beat.

**Content & cast**: A narrative short, 4 shots; lead character A, "Xiao Huang" (a yellow cartoon hippo, pink onesie, pink crossbody bag, appears throughout), B the boyfriend (matching color scheme, #3–#4); scene S1 a city street in daytime. For the remake, pin one reference image each for A and B and carry it across all generation units.

**Shot list**

| # | Time | Visible in frame | Overlays | Production method (inferred, confidence) | Basis for the inference | Asset source | Remake route |
|---|---|---|---|---|---|---|---|
| 1 | 0:00–0:02.5 | Xiao Huang waving by the street; left-hand fingers deformed | Subtitle, title | AI video (86%) | 24→30 duplicate frames | AI-generated | capsule-cinema first/last-frame |

"Visible in frame" only lists what you can verify against a screenshot (content, camera move, tells); tool measurements like frame rate, noise, and metadata go under "basis for the inference," and the production method is marked as inferred. Write it out when the two disagree.

**Shooting, lighting & editing** (write this when there are live-action shots, matching the framing / camera position / lighting / editing / format lines at the end of `rea-video summary`): video format (talking-head/tabletop/vlog), shot size and camera angle, handheld vs. tripod, number of camera positions/scenes; lighting setup (key light direction and height, lighting ratio, hardness, color temperature, any fill/rim light, visible light sources in frame); editing technique (jump cuts on breaths, push-in cuts, the proportion of inserted screenshots/recordings); give a gear list for the remake. State your basis when the measurement conflicts with what you see by eye.

**Editing/packaging**: subtitles (how many lines, position, color/outline/background box, keyword highlighting, entrance style, SRT/ASS file paths); styled text, titles, chapter cards, tracking labels (time, text, style, animation); stickers/GIFs/emoji/picture-in-picture (time, position, what it is, whether it loops); which sound effects land on a text pop-in or sticker appearance; which styled text/sticker is anchored to which line of dialogue ("anchored to dialogue"), and which are persistent layers. Note anything the tool missed that you found by eye.

**Audio**: voice (TTS or real, timbre — dialogue is in the report's script table) / BGM (song-title guess, style, BPM, key, source and licensing risk, sections) / sound effects (timestamp → matching on-screen event) / mix (is the music ducked). Each item in the report's "Audio" section is its own card — copy it over as-is.

**Real assets you need to supply**: the GitHub page screenshot in the middle of shot #4...

**Remake plan**: write it from `production_plan`. AI shots as generation units (shot range, model, reference material, multi-shot prompt, how they connect); live-action shots as shoot blocks (scene → camera position → shots, lighting); code-render as however many code-composite pieces, what approach each uses, pointing to the per-shot motion timeline and element screenshots; still-image pans get one line. Confirm with the user before anything that costs money.

> Example: 8 AI shots → 3 generation units, 3 Seedance 2.5 calls (the default Seedance 2.0 doesn't support multiple shots per call, which would take 8 calls shot-by-shot). G1 #1–#5 (9.0s): @image1 product photo + @video1 original 0–9s, produces 5 shots in one call...

**Open questions**: #7 live action or AI unconfirmed (leaning AI 64%): no hands or text to judge by — suggest...

**Report**: out/ref/report.html

**VLM**: copy the `VLM:` line from `rea-video summary` verbatim (active / partially active / all calls failed / not configured / off). If it's not active and you can't see images either, add one line of advice: "Configure any multimodal model (REA_VLM_BASE_URL / REA_VLM_API_KEY / REA_VLM_MODEL) and rerun with --vlm — it will sharpen the live-action/AI/3D calls." If you can see images yourself, just write "reviewed visually by the agent."
