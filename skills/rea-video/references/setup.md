# Installation and VLM setup

## Installation (one-time)

```bash
pip install "rea-video[ocr] @ git+https://github.com/JuneYaooo/rea-video"   # needs ffmpeg; Python ≥ 3.10
# or inside the repo: bash scripts/install_skill.sh (installs the CLI and links the skill into agent directories)
rea-video --help
```

If pip reports `externally-managed-environment` (Homebrew / newer Ubuntu system Python), don't add `--break-system-packages` — use
`pipx install "rea-video[ocr] @ git+https://github.com/JuneYaooo/rea-video"` instead, or just run `install_skill.sh` (it automatically switches to pipx or an in-repo `.venv`).

If it reports no matching wheel for `onnxruntime` (e.g. "no wheels with a matching Python ABI tag"), the default Python is too new and the dependency hasn't published a build for it yet: switch to a 3.10–3.14 interpreter — `pipx install --python python3.12 "rea-video[ocr] @ git+https://github.com/JuneYaooo/rea-video"` or `PYTHON_BIN=python3.12 bash scripts/install_skill.sh`. `install_skill.sh`'s auto-picked interpreter already prefers 3.10–3.14. The full `[ocr]` extra installs on any of 3.10–3.14: 3.12 and below use `rapidocr-onnxruntime`, 3.13+ automatically switches to `rapidocr` 3.

If you installed `[ocr]` manually on Python 3.12 or below, its OCR dependency also installs a second `opencv-python`, which fights with rea-video's `opencv-python-headless` over the same `cv2` directory. After installing, run:
`pip uninstall -y opencv-python && pip install --force-reinstall --no-deps "opencv-python-headless>=4.8,<5"` (`install_skill.sh` does this automatically).

## Local speech recognition (optional)

When there are no burned-in subtitles and no audio-capable VLM configured, where does dialogue come from? Install the optional speech-recognition extra:

```bash
pip install "rea-video[ocr,asr] @ git+https://github.com/JuneYaooo/rea-video"   # if already installed, just pip install sherpa-onnx
```

Once installed, `decompose` automatically runs recognition on the separated voice track (SenseVoice, supports Chinese/English/Japanese/Korean/Cantonese; downloads a ~160 MB model to `$REA_VIDEO_MODELS` the first time, default `~/.cache/rea-video/models`) —
a few seconds for a one-minute clip. The result goes into `breakdown.json`'s `speech.lines` (each line has a start/end time); the report's "script" table and each shot's `said` field in `review_tasks.json` use it.
When the dialogue is in Bengali, add `--asr-lang bn` (or set it as the default with `rea-video prefs asr_lang bn`): SenseVoice doesn't recognize Bengali and will transcribe it as garbled Chinese/Japanese/Korean;
`bn` switches to a Bengali streaming recognition model (another ~90 MB download), `speech.language` becomes `bn`, and recognized sentences come without punctuation.
The same optional package can also separate speakers (another ~35 MB model download): multi-person dialogue then gets tagged with who said it, and a separate audio track is output per person — see `audio.md`.

This is machine transcription — it will have homophone typos and mishear proper nouns: when there are on-screen subtitles, go by the subtitles; when the VLM gives dialogue, the report uses the VLM's version. When quoting dialogue to the user, note "speech-recognized, unproofread."
Within a long sentence, per-line timing is estimated by character count (`timing: estimated`); the start/end of the whole segment is actually measured. If it isn't installed, `speech.reason` is `not_installed` and everything else runs as normal.

## VLM setup (recommended, not required)

**VLM (recommended, not required)**: a VLM is a multimodal model that can see images/video (Gemini, GPT-4o, Doubao, Qwen-VL, etc.). Without one, the tool judges purely from pixel statistics, frame rate, OCR, and metadata — it still runs end-to-end and produces a report, but **the live-action / AI / 3D distinction gets noticeably weaker**, and the sound-effects list is limited to low-confidence signal-level candidates. With a VLM on, the tool sends a short clip and keyframes per shot to it, then fuses the result with the deterministic findings.

Configuration goes through an OpenAI-compatible endpoint, read in this order:

1. `REA_VLM_BASE_URL` / `REA_VLM_API_KEY` / `REA_VLM_MODEL` (`REA_VLM_MODEL` can list several models comma-separated — it falls through to the next one if one fails)
2. If none of those are set, falls back to capsule-cinema's `VIDEO_ANALYSIS_*` or `GEMINI3_*`

Load config with `--env-file <some .env>` (e.g. reusing capsule-cinema's). **Never put a key into the chat, the repo, or the report, and never open the .env file to read it.**

When the user hasn't configured a VLM: just run without `--vlm` as usual — don't stop the task because of this.

- **You yourself can see images (a multimodal agent)**: you are this run's "VLM" — there's no need to suggest the user configure one separately. Follow the "Visual review and merge" step in SKILL.md section 1: `rea-video review` → look at images shot by shot and fill in answers → `rea-video apply-review` — your judgment merges into the tool's conclusions and report. State in the verdict: "VLM off, reviewed visually by the agent."
- **You can't see images**: follow the "rules when you can't see images" in `review.md`, and add one line at the end of your reply suggesting VLM setup (note that it improves the live-action/AI/3D distinction and sound-effect detection).
- Measured (repo `eval/RESULTS.md`): an agent's visual review merged in is more accurate than Gemini VLM alone; Gemini tends to misjudge cartoon-3D-style AI video as a 3D render, and a game UI shown picture-in-picture as a screen recording. So **always do the visual review pass when you can see images**, even with VLM on, and judge independently — don't just copy the VLM's answer.
- An external VLM still does things you can't substitute for: it can watch a video clip's actual motion and listen to the audio track directly. When you can only see still frames and can't hear the audio, still mark the voice/sound-effect source "unconfirmed."

**Whether the VLM actually kicked in is decided by the `VLM:` line in `rea-video summary`** (also in `breakdown.json`'s `analysis.vlm.status`): `ok` means it ran, `partial` means some shots got a result, `failed` means it was called but every call failed (e.g. a 503 from the server), `not_configured` means it was called but no config was found, `off` means it wasn't called at all. `failed` / `not_configured` both count as the VLM being off — state this honestly in the report; never write "VLM was on." When there are many shots, this line may say "only sent a/n shots": by default at most 60 calls are made, and shots that weren't sent rely on deterministic evidence alone — rerun with `--max-vlm-shots N` if you need full coverage, or do a visual review pass on those shots yourself.
