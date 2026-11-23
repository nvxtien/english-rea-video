# Production-method taxonomy and commonly-confused pairs

## The 11 underlying-footage categories

| Label | Name | Typical evidence | Remake route |
|---|---|---|---|
| live_action | Live action | Sensor noise σ>1, temporal noise >0.8, pixel-identical ratio <0.55, handheld shake, native 60fps, camera metadata | Real footage (user-supplied) or an AI image-to-video approximation |
| ai_video | AI-generated video | Photo-like texture with no sensor noise, 24→30 duplicate-frame or frame-interpolation periodicity, smoothly floating camera moves; VLM sees deformed fingers/text/objects or texture flicker | capsule-cinema first/last-frame generation |
| avatar | Digital-avatar talking head | Fixed frontal shot, only the mouth and head move, audio-driven | TTS/voice clone + digital avatar |
| cg_3d | 3D render | Flat/clean materials but the background isn't pixel-frozen, geometry is stable with no deformation | Blender/C4D, or an AI first/last-frame approximation |
| code_render | Code/template render | Flat blocks >20%, fewer than 70 colors, many horizontal/vertical edges; the background is pixel-frozen with only local elements moving; no material texture (smooth color gradients, large saturated flat-color blocks, glow on a dark background, fractal/shader color fields); on a dark background, the bright areas are all tiny specks or thin lines (particles, point clouds, wireframes) | Rebuild in Remotion/HTML; the report gives an approach per visual type (particles → Three.js, color fields → GLSL, gradients → CSS blur, charts → SVG/D3) |
| screen_capture | Screen recording/screenshot | Large light-colored UI areas, dense small text, low saturation, whole-page scrolling | Real UI recording (user-supplied) |
| still_motion | Still-image pan (Ken Burns) | Rigid-transform inlier ratio >95%, residual <0.3px, no residual after compensation, no 24fps frame-rate artifacts | One image + local ffmpeg rebuild using the measured parameters |
| static_hold | Freeze frame | 85%+ of frames are pixel-identical | One image |
| anim_2d | 2D animation | Undetectable by the tool, visual judgment only: stable lines and flat color blocks, the character has rigging or frame-by-frame motion, often "animated on twos" | Character turnaround sheet + first/last-frame generation; do it in AE / Live2D / Spine when precise lip sync/motion is needed |
| stop_motion | Stop motion | Undetectable by the tool, visual judgment only: real materials (clay, felt, paper), jerky motion, slight inter-frame flicker, consistent lighting on the same physical object | Best to actually shoot it (fixed camera, ~12 shots/sec); AI can only approximate |
| film_footage | Film/TV footage | Undetectable by the tool, only gives clues: the picture is letterboxed in black bars, a 24/25fps source. Judge by eye: recognizable actors or a show, professional lighting with multi-camera shot/reverse-shot, large numbers of extras and full sets, a channel logo or platform watermark | Find and cut in the actual source clip (copyright belongs to the studio — needs licensing for commercial use); without the original, shoot your own with the same framing and lighting, or approximate with AI |

`anim_2d` / `stop_motion` / `film_footage` are decided by visual judgment only: if only one side (VLM or agent) has looked, mark the verdict "unconfirmed"; only drop that once both have looked.

`live_action` is only for footage the video's creator or client shot themselves; footage cut from an existing movie, TV show, or short drama is always `film_footage`, no matter how much it looks like live action.

## AI edits on live-action footage (`ai_edits`)

When the underlying footage is live action or stop motion, also check whether it was AI-edited afterward, and record it in `ai_edits`; the remake plan will add "AI edits to do after shooting" after the shoot listing:

| kind | Name | What to look for | Remake |
|---|---|---|---|
| face_swap | Face swap | Flicker at the face edge and hairline, mismatched skin tone/lighting between face and neck, features drifting in profile | R12 video-editing model for face replacement |
| lip_sync | Lip sync / redubbed lines | Mouth shape doesn't track the jaw, blurry teeth, only the mouth region has different image quality | R13 lip-sync redub |
| restyle | Full restyle repaint | The motion is a real person's, but the look is anime/oil-painting/clay/etc., with texture drifting between frames | R12 video-editing model |
| outfit_swap | AI outfit swap | Flickering clothing texture, blur at the arm boundary | R05 virtual try-on image + video |
| bg_replace | Background replacement via matting | Halo or jagged edges at the hairline, mismatched light direction between subject and background, wrong background perspective | Green-screen matting, or R12 |
| object_edit | Object added/removed/replaced | Flickering object edges, missing shadow | R12 |
| beautify | Beautify/face-slimming | Over-smoothed skin, straight background lines bending near the face | Beautify filters like CapCut |

## Quick judgment-point cheat sheet

| Observation | Points to |
|---|---|
| Pixel-identical adjacent frames, no change from start to end | Freeze frame; if it's a flat-color rendered card, classify as code render |
| The whole frame scales/translates under one rigid transform, no residual after compensation, and there is **no** 24fps-source frame-rate artifact | Still-image pan |
| Lots of flat color blocks, few colors, many horizontal/vertical edges | A render (code or 3D); background pixels completely frozen frame-to-frame with only local motion → 2D code-driven motion graphics |
| About 20% duplicate frames with a period of 4–6 (24→30) | Source is 24fps: AI-generated / 3D / movie or TV footage — not native phone capture |
| A 16:9-or-wider block embedded in black bars within a vertical frame, with a title above and commentary subtitles below | A commentary-video layout → leans film footage; AI cinematic shorts also add black bars, so check by eye |
| "Year · Show title" text on screen, a channel logo, a platform watermark, or several segments with different looks/grading rapidly cut together | Film montage / highlight-reel compilation → leans film footage |
| Only sharpness oscillates periodically, with no duplicate frames (frame interpolation) | Weak evidence: both frame interpolation and re-compression can cause this — confirm visually |
| Native ≥48fps with no duplicates and no interpolation period | Mostly native phone/camera footage |
| Noticeable sensor noise and temporal noise, handheld shake | Live action |
| A platform-transcoded file: the whole video is native 30fps, live-action-type shots have no duplicate frames | Leans native phone footage (noise evidence is unreliable after transcoding, go by frame rate) |
| A nearly colorless flat frame using only single-digit colors | UI/document motion graphics (code render), not 3D |
| On a near-black background, the bright areas are almost entirely tiny specks or thin lines with no continuous light/dark surface | Particle/point-cloud/wireframe graphics (code render), not a 3D solid, AI, or live-action night scene |
| Almost no fine texture anywhere in the frame: smooth color gradients, large saturated flat-color blocks, glow on a dark background, or a rainbow color field | A code-drawn gradient/vector/shader scene, not a 3D render (3D has cloth, wood-grain, and other material texture); still true even if the whole frame is animating |
| Music fills the whole video, the voice is singing (lots of sustained pitch) | Music MV → the song is very likely AI-generated |
| Narration + background BGM | Ordinary soundtrack → leans stock music library (unconfirmed — AI workflows often use Suno) |
| A jump in background noise level at cuts | Audio follows the clip: either native audio from an AI video, or live-action sync sound |
| The file carries a GB 45438 implicit AIGC marker (Label=1) | Declared as AI-generated (only tells you the whole video is declared as such — still judge each shot individually) |
| A light-colored panel with lots of small text | An embedded UI screenshot → real asset |
| A loud, tonal sound still present in the gaps between voice | There's BGM underneath |
| Large near-black areas with grain, uneven bright areas, a face present | Night/low-light live action (flat blocks from transcoding crushing shadows are not a render signature) |
| A face covering ≥6% of the frame, present the whole shot, talking | Live-action close-up of a person, or a digital avatar; flat blocks on a wall or clothing don't count as render evidence |

## Commonly-confused pairs (check each one by one during review)

**Live action vs. AI video**
- Live action: noise grain varies frame to frame; hands, text, and distant people look normal; physics is correct (cloth, liquid, collisions).
- AI: skin and lighting look too perfect; background people or text are garbled; object edges "boil"; motion drifts slowly; the shot often freezes near the end.
- Frame rate: a 24fps source (AI/film) vs. native 60fps (phone). Note that AI interpolated up to 60fps leaves a sharpness period, `sharp_period_ac>0.45`.
- The deterministic layer is weakest on this pair — **you must combine the VLM and your own frame-by-frame look.**

**Cartoon-3D-style AI video vs. a real 3D render**
- Real 3D: geometry and materials are frame-to-frame stable, hair and cloth move physically, can loop exactly, lots of flat blocks, few colors.
- AI: the same object deforms slightly within the shot, texture flickers, noise looks more photographic (>100 colors).
- Many "3D cartoon" accounts on Douyin are AI-generated, but without a watermark you can only state a leaning.

**Dark-background particle graphics vs. 3D render vs. AI video / night-scene live action**
- Particle graphics (code render): organs, bodies, the Earth, molecules, etc. made of thousands of glowing dots or thin lines with no continuous light/dark surface; often paired with a chapter card in the top-left, leader-line labels, ticking numbers. Doable in Three.js, AE Plexus/Trapcode — remake it as a code project.
- 3D render: has a continuous light/dark surface, materials, and shadows.
- Night-scene live action / AI: bright areas are objects and light spots with real light/dark gradation, with grain or deformation.
- When reviewing, the `description` must spell out what shape the particles form in each shot, their color, how they move (particle flow direction, deformation, push-in/orbit), and where labels and numbers are placed — the report's "code composite" per-shot table uses this directly. Writing only "particle background" will get you a blob of light spots when remade.

**Code-drawn gradients/glow/color fields vs. 3D render**
- Code-drawn: almost no fine texture anywhere in the frame, only smooth gradients, flat color blocks, glow, and text; common look: a gradient-blob background, frosted-glass cards, neon text, CSS 3D flip cards, fractal zooms, and plasma shaders. Still a code render even if the whole frame is animating (background drift, breathing glow).
- 3D render: objects have material texture (cloth, wood grain, metal, plastic grain), with shadows and light interplay between objects.
- A flat poster with film grain added: strip the grain and it's a flat color block — counts as code render. When the grain is heavy (tool-measured noise σ≥3), the tool may be fooled — judge by eye.
- A smooth gradient that shifts color slowly can be mistaken for a cross-dissolve; the tool filters these out by checking whether the rate of change is steady — if a continuous background animation still gets split into several shots on review, treat it as one shot.

**Code render vs. screen recording/screenshot**
- Code render: large text, brand colors, crisp vectors, elements animate in.
- Screenshot: light background, dense small text, UI controls (buttons, icons, navigation), often scaled or scrolled as a whole.
- A screenshot embedded in a template: the underlying footage still counts as code render, with the screenshot listed separately under "real assets."

**How to write `description` for a code-render shot**
- The report's "code composite" section has a per-shot "motion timeline" that's measured by the tool: each element's position, appearance time, direction, speed curve, and color value; elements appearing in sequence are merged into one "every X seconds" entry. The keyframe strip is `shots/shot_XXX_strip.jpg`, and the element screenshots/numbered collage is `shots/shot_XXX_elements.jpg`.
- Against these, write in `description` what each numbered element actually is ("1–12: rounded icon cards, popping in one after another") — no need to repeat the measured values.

**Still-image pan vs. a slow AI push-in**
- Pan: rendered frame-by-frame by editing software at the timeline frame rate, every frame different, no 24→30 duplicate-frame pattern.
- AI push-in: has the duplicate-frame pattern of a 24fps source, the subject often has subtle motion (breathing, blinking).

**2D animation vs. code-driven motion graphics vs. AI animation-style video**
- 2D animation: has characters and performance, stable lines, precise per-frame timing, often animated on twos (changes every other frame).
- Code-driven motion graphics: text, shapes, and UI entrances and eases, no character performance.
- AI animation-style: lines drift between frames, details deform, shows 24fps-source artifacts.

**Stop motion vs. AI video vs. live action**
- Stop motion: the same physical object moved frame by frame, lighting constant, slight flicker and jerkiness between frames, fingerprints and rough edges visible on the material.
- AI pseudo-stop-motion: the material itself changes between frames (clay texture drifting), with none of the real jerky rhythm.
- The tool often misjudges stop motion as AI (similar frame-rate pattern) — when reviewing by eye, focus on whether the material stays stable.

**Film/TV footage vs. AI short-drama vs. one's own live-action footage**
- Film/TV footage: recognizable actors or a show; professional-crew lighting (key light, rim light, practical lights in the set), depth of field, and multi-camera shot/reverse-shot; large numbers of extras, full sets and props, stunt doubles and wire-work in action scenes; on-screen text (signs, newspapers, screens) is clearly legible; often carries a channel logo, platform watermark, opening/closing credits, or is letterboxed in a wide aspect ratio.
- AI short-drama: faces are too perfect, different characters look alike; hands, teeth, and text come out wrong; few, blurry background people; object and clothing texture drifts between frames; shots are mostly 5–10-second single-person close-ups. Disclaimers like "dramatized" or "for the story" also show up in AI short-drama commentary and can't be used as evidence on their own.
- One's own live-action footage: talking-head, vlog, unboxing-type videos where the creator or client appears on camera, shot on a phone or by a small team, without crew-scale sets or extras.
- If a commentary video cuts in the creator's own talking-head or AI footage, label it shot by shot.
- When the tool only looks at pixels, it tends to misjudge film footage as "AI video" or "live action" and flag it "unconfirmed" — correct this on review.

**Game footage vs. 3D render vs. screen recording**
- Gameplay footage, game commentary, game montages: classify as `screen_capture` (it's a recording of a game rendering live on screen), not `cg_3d`. Look for HUD elements: health bars, minimap, crosshair, skill icons, damage numbers, chat box. To remake, record the game; write the game's name (if recognizable), viewpoint, and HUD position in `description`.
- Game CG trailers, cutscenes: no HUD, cinematic camera direction — classify as `cg_3d`.
- Both the tool and the VLM can misjudge game footage with a HUD as 3D or a screen-recorded UI — on review, go by the HUD.

**Motion comics / "comic dramas" vs. 2D animation vs. AI video**
- A static comic panel that's only panned/zoomed → `still_motion`; a panel with local animation layered on (blinking, hair blowing, mouth moving) → `anim_2d`; lines drifting between frames with deforming details → `ai_video`.
- Comic dramas are often an AI image plus local motion effects; set `image_source` to `ai_image` or hand-drawn based on details like line quality and fingers.

**Freeze frame: AI image vs. a real photo vs. a rendered card**
- A rendered card (flat color, few colors) → classify as code render.
- An AI image and a real photo can't be told apart at the deterministic layer — leave it to the VLM; if it still can't be determined, write "image asset (real/AI unconfirmed)."

## Cuts

- Hard cut: a spike in histogram distance and pixel difference, with the changed area exceeding 25%. An element popping in doesn't count as a cut.
- Jump cut / push-in cut: same camera position and background, the subject's position or framing jumps within one frame. The changed area is only above 17%, but more than 2.5x higher than surrounding frames; the frames right before and after are both stable, and the picture never reverts to what it was before the cut.
- Flash, muzzle flash, strobing: the frame brightens for a frame or two and then returns to normal — doesn't count as a cut; a white-flash transition that fades to reveal a new scene still counts as a cut.
- Short shots: a shot between two clean hard cuts with a stable picture is kept down to a minimum of 0.12 seconds (quick-cut inserts, an opening cover frame); boundaries closer together than that belong to the same transition — keep only the strongest one.
- Cross-dissolve: the middle frames are a linear blend of the two ends (α between 0.2–0.8, residual <0.2), with optical flow used to rule out a push/pan.
- Fade: brightness drops to near-black or near-white and back.
- A transition with no hard cut: counts as one boundary only when a stable segment is followed by a transitional segment and then another stable segment, the two ends show different footage, and the middle segment can be explained as a fade/dissolve/wipe/slide. A scene just getting brighter/darker, or an in-shot push-in/pan, doesn't count.
- Black bars and fixed top/bottom banners are cropped out before measuring, so a transition inside the black-barred picture isn't diluted by them.

**Transition types** (per-shot `transition`: `type`, start/end time `start`/`end`, `duration`, with `direction` when there is one)

| type | What it looks like | How to recognize it |
|---|---|---|
| `fade` | Through black/white (`via`) | A pure black or white flat frame in the middle, same scene darkening/brightening before and after |
| `flash` | White flash, a light-leak burnout revealing a new scene | The middle brightens to near-white but isn't a flat frame; or the first frame after the cut is very bright and then settles |
| `dissolve` | Cross-dissolve | Every middle frame is a linear blend of the two ends |
| `wipe` | Wipe (straight edge, circular, blinds, clock) | Checked block by block — each block is either the old or the new picture, and the new picture's area keeps growing; `direction` is which way the edge moves, `center_out`/`edges_in` for a circular open/close |
| `slide` | Slide (`mode`: `push` both move together, `cover` the new scene slides in, `reveal` the old scene slides away) | The middle frames can be reconstructed by translating the two end frames along one straight edge |
| `whip` | Whip pan: a fast pan with motion blur hiding the cut | Before and after a hard cut, the picture translates heavily in one direction with detail smeared out |
| `zoom` / `spin` | Zoom/spin into the next shot | A large per-frame estimated zoom amount or rotation angle around the hard cut |
| `blur` | Defocus and refocus | Detail on both sides of the hard cut is clearly lower than the shot's normal level |
| `glitch` | Glitch: misalignment, flickering between old and new | Multiple hard jumps within the span, with the picture jumping back to the old shot |
| `shake` | A burst of shake around the cut point | Large frame-to-frame displacement that cancels itself out |

Whip pans, zooms, spins, blur, and shake are only recognized around a hard cut: the same kind of motion without a hard cut is just in-shot camerawork.
- Motion-graphics beat: within one long shot, a "hold → animate → hold" pattern where the two holds differ by more than 12% is logged as an in-shot beat, not a cut.
