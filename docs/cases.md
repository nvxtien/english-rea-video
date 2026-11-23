# Case Studies

All 7 reports below were generated with this version: after tool measurements, an AI assistant visually reviewed every shot (without calling a vision model API) to produce the report and breakdown package. In each storyboard grid, each cell is one shot with a color bar showing production method, confidence, and timestamp; each full report includes key takeaways, narrative arc, shot-by-shot breakdown table, remake steps, plus per-shot evidence and prompts.

The sample videos in `docs/cases/videos/` were re-compressed to keep the repository small, which strips platform metadata (Douyin / TikTok / WeChat) and adds another compression pass. Running the tools directly on these samples will differ slightly from the published reports—especially in tools-only mode without visual review. To reproduce exact results, use original platform downloads.

## 1. Original vs. AI Remake: TikTok Live-Action → Product Swap

20 s · Original 14 shots / Remake 21 s 8 shots

![Shot-by-shot comparison of original vs AI remake](cases/juicer_pair.jpg)

▶ [Original TikTok](cases/videos/v11_tiktok_original.mp4) · [AI Remake](cases/videos/v12_oiioii_juicer_remake.mp4) · Full reports: [Original](cases/v11/report.md) · [Remake](cases/v12/report.md)

| | Original | AI Remake |
|---|---|---|
| How it was made | 100% first-person handheld phone footage, human voiceover | 100% AI-generated |
| How to remake | 1 set, 2 camera positions | 8 shots batched into 3 Seedance 2.0 generations |
| Assets needed | Physical product | Product reference images (clear bottle logo) |

- **Original:** Opens with a hook statement ("This bottle is a GAME CHANGER"), then demonstrates step-by-step until drinking. Hands, countertop bottles, and background remain consistent across every shot with natural handheld shake.
- **Remake:** Adding fruit, closing lid, blending, pouring—one shot per step. The brand text on the cup is garbled in every shot; whole fruit turns into a smoothie in half a second with no blending process; the glass changes shape three times in two seconds—classic generative artifacts.
- **Generation batching:** 8 shots grouped into 3 generations using reference images to lock product appearance. The report specifies exact inputs, sources, and bilingual prompts for each batch.

## 2. Mixed Media: AI Dialogue + Code UI + Real Screenshots

19 s · 10 shots

![Mixed media storyboard](cases/dual_role/storyboard.jpg)

▶ [Original Video](cases/videos/dual_role.mp4) · [Full Report](cases/dual_role/report.md)

- **How it was made:** 59% code / template render + 41% AI-generated video. Two AI characters have a Q&A dialogue (5 shots across two night rooms), intercut with 5 dark-gray UI template segments featuring diagnostic reports, note illustrations, and GitHub screenshots, each slowly zooming in. No background music or post-production SFX.
- **How to remake:** Batch the 5 dialogue shots into 4 generations using two character reference images; build the 5 UI template segments in a single code project and swap in your own screenshots.

## 3. 3D Explainer: Why Salted Duck Eggs Leak Oil

66 s · 29 shots

![3D explainer storyboard](cases/salted_egg/storyboard.jpg)

▶ [Original Video](cases/videos/salted_egg.mp4) · [Full Report](cases/salted_egg/report.md)

- **How it was made:** All 29 shots are 3D-rendered. A plush monkey host holds retention while a cross-section duck egg model explains the mechanism in 4 steps—turning every invisible chemical process into a visible visual metaphor. The same cross-section model reappears across 8 shots with consistent fur and material shaders across frames. Narration + stock music + 6 post-production SFX synced to animation beats (salt crystals popping, oil droplets falling).
- **Two ways to remake:**
  - **Video model route:** Batch 29 shots into 14 generations.
  - **Code route (Blender / Three.js):** Have an LLM model the character and cross-section once, set up camera angles per shot, and render—giving the best multi-angle consistency and effortless copy edits.

## 4. Code UI Motion Graphics: Product Promo

70 s · 9 shots

![Code UI promo storyboard](cases/v10/storyboard.jpg)

▶ [Original Video](cases/videos/v10_oiioii_gen_coding_first.mp4) · [Full Report](cases/v10/report.md)

- **How it was made:** Promo for OiiOii's "Video Remake" feature—100% code-driven UI animation with no voiceover, demonstrating the workflow ("Upload viral hit → Rewrite in one sentence → Batch render"). AI characters and products shown on screen are static images embedded inside UI cards, not full-screen AI video. One SFX lands when particles disperse at the end.
- **How to remake:** No video model needed. Build one Remotion project where text and UI are data-driven; the report provides a shot-by-shot motion timeline ready for an LLM to code.

## 5. AI Music Video: Historical Figure in a Recording Studio

164 s · 118 shots

![AI music video storyboard (first 24 shots)](cases/qing_mv/preview.jpg)

*(Image shows the first 24 shots from an earlier split)* ▶ [Original Video](cases/videos/qing_mv.mp4) · [Full Report (118 shots)](cases/qing_mv/report.md)

- **How it was made:** 53% AI-generated video + 32% lip-sync avatar + 13% Ken Burns still zoom/pan + 3% code-rendered titles. Zeng Guofan wears headphones in a recording studio singing his life story, with each lyric paired with historical scenes. Opening credits state lyrics by Claude and music by Suno. Five shots are reused clips appearing twice—only needing to be generated once.
- **How to remake:**
  - Generate the song first in Suno matching style and tempo.
  - Drive the studio singing shots using a portrait + vocal track via a lip-sync avatar model.
  - Batch the remaining AI shots into 33 generations.
  - Render Ken Burns pan/zoom shots locally from still images—no video model needed.

*(Note: The automated Audio section misclassified the singing as TTS + stock music because visual review only inspects frames; classifying AI singing requires an audio-capable model.)*

## 6. Live-Action Product Demo: ASMR Unboxing

71 s · 69 shots

![Unboxing storyboard](cases/unboxing/storyboard.jpg)

▶ [Original Video](cases/videos/unboxing.mp4) *(Douyin @阿咪🍓, 720p re-encode)* · [Full Report](cases/unboxing/report.md)

- **How it was made:** 99% live action. A phone mounted overhead on a top-down rig films a pair of hands unboxing 7 cream-toned home items on a rug; no spoken narration, relying on crisp unboxing foley and short floating captions to keep an average pace of 1 shot per second. Unboxing sounds are live production audio rather than added SFX.
- **How to remake:** No AI model needed. One phone on an overhead rig, products arranged on a light-colored rug, hands entering from the bottom edge; add upbeat stock BGM in post and place small captions beside each item.
