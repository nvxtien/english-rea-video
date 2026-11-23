# HyperFrames

[HyperFrames](https://github.com/heygen-com/hyperframes) is an open-source HTML video framework: an AI coding assistant writes HTML/CSS animation, which it renders to MP4. Code-render, motion-graphics, and illustrated-animation shots were made this way in the first place, so handing them to it to redo as an editable project gets closer to the original than rea-video's own `code-remake` draft.

## Cost

- The framework itself is free and open-source (Apache-2.0), rendered locally.
- Its Studio app has a separate paid tier (hooks into HeyGen's generation models) — not needed just to redo code-type shots.

## How to install it

Needs Node.js 22+. In Claude Code:

```bash
claude plugin marketplace add heygen-com/hyperframes
claude plugin install hyperframes@hyperframes
```

For other coding assistants (Codex, Cursor, etc.), run `npx hyperframes skills update` once. After installing, follow its skill to have the assistant write the video.

## What to give it

Hand these to the assistant writing the HyperFrames project, telling it "redo this with the same timing and visuals, don't use the original's pixels":

1. `video/rhythm.md`: the time of every cut and transition, the transition type and duration, and each shot's keyframes (time + what kind it is: motion start/landing, a turn, an element entering/exiting, a composition change...). The `@number` after each timestamp is the original frame number — order by frame number (HyperFrames uses the same frame rate), not by a rounded-off second; lay out the animation's keyframes by this, and leave the in-between to easing.
2. Each shot's keyframe sheet `shots/shot_XXX.jpg`, and the report's "code composite" section: layout, color scheme (color values), what each element is, and how it enters.
3. Audio: use `stems/music.wav` as the soundtrack, or swap in a different track matching the BPM and mood in `video/sound.md`; sound-effect timestamps are in `video/sound.md`.
4. Subtitles and styled text: `packaging/timeline.json` (time, position, style).

If you only want to redo some of the shots, spell out the shot numbers and time range; the rest can stay as original-footage placeholders — write the finished segments into `remake/mine.json` and cut them back into the original timeline with `rea-video remake out/ref` (see `diy.md`).
