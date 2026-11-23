# Decomposition Methodology: How Shot Production Methods Are Classified

Goal: for every shot, determine the underlying visual production method, overlay layers, evidence, and confidence, then assign a remake route.

## 1. Taxonomy

| Label | Meaning | Remake Route (`rea_video/capabilities/routing.yaml`) |
|---|---|---|
| `live_action` | Live-action camera footage | R07: AI image-to-video approximation, or re-film |
| `ai_video` | AI-generated video | R07: First/last keyframes + prompt, generated via capsule-cinema |
| `avatar` | Digital human / talking avatar | R03: Voice cloning + avatar |
| `cg_3d` | 3D rendering | R08: 3D modeling & render, or AI keyframe approximation |
| `code_render` | Code / template render (motion graphics, text, UI, charts, cards) | R01: Remotion / HTML rebuild |
| `screen_capture` | Screen recording / screenshot | R02: Real UI capture |
| `still_motion` | Still image + pan / zoom / tilt (Ken Burns) | R17: Single image + measured motion parameters, exact local rebuild |
| `static_hold` | Frozen still frame | R18: Single image |

Overlays are tracked separately: subtitles, titles, text callouts, graphic stickers, and layout templates. A single shot can be "AI video + code subtitles".

## 2. Where Evidence Comes From

Deterministic forensics handle "measurable" signals while VLMs handle "semantic" visual understanding, with a clear division of labor:

| Question | Method | Reliability |
|---|---|---|
| Where are the cuts | HSV histogram distance & pixel difference with local adaptive thresholding; changed area must be large enough so element pop-ins aren't counted as cuts. Jump cuts / punch-in cuts: single-frame spike on the same camera setup surrounded by stable frames; 1–2 frame flashes/fire bursts are excluded. Cross-dissolves: checks whether middle frames are linear blends of endpoints while using optical flow to exclude camera push/pull. Transitions: measures start/end span per boundary and classifies fade / flash-white / dissolve / wipe / push / whip-pan / zoom / spin / blur / glitch / shake (with direction for wipes & pushes) | High (F1=1.0 on two ground-truth clips, including one with 14 × 0.25s dissolves) |
| Is the frame moving | Ratio of pixel-exact duplicate frames + first-to-last frame diff | High |
| Is it a single still panning/zooming | LK optical flow fitted to a similarity transform (RANSAC): high inlier ratio, low residual, no local motion, near-zero residual after motion compensation | High |
| Rendered vs. photographed texture | Flat 8×8 block ratio, color count covering 95% of pixels, horizontal/vertical straight-edge ratio, Immerkær noise | High (separates renders from photos) |
| 2D compositing / code motion | Background pixels stay identical frame-to-frame while only local regions move | High |
| Native frame rate | Duplicate frame cadence (~20% on 24→30fps, period 4–6); periodic sharpness oscillation from frame interpolation (weak evidence alone since re-encoding can mimic it); native ≥48fps with zero duplicates | Medium-High: generative models output 24/25fps; phones often shoot 60fps |
| Live action vs. AI vs. 3D | Sensor & temporal noise, handheld shake, frame-rate fingerprint, plus VLM artifact checks (hands, text, physics, texture flicker) | Medium deterministic-only; High with VLM |
| Metadata | Capture device make/model; GB 45438 implicit AIGC watermarks (Label 1/2/3 & provider codes); C2PA; editor/renderer signatures | Very strong when present; often stripped by platforms |
| Overlays | Edge clusters where pixels freeze over moving background + OCR text tracks (position, temporal change, horizontality) | Medium-High |
| Audio | Onsets / beats, cut-on-onset ratio, speech modulation features | Medium |

Jump-cut segments come from the same source clip (same camera, same background); when VLM is enabled, they are combined into a single clip query (max 12s, max 8 segments): production method and filming style are answered once for the whole group, while per-segment descriptions and remake prompts are returned individually. When VLM call budgets are capped, the most uncertain and longest shots are prioritized. On-screen text per shot is pulled from the full-video OCR pass; only when full-video OCR is too sparse (long videos) or the packaging layer is disabled does it sample 1–4 frames per shot.

## 2.1 User Real Assets

Source materials fall into four types: AI-generated, producer-rendered, user real assets, and mixed.

- **Full-frame:** Inferred from the base visual label. Live action = live footage; screen capture = UI recording/screenshot; static hold sourced from a photo or screenshot = real asset.
- **Embedded** (detected only inside rendered shots):
  - ≥6 small-font text regions inside a light, low-saturation panel in a single frame → UI screenshot (confirmed real asset)
  - High grayscale entropy without flat fills → photo or video clip (real vs. AI **to be confirmed**)
  - Subtitle bands are excluded
- In the remake plan, all real assets are listed under `user_assets` and must be supplied by the user rather than generated by models.

## 2.2 Audio

HPSS (median filtering separating harmonic and percussive components) on STFT:

- **Voice:** 3–8 Hz syllable modulation in the speech band plus inter-sentence pauses; fundamental frequency hints male/female. Multi-speaker tracks are diarized by timbre on the dry vocal stem (`stems/voice_S1.wav`, etc.).
- **Music:** Harmonic music bed remaining active during vocal gaps or clear beat periodicity; onset autocorrelation estimates BPM, Krumhansl profiles estimate key, and chroma novelty detects section changes.
- **Editing:** Ratio of cuts landing on beats or onsets.
- **SFX:** Deterministic stage flags prominent non-vocal transients as low-confidence candidates; the full list is identified by an audio-capable VLM (see `eval/RESULTS.md`).
- **Outputs:** `audio` (raw measurements), `audio_plan` (merged VLM + deterministic audio tracks & remake plan), `audio.png` (timeline).

## 3. Fusion

Final probability $\propto P_{\text{det}}^{0.6 \times w_{\text{conf}}} \times P_{\text{VLM}}$, plus two hard constraints:

- A frozen shot cannot be classified as any "video generation" category.
- A rigid full-frame pan/zoom shot cannot be classified as live action or avatar.

Confidence weights: `static_hold` 1.6, `still_motion` 1.4, `code_render` 1.0; `live_action`, `ai_video`, and `cg_3d` are only 0.6 so VLM/visual review carries the primary decision among those three.

When visual sources (VLM or agent review) participate, classification runs in two stages: first classify the production group (Generated: `ai_video` + `avatar`; Rendered: `cg_3d` + `code_render`; `live_action`; `screen_capture`; Still: `still_motion` + `static_hold`), then pick the fine-grained label within that group using the group's total probability as confidence. That way, if tools say `ai_video`, agent says `avatar`, and VLM says `live_action`, the first two votes combine under "Generated" instead of splitting and losing to `live_action`. In tools-only mode, the top fine-grained label is used directly (since tool `avatar` scores are carved out of `ai_video`, merging them would unfairly suppress `live_action`—measured drop from 0.51 to 0.33 on talking-head evals).

Majority rule: when the tool's top choice agrees with either visual source (same label or same group), the final decision must stay within that group.

Other safeguards:
- When tools and VLM disagree on live-action vs. non-live-action without agent review, the shot is flagged "disputed" and added to the mandatory checklist in `review_tasks.json` (VLMs can hallucinate nonexistent evidence).
- Frame rate only proves a 24fps source: when labeled `ai_video` with no AIGC watermark while visual review leans live action, it is flagged "AI vs. 24p cinema camera to be confirmed" with confidence capped at 0.7.
- OCR text absent from VLM's overlay text list (or moving along with an object) is treated as in-scene text rather than a post-production overlay.

## 4. Outputs

```
out/<name>/
  breakdown.json   Video IR (rea_video.breakdown.v1): per-shot features, evidence, probabilities, overlays, remake plan
  report.html/md   Human-readable report (HTML rendered from Markdown): Verdict → Content & Structure (summary, hook, structure timeline, script) →
                   Cast, Settings & Props → Audio (separate cards for Voice / BGM / SFX / Ambience) → Cinematography, Lighting & Editing →
                   Editing & Packaging Layer → Remake Plan (including real assets & reference assets) → Shot Table → Uncertainties →
                   Appendix (collapsible per-shot forensics, metadata). Cast/settings/props/script come from VLM full-video analysis or agent visual review.
  storyboard.jpg   Shot overview grid (color-coded by production method)
  shots/           Per-shot contact sheets, first frame, last frame (keyframes double as remake inputs)
  clips/, vlm_cache/  VLM input clips and cache (--vlm only)
```

`finalize()` can re-classify from saved features and VLM answers without re-decoding video; `rea-video eval` uses this automatically after rule updates.
