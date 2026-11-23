# What It Detects

## How Visuals Are Made

Each shot is classified into one of the 11 categories below. "Tool-measured" means it can be detected offline without a vision model; the rest rely on a vision model or an agent inspecting frames.

| Category | Common Examples | Tool-measured | Evaluated Videos |
|---|---|---|---|
| Live action | Talking-head, vlog, product demo, store visit | ✓ | 5 |
| AI-generated video | Text/image-to-video, AI short drama | ✓ | 24 |
| Avatar | A portrait driven by audio (lip-sync) | Partial | 0 |
| 3D render | Blender / C4D explainers, product animation | ✓ | 14 |
| 2D animation | Hand-drawn, Live2D, After Effects character animation | ✗ Visual | 0 |
| Stop-motion | Clay, felt, paper puppets shot frame-by-frame | ✗ Visual | 0 |
| Film / TV clips | Movie, TV drama, or short-drama clips cut into recaps or mashups | ✗ Visual (tools give clues) | 8 |
| Code / template | Kinetic typography, UI demos, charts, slide-style cards | ✓ | 16 |
| Screen recording / screenshot | Software tutorials, web demos | ✓ | 11 |
| Ken Burns (still pan/zoom) | Zooming and panning across a single still image | ✓ | 3 |
| Still frame | Static image or card held on screen | ✓ | 5 |

Each shot also separates and labels:

- **Post-production overlays:** subtitles, styled captions, titles, stickers, layout templates. Text printed on physical objects (packaging, signs) is separated from subtitles whenever possible. See "Editing & Packaging Layer" below.
- **Real source assets:** embedded screen recordings, screenshots, photos, or video clips that require original files when remaking; products, logos, and characters in AI shots are listed as reference images for remakes.
- **AI post-processing on live footage (visual check):** face swap, lip-sync, stylized redraw, AI outfit change, background replacement, object add/remove, beauty filter. The remake plan specifies "film first, then apply which AI step and which model."

## How Live Action Is Shot

| Aspect | What Is Extracted |
|---|---|
| Framing | Shot size (wide to extreme close-up), camera height, composition |
| Camera | Handheld / gimbal / tripod, push/pull/pan/tilt, phone vs. camera, slow-mo / time-lapse / speed ramp |
| Lighting | Key light direction & height, contrast ratio, soft vs. hard, color temperature, background bokeh, rim light, catchlight, visible practical lights |
| Lighting setups | 12 presets with gear lists: front soft, key + fill, single-side hard, backlight, night ambient, low-key, high-key, colored ambient (RGB / neon), tabletop product still-life, outdoor direct sun, outdoor overcast, outdoor backlight |
| Color grade | Style (natural, teal-orange, Japanese film, vintage film, B&W, moody dark, high saturation) plus saturation, contrast, black point, warmth |
| Editing | Pacing, jump cuts, punch-in cuts, insert-shot ratio, whether cuts hit the beat |
| Format | Talking-head, tabletop, handheld product demo, vlog |

Contrast ratio, softness, color temperature, and color grade are measured from pixels; light source type, rim light presence, outdoor/indoor, and frame-rate feel are supplemented by visual inspection. Both are shown side by side in the report.

## Editing & Packaging Layer

The entire video is analyzed along one continuous timeline rather than split by shot cuts (since a single subtitle or sticker often spans across cuts).

| Item | How It Is Measured | Output |
|---|---|---|
| Subtitle timeline | 2 fps OCR tracking text at the same screen region line by line, then refined against 10 fps grayscale frames to ~0.1s precision; two-line subtitles merged into one entry | `packaging/subtitles.srt` |
| Text styling | Measured from pixels: fill color, stroke color & thickness, background box color, gradient, highlighted keyword color (including mid-sentence color shifts), font size, slant, weight | Style column in report |
| Text classification | Subtitle (continuous lines in the same spot), styled caption (large / colored / slanted / thick stroke / multi-color / pop-in, matching ≥2 criteria), title (persistent top header), text callout (chapter cards, object-tracking labels), in-scene text | Type per text entry |
| In / out animation | Cut-in, scale pop, fade, slide, word-by-word reveal; mid-line bounce or color shift | Animation notes in report & ASS |
| Stickers / GIFs / PiP | Sharp-bounded static region over moving footage; stays fixed across a scene cut; pops in/out mid-shot (marked "suspected" on live footage); looping internal motion with fixed period indicates a looping GIF | Timestamp, position, crop (`packaging/sticker_XX.jpg`). ~50% tool recall; missed ones caught visually |
| SFX beat sync | Each SFX aligned to the nearest visual event (subtitle / caption / sticker appear, or cut, ±0.2s); without VLM, transient audio spikes at caption/sticker timestamps are flagged as low-confidence candidates | "Synced visual" column in SFX table |

Also exports `packaging/overlays.ass` (with color, outline, position, and pop-in animation—burnable directly via `ffmpeg -vf ass=` once matched with the original font) and `packaging/timeline.json`.

Exact font names and sticker content cannot be measured from pixels alone and are filled in by VLM or agent visual review. In 3D, animation, or motion-template shots where objects naturally pop in, only cross-cut or looping overlays are marked as stickers; the rest are treated as scene content and verified visually if needed. See [eval/RESULTS.md](../eval/RESULTS.md) for benchmark results.

## Audio

- **Voice:** Human vs. AI voiceover, plus full spoken transcript.
- **Music:** BPM, key, sections, AI-generated vs. stock library music.
- **Sound effects:** Timestamp, type, and the visual event each SFX lands on (subtitle / caption / sticker appear, or cut).
