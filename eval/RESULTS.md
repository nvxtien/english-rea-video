# Evaluation Results

Configuration: **Deterministic forensics + OCR only, VLM disabled** (VLM strengthens live/AI/3D separation, real photo vs. AI image distinction, and subtle SFX). Reproduction commands are at the bottom.

## 1. Base Visual Production Method (8 classes, duration-weighted)

| Dataset | Videos | Accuracy | Cut F1 | Independence |
|---|---|---|---|---|
| `labels.yaml` (tuning set) | 15 | **98.3%** | 1.00 (2 ground-truth clips, incl. 14 × 0.25s cross-dissolves) | Rules designed on this set |
| `holdout.yaml` | 9 | **99.6%** | — | 2 initial errors led to frame-rate rules; no longer strictly independent |
| `holdout2.yaml` | 7 | **98.9%** | — | Run after production-method rules were finalized |

Tuning-set confusion matrix (seconds): live action 16.0/16.0; AI video 141.2/141.2; 3D 66.5/66.5; code render 155.9 correct, 2.4 misclassified as 3D; screen capture 19.0/19.0; still hold 42.8 correct, 5.3 misclassified as code render.

Decoded from scratch (no cached features); with cached features scores are 99.9% / 99.6% / 99.8% due to a decoding-stage change not reflected in old caches. Splits are by clip source rather than creator, so holdout numbers reflect performance on similar-style samples.

## 2. User Real Assets (whether a shot contains live footage / screenshot / screen capture / photo, duration-weighted)

| Dataset | Accuracy | Notes |
|---|---|---|
| labels | 97.6% | Single miss: dark semi-transparent NASA background photo in `mediank` (known limitation) |
| holdout | 99.6% | 2 initial false positives (subtitles on light code UI; 3D fur); fixed by rule adjustments |
| holdout2 | 97.2% | **Initial run was 75.6%** (subtitle band counted as photo/video; embedded images defaulted to "real"). Remaining miss: dark-theme webpage screenshot |

Rule updates after round 1:
- Count text per single frame rather than accumulating across timestamps.
- Exclude subtitle bands from photo/video asset detection.
- Mark embedded photo/video regions as "real vs. AI to be confirmed"; only UI screenshots and screen recordings count directly as confirmed real assets.
- Only detect UI screenshot panels inside 3D shots.

## 3. Audio

| Metric | Result | Ground Truth Source |
|---|---|---|
| Voice present | 5/5 | `capsule-cinema` `tts.py`, talking-head clips |
| BGM present | 4/4 | `bgm.py`, `bgm_fit.json` |
| BPM (±4%, incl. half/double time) | 2/2 (110.0 / 112.3 vs. 110.15) | `bgm_fit.json` |
| Musical key | 2/2 (D major) | `bgm_fit.json` |
| SFX present (≥3 events) | 2/4 | `cues.py`, asset manifests |
| Music source | 2/5 (see below) | Suno logs, stock library download records |

**SFX is the weak spot of deterministic audio:** On `salted_egg`'s voice + BGM mix (110 ground-truth cues), signal features alone cannot separate SFX from speech/music onsets (~0.2 precision). Deterministic analysis now only flags prominent non-vocal transients as low-confidence candidates; **full SFX detection relies on an audio-capable VLM (Gemini)**.

**Music source** combines video-format priors + audio evidence (rules in `skills/rea-video/references/audio.md`), without VLM or platform metadata:

| Video | Format | Ground Truth | Prediction |
|---|---|---|---|
| `qing_hiphop_mv` | AI rap MV (dry vocal chorus repeat 0.26) | Suno v6 | AI-generated 52% ✓ |
| `mediank_pure_data` | Pure instrumental | Stock (`manten diloty`) | Stock library 60% ✓ |
| `monkey_wine_v3` / `salted_egg` / `birdnest` | TTS narration + BGM (chorus repeat 0.00) | Suno-generated | Stock library 52% ✗ |

The 3 misses reflect a **prior limitation**: AI-workflow explainers often use Suno instrumental BGM, which is signal-indistinguishable from stock instrumentals. Correcting those requires platform music tags (`--meta`) or audio fingerprinting, so ordinary BGM defaults to "likely stock, to be confirmed".

**Vocal separation (`stems/`):** On `salted_egg` TTS mixed over Suno BGM (+6 dB vocal), separation achieves 19.7 dB SDR for both voice and music, with −42.5 dB BGM leakage into the vocal stem. Dry-stem vocal pitch continuity is 0.24 on rap MV vs. 0.44 on full mix, so singing vs. speech is always measured on the dry vocal stem.

## 4. Platform-Recompressed Videos (10 OiiOii article clips, dev set)

`eval/articles.yaml`: WeChat-transcoded 30fps downloads. Transcode-handling rules were tuned on this set.

| Video | Ground Truth | Before Rules | After Rules |
|---|---|---|---|
| v11 TikTok product demo | Live action | AI 100% (0.0) | Live action 100%, flagged "confirm live vs AI" (1.0) |
| v09 Gemini launch | 2D UI motion | Code 63% + 3D 37% (0.63) | Code 81% + 3D 19% (0.81) |
| v10 OiiOii product promo | Code UI motion | Code 78% + 3D 17% (0.83) | Code 92% + 3D 4% (0.96) |
| v04 promo | 3D + kinetic text | 3D 65% + AI 35% (0.65) | Unchanged (0.65) |
| Remaining 6 AI / split-screen clips | — | All correct / 0.8 | Unchanged |
| **Average** | | **0.79** | **0.92** |

Three transcode rules:
1. Detect platform transcoding (`Tencent CAPD MTS` / `Wxmm_` / `vid:v…`); disable noise absence as an AI signal and ignore GOP-induced sharpness periodicity.
2. Whole-video frame-rate signal: in 30fps files, ≥70% duplicate-free live-like duration favors phone capture over 24fps AI sources.
3. Nearly achromatic flat frames using only a handful of colors classify as UI/document rather than 3D.

### Text-only Agent Review (`dsh`, DeepSeek V4.1 Flash, no vision)

Across 11 ground-truth clips: tools-only got 5 full / 5 partial / 1 wrong; `dsh` got 6 full / 2 partial / 3 wrong, overturning tools 4 times (2 right, 2 wrong). `SKILL.md` now forbids non-vision agents from overturning tool production-method calls.

## 5. After Visual Review (production method, duration-weighted)

| | `holdout` (9) | `holdout2` (7) | Transcoded (10, dev) |
|---|---|---|---|
| Tools only | 99.6% | 99.8% | 88.6% |
| + Vision model (Gemini) | 99.6% | 99.8% | 99.6% |
| + Agent visual review | 99.6% | 99.8% | 100% |
| Both enabled | 99.6% | 99.8% | 98.8% |

Uses cached features (hence 99.8% for `holdout2` tools-only vs. 98.9% decoded from scratch). Blind agent review followed `eval/agent_test/JOB.md`.

## 6. First-Run Unseen Samples

| Dataset | Tools Only | + Vision Model | + Agent Visual Review |
|---|---|---|---|
| Douyin product demos (2 clips: unboxing + AI/live tutorial), `daihuo.yaml` | 46.4% | 63.9% | 97.5% |
| — Real assets on above | 62.4% | 86.8% | 97.5% |
| Douyin talking-head (2 clips: night street + indoor close-up), `koubo.yaml` | 75.5% (51% / 99%) | 88.5% | 96.7% |

Cut detection across 9 frame-annotated clips (talking-head, unboxing, movie mashups, phone demos): **95% recall, 94% precision**.

### Agent Visual Review Only (no VLM API)

When no VLM is configured, the agent inspects per-shot contact sheets alongside tool evidence and `SKILL.md` rules and may override the tool's top-2 choices (provided no 24→30fps duplicate-frame cadence contradicts it):

| Dataset | Tools Only | Agent Review Only |
|---|---|---|
| Product demos (2) | 46.4% | 98.6% |
| Film/TV mashups (9) | 15.5% | 98.8% |
| Talking-head (2) | 75.5% | 96.8% |
| `holdout` (9) | 99.6% | 100% |
| `holdout2` (7) | 99.8% | 99.8% |
| Article videos (13, blind round 2) | 88.6% | 100% |

## 7. Error Distribution by Resolution & Camera Motion (Tools Only)

Aggregated across 9 datasets (excluding `film.yaml`, which is 15% tools-only because film clips require visual recognition):

| Resolution | Duration | Accuracy | Misclassified Shots | Main Confusions |
|---|---|---|---|---|
| 1080p | 913s | 90.5% | 40/283 | Live→3D 21.5s; Code→3D 18.0s; Live→Code 15.4s |
| 720p | 607s | 97.6% | 3/145 | Code→Still motion 6.0s; 3D→Still motion 6.0s |
| ≤480p | 94s | 100% | 0/15 | — |
| >1080p | 16s | 100% | 0/1 | — |

| Camera Motion | Duration | Accuracy | Misclassified Shots | Main Confusions |
|---|---|---|---|---|
| Static | 727s | 91.6% | 26/184 | Live→3D 18.6s; Live→Code 15.4s; Code→3D 15.2s |
| Smooth camera move | 550s | 98.0% | 8/142 | 3D→AI 5.0s; Live→AI 3.8s |
| Static (moving subject) | 251s | 97.8% | 4/98 | Code→3D 5.1s |
| Handheld shake | 63s | 90.5% | 2/12 | 3D→AI 5.4s |
| Full-frame pan/zoom | 38s | 53.1% | 3/7 | 3D→Still motion 12.0s; Code→Still motion 6.0s |

## 8. Full From-Scratch Re-Eval (Tools Only, No Feature Cache)

| Dataset | Accuracy |
|---|---|
| `labels.yaml` (11 missing locally) | 100% |
| `holdout.yaml` (3 missing) | 99.5% |
| `holdout2.yaml` (4 missing) | 100% |
| `articles.yaml` (transcoded) | 97.2% |
| `daihuo.yaml` | 63.2% |
| `koubo.yaml` | 93.3% |
| `film.yaml` | 15.8% |
| `code_videos.yaml` | 94.8% |
| `code_videos_holdout.yaml` | 100% |
| `particles.yaml` (3 clips) | 66.7% (bokeh sample missed) |

## Known Limitations

1. Only 3 live-action eval clips (all 60fps Douyin phone footage); no real Ken Burns clips (tested on synthetic clips only).
2. Dark/semi-transparent background photos and dark-theme webpage screenshots can be missed; product/logo overlays on live or AI shots require VLM.
3. Deterministic layer cannot tell whether an embedded photo is real or AI-generated.
4. Combined VLM + agent scores are re-fused from cached answers rather than freshly decoded features.
5. Fine dark-background graphics (particles, point clouds, thin lines): works with soft background halos, but large out-of-focus bokeh orbs on dark backgrounds are not yet recognized.
6. Music source: AI-generated instrumental BGM in regular videos cannot be distinguished from stock music by signal alone.

## Reproduce

```bash
export CAPSULE_CINEMA_ROOT=~/code/github/capsule-cinema
rea-video eval eval/labels.yaml   -o out/eval_main
rea-video eval eval/holdout.yaml  -o out/eval_hold
rea-video eval eval/holdout2.yaml -o out/eval_hold2
rea-video eval eval/articles.yaml -o out/eval_articles
rea-video eval eval/audio_mv.yaml -o out/eval_audio_mv --stems
# With VLM:
rea-video --env-file $CAPSULE_CINEMA_ROOT/.env eval eval/labels.yaml -o out/eval_main_vlm --vlm
```
