# Doing it yourself

The user shoots and edits it themselves, or generates it manually on another platform (Jimeng, Kling, etc. web apps), following the report's shooting plan and shot list — you cut the footage back into the original timeline.

## Cost

- rea-video itself is free.
- Manual generation on another platform is billed by that platform; the user supplies their own gear and location for live-action shoots.

## When the user wants to do it themselves: hook them up

1. Give them report part two's "follow these steps" first (see `remake.md`'s "Follow these steps") along with the list of material to prepare (what to shoot, how long each segment, where the generation-unit prompts are).
2. Once the user has shot or generated it, they just need to send you the file paths — you handle the rest: matching it to the right shot, writing `mine.json`, running `rea-video remake`, showing the user `remake/compare.mp4`, and telling them which shots and audio are still outstanding per `remake/status.md`.

## Cutting it back into the original timeline

1. Match the footage to shots per the shot list (the shot numbers and durations in `report.md`). When one long piece of footage covers several shots, cut it by action with ffmpeg first; if you're not sure which part matches which shot, show the user a screenshot to confirm.
2. Write it into `out/ref/remake/mine.json`: `{"results": {"<shot id>": {"video": "<absolute path to the footage>"}, ...}}`. When a manually generated "generation unit" video covers several shots, write it as `{"G2": {"unit": "G2", "shots": [4, 5, 6], "video": "<path>", "trim_to": <unit duration in seconds>}}` — `remake` splits it back into individual shots per the original cut points.
3. `rea-video remake out/ref`: fits each piece of footage into its original shot's duration by matching its action to that shot's rhythm (the action's start/end and peak land on the original's timing; sped up by at most 3x, slowed by at most 1.25x, with anything beyond that trimmed; footage with little action is uniformly retimed across its whole span); cut points are aligned to the original's frame numbers so drift doesn't accumulate across the timeline; any shot not yet filled in stays as an original-footage placeholder, and audio carries over from the original. Produces `remake/remake.mp4`, a side-by-side `remake/compare.mp4`, and `remake/status.md`. Tell the user which shots are still original-footage placeholders and which audio still needs replacing. When you only want to assemble without redoing code shots, use `rea-video assemble out/ref/breakdown.json <path to results.json>`.
   - Overlays are layered back on (`remake` does this by default; `assemble` needs `--overlays`): the measured subtitles/styled text/titles are layered onto the replaced shots in their original style (an original-footage-placeholder shot already has its own text, so it isn't layered twice). The font is a generic sans-serif, and color/background-box values are measurements — they may differ from the original. Stickers are only listed under `stickers_to_add` in `assemble_log.json` — you'll need to find and paste in that asset yourself.
