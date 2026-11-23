# capsule-cinema Integration

`remake-run` and `capsule-draft` run `rea_video/integrations/run_remake.py` and `to_capsule.py` inside **capsule-cinema's own Python environment** (importing capsule-cinema's `lib/` and `scripts/` and reading its `.env`). The interpreter is resolved in order: `--python` > `$CAPSULE_CINEMA_PYTHON` > `<capsule-root>/.venv/bin/python`. `rea-video` itself only depends on `numpy`, `opencv`, `pillow`, and `pyyaml`.

```bash
CC=~/code/github/capsule-cinema          # capsule-cinema repository

# 1) Decompose (VLM recommended; reuses capsule-cinema's Gemini config)
rea-video --env-file $CC/.env decompose ref.mp4 -o out/ref --vlm

# 2a) Generate a capsule draft (no changes to capsule-cinema code; injects rea-video breakdown via tool_factory)
rea-video capsule-draft out/ref/breakdown.json \
    --capsule-root $CC --capsule-name my_ref_remake            # Add --write-capsule to persist to capsules/

# 2b) Remake: plan by intent & model → build jobs → run capsule-cinema → stitch onto original timeline + side-by-side comparison
rea-video plan out/ref --model seedance-2.5               # Reads out/ref/remake/intent.yaml; final prompts go in remake/prompts.json
rea-video remake-plan out/ref/breakdown.json --capsule-root $CC
rea-video remake-run out/ref/remake/jobs.json --dry-run   # Preview jobs first
rea-video remake-run out/ref/remake/jobs.json             # Execute paid model calls
rea-video assemble out/ref/breakdown.json out/ref/remake/results.json
```

- **Generated shots:** One job per call per generation unit using the planned input mode (reference original clip / one original keyframe per shot / first & last frames / text-only) and final prompts from `remake/prompts.json`; aborted if plan validation finds errors.
- **Ken Burns & still frames:** No model calls; `assemble` rebuilds them locally from measured zoom/pan parameters.
- **Code render & screen capture shots:** Kept as original (`keep_original`) by default; rebuild in Remotion using the text tracks and layouts in the report.
- **Partial runs:** `remake-run ... --only G2` (generation unit), or `--only 2,5` / `--only shot_002` (units containing those shots). Exits non-zero if any shot fails to generate, with failure details in `results.json`.
- **Custom capsule-cinema path:** If not at `~/code/github/capsule-cinema`, set once via `rea-video prefs capsule_root <path>` or pass `--capsule-root` on every command (must match between `remake-plan` and `remake-run`, otherwise `jobs.json` refuses to run).
