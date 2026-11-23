# Visual-review rules

Supplement to the SKILL.md section 1 workflow: how a visual-review result gets fused with the tool, and a few situations that commonly get misjudged.

## Fusion rules

From measurement (repo `eval/RESULTS.md`):

- When there's only one visual-review source (VLM only, or you only): a visual-review result can only be chosen from the tool's top two candidates. When the tool is confident about "AI / live action / digital avatar," don't override it; a transcoded shot with 24→30 duplicate frames won't be changed to live action just because it "looks real." If the shot itself has no duplicate frames (native 30fps rhythm), your live-action call is adopted even if the whole video wasn't flagged as native 30fps because of an AI-generated intro mixed in; when the tool's top three candidates are all below 0.4 and the category you picked is only slightly behind the runner-up, your call wins too. Visual review is most useful for distinguishing between 3D / code-driven motion graphics / screen recording. 2D animation, stop motion, and film footage are exceptions: the tool can't detect these three at all, so when you call one of them it's adopted directly — but if only one side has looked, the verdict stays "unconfirmed."
- When both the VLM and you have answered: decide the broad category first (generated / rendered / live action / screen recording / still image), then the fine-grained one. If the tool's top choice is backed by either of you (same fine category, or same broad category — e.g. the tool says AI video, you say digital avatar), go with the tool's top choice at the broad-category level; when you two agree against the tool, fuse normally.
- Always look at shots flagged "disagreement" (the tool and the VLM disagree on whether it's live action): check whether the evidence the VLM cites is actually visible in the screenshot — the VLM does fabricate reasons.
- A shot where `review_tasks.json` notes "the user has corrected the tool's X to Y n times before" is a type the user has repeatedly corrected — look at it closely.

## Rules when you can't see images (important)

First confirm whether you can actually see images at all (can you read `storyboard.jpg`, `shots/*.jpg` and describe what's in them). **If you can't see images and there's no VLM configured:**

- **Don't override the tool's production-method verdict** (live action / AI / 3D / code / screen recording). Pixel statistics (color count, flat-block ratio, noise) are exactly the evidence the tool already used — running the same analysis again in your head will only get you the same conclusion or a worse one. Measured: an agent that can't see images misjudged an AI-generated pencil sketch as a "white-background layout card," and a glass-material 3D element as a "code UI."
- What you can and should do instead: check the timeline and cut points, read the OCR text track to catch misreads, use the levels in `stems/` to judge whether voice is present, read the metadata (AIGC marker, platform transcoding flag), and list the tool's low-confidence shots under "open questions."
- In the one-line verdict, state plainly "VLM off, footage not reviewed visually," and leave any shot the tool flagged "live action or AI unconfirmed" exactly as unconfirmed.

## Original/reference footage: write the counter-evidence before calling it AI

A single continuous shot with a documentary or handmade feel (stop motion, an overhead everyday-life shot, a handcrafted miniature scene), with no AIGC marker and no 24→30 duplicate frames, is something the tool often misjudges as AI. If, after looking at it, you can't find a clear AI tell (deformation, texture flicker, broken physics, garbled text), **don't assign high confidence**: write it as "AI vs. live action / stop motion unconfirmed (leaning X)," list the evidence on both sides, and cap confidence at 0.7.

## Commentary videos, montages: first decide whether it's cut-in film/TV footage

When the picture is letterboxed in black bars, the tool notes "film-footage clues" in the evidence and flags the shot "AI video or film footage unconfirmed" or "live action or film footage unconfirmed." The tool can't tell who shot it — this step needs your eyes:

- Recognizable actors or a show, crew-scale lighting and sets, extras, a channel logo or platform watermark, clearly legible on-screen text → `film_footage`.
- Faces too perfect, different characters look alike, hands and text come out wrong, texture drifting → `ai_video` (an AI short-drama commentary uses the exact same layout as a real one — don't judge by layout alone).
- If the creator's own talking-head, or an inserted screenshot or AI footage, is cut in, label it shot by shot.
- `live_action` is only for footage the creator or client shot themselves.

## Platform-transcoded video (files downloaded from WeChat / Douyin / TikTok)

When the metadata contains `Tencent CAPD MTS`, `Wxmm_`, `vid:v…`, the tool flags `video_context.transcoded`. Re-compression wipes out sensor noise and introduces a fake interpolation rhythm, so:

- Noise-based evidence stops working — "no noise" no longer means AI; the tool will flag these shots "live action or AI unconfirmed."
- Evidence that's still reliable: 24→30 duplicate frames (generative models mostly output 24fps; the whole video being native 30fps with no duplicate frames looks more like a direct phone export), an implicit AIGC marker, and the actual content (real hands, a real environment, whether physics looks natural — this one requires looking at the picture or using a VLM).
- Priority when evidence conflicts: **a regular 24→30 duplicate-frame pattern plus an implicit AIGC marker** outweighs "I looked and didn't spot an AI tell" — don't use the latter to override the former into live action; at most write "leaning AI, live action unconfirmed." Conversely, only call it live action when the whole video is native 30fps, has no duplicate frames, no AIGC marker, and the footage looks normal on inspection.
- A file that's been re-compressed with the platform marker stripped (a download tool removed the metadata, or a sample clip from the repo's `docs/cases/videos/`) won't be flagged as transcoded, but still carries the same fake interpolation rhythm. The tool flags a shot "AI vs. live action unconfirmed" only from "sharpness oscillating periodically" with no duplicate frames — your visual-review verdict can override this one directly.
