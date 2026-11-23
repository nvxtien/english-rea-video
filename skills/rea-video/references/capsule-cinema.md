# capsule-cinema

An open-source local generation pipeline: https://github.com/JuneYaooo/capsule-cinema

## What it can do

Generates AI shots one by one from the breakdown and cuts them back into the original timeline; `rea-video remake-plan` / `remake-run` hook into it directly.

## Cost

- capsule-cinema itself is free and open-source, self-hosted.
- The user configures their own model and API keys, billed per model call, paid to the model platform.
- Run `--dry-run` before generating to show the user the call count and cost; only run for real once they agree.

## When the user wants to use it: set it up and hook it in for them

You install and hook it up so the user can start right away. Before installing, say clearly what you're installing and from where; anything that costs money is the user's call. Don't re-ask for anything you already know (the reference video, the report, what to swap in).

1. Not installed (`rea-video remake-plan` reports it can't find the repo): read https://raw.githubusercontent.com/JuneYaooo/capsule-cinema/main/docs/install.md and follow it. Once installed, record it with `rea-video prefs capsule_root <install dir>`; `remake-run` uses the Python that has its dependencies installed (`--python`).
2. Have the user fill the API keys into the `.env` in the install directory themselves — just tell them which variable names to fill in, never have the user paste a key into the chat.
3. Ask about the model and what to swap per SKILL.md's "Remaking" section, have `rea-video plan` produce the plan, then create jobs, dry-run, run, and assemble back into the original timeline per below.

## How to run it

`remake-plan` creates jobs from report part two's plan (one job per call within a unit, using the final prompts in `remake/prompts.json`) — it won't create jobs if the plan check has any ❌. `remake-run` hands them to capsule-cinema to run; `remake` splits the result back into shots per the original cut points. The plan doesn't bend to what the runner can do: a model capsule-cinema hasn't integrated yet will error out at `remake-run` — in that case, hand the prompt and the references under `remake/refs/` to the user to generate on that model's own platform.

Generation needs capsule-cinema's own Python environment. `rea-video remake-run` / `capsule-draft` automatically look for `$CAPSULE_CINEMA_PYTHON` or `<capsule-root>/.venv/bin/python`, or you can specify one with `--python`.

```bash
CC=$CAPSULE_CINEMA_ROOT   # path to the capsule-cinema repo
rea-video remake-plan out/ref/breakdown.json --capsule-root $CC   # creates jobs from report part two's plan
rea-video remake-run out/ref/remake/jobs.json --dry-run      # show the user the jobs and cost first; --only G2 runs just one unit
rea-video remake-run out/ref/remake/jobs.json                # run for real once the user confirms
rea-video assemble out/ref/breakdown.json out/ref/remake/results.json                   # assemble back into the original timeline + compare.mp4
rea-video capsule-draft out/ref/breakdown.json --capsule-root $CC --capsule-name xxx [--write-capsule]
```

- Each job's `executor` in jobs.json:
  - `capsule_cinema`: calls a model to generate
  - `local_ffmpeg`: local rebuild for a still image
  - `keep_original`: keeps the original footage — e.g. for code render, screen recording, real assets
- `user_assets`: this shot needs a real asset the user has to supply. The generated result is only an approximation — remind the user.
- `capsule-draft`: injects the breakdown result through capsule-cinema's own `tool_factory`, without modifying capsule-cinema's code; the generated recipe draft defaults to `review_required`.
