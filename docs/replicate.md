# Remaking From the Report

The "Remake It Yourself" section of the report begins by classifying the video format (talking-head, tabletop / unboxing, vlog, live-action drama, AI short drama, AI product ad, AI music video, avatar talking-head, AI short clips, 3D / animation, movie recap / mashup, motion-graphics explainer, screen-capture tutorial, Ken Burns still-image video, beat-synced montage, or dance / action performance), then lists production steps in the order suited to that format. The format only determines which steps exist and their order; the details of every step come from measurements of the original video:

- **Filming:** Measured shot sizes, camera positions, lighting (key direction, contrast ratio, softness, color temp), color grade, and inferred lighting setup.
- **Generation:** Number of generation units, which model to call and how many times, plus first-frame and reference images required for each unit.
- **Editing:** Shot count, average seconds per cut, jump cuts and punch-in cuts, non-hard transitions.
- **Packaging:** Subtitle line count, position, color, entrance animation, and the exported SRT timeline.
- **Audio:** Voice and music source, BPM, key, and SFX count.
- **Final assembly:** Placing all recreated shots back onto the original timeline and rendering a side-by-side comparison video.

If an aspect wasn't present in the video, that step is omitted. Many videos mix multiple methods; whenever a secondary method makes up at least a quarter of the runtime, its workflow is included as well. Below is how each visual type is remade before the agent stitches everything back onto the original timeline.

## Live-Action Shots: Film Yourself

1. The report outputs a shooting schedule: total number of locations/sets, camera setups per set, shooting order, framing, handheld vs. tripod/gimbal, key light direction, fill needs, and gear list. Shots from the same set are grouped together even if they appear separated in the final edit.
2. After filming by set, send the clips to the agent and say: *"Cut my filmed clips into the storyboard timeline."*
3. The agent trims each clip to its matching shot slot and duration, places it onto the original timeline, and exports a side-by-side comparison video so you can check pacing. Unfilmed shots stay filled with original footage as placeholders, so you can shoot and review one set at a time.

## AI-Generated Shots: Use Video Models

1. The report outputs generation units: consecutive shots in the same scene are batched into one generation, specifying the model, required reference images (product shots, character sheets), multi-shot prompts (in Chinese and English), and how each generation chains into the next.
2. Two execution paths:
   - **Manual:** Paste each unit's prompt and reference images into web tools like Jimeng or Kling, then pass the generated clips to the agent to assemble onto the original timeline.
   - **Automated:** With [capsule-cinema](https://github.com/JuneYaooo/capsule-cinema) installed, say *"Remake the AI shots using capsule-cinema"*. The agent calls the video models, stitches the results back in, and outputs a comparison video. Because model calls cost money, it runs a free dry run first and waits for your confirmation before generating.
3. Automated generation currently calls the model once per shot. Multi-shot prompts in the report are already written to save cost, though batch multi-shot generation currently requires the manual path.

## Dance & Action Shots: Drive From the Original Video

Dance, combat, and fitness shots rely on large full-body motion. Product demos, talking-heads, or singing/speaking shots with small gestures do not fall into this category—they follow normal generation units with lip-sync or avatar tools. Generating from start/end keyframes alone forces the model to guess mid-action motion, losing choreography and beat sync, so the report separates them into an "Action Drive" track instead of normal generation units.

1. Have the agent run `rea-video drive <report_dir>`: removes subtitles and stickers, then splits the video at motion pauses near beat hits into ~10-second driving clips.
2. Prepare one character turnaround sheet (same outfit: front, side, back full-body plus face close-up) to use across all clips.
3. In most cases, feed the driving clip directly as reference without depth maps or clay renders: use Kling Motion Control for single-person, full-body, continuous shots; Seedance 2.5 Omni-Reference for multi-person, camera movement, or mid-clip cuts; or Wan Animate for local runs. The report recommends the best option and includes ready-to-use prompts:
   - **Cluttered original background** (streets, crowds, props) or completely different target scene: convert to a depth video (`--depth` or Video Depth Anything) so only motion, space, and camera move remain.
   - **Changing space/camera angles** or reusing the same choreography: convert motion capture to a clay/base mesh in Blender, re-render with new camera angles, and use that as generation reference.
   - **Facial expression is key:** keep direct video reference. Depth, skeleton, and clay renders only transfer body pose and lose facial expressions.
   - **Different body proportions** (chibi, animal, cartoon): use skeleton video (DWPose + Wan VACE).
4. Send the generated clips back to the agent; it runs `rea-video drive --join` to stitch them onto the original timeline and exports a side-by-side comparison against the original music.

## Code / UI Motion Graphics: Write Code

The report provides exact text, layout, and animation timing per segment. Tell the agent *"Build a Remotion project from this report"*—afterward, changing copy only requires editing data. The tool does not auto-generate the Remotion project yet; the agent writes it.

## Ken Burns (Still Pan/Zoom): Local Generation

Given one still image and the measured zoom/pan speeds, the agent generates the clip locally without calling any video model.

## Audio: Build Three Separate Tracks

- **Voiceover:** Script is included in the report; generate with TTS or record yourself.
- **Music:** Search stock libraries by the measured BPM, key, and style, or generate with Suno.
- **SFX:** Place sound effects according to the timestamp table.
