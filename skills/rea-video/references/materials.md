# The user's real assets

**Definition**: pixels in the frame that weren't generated or rendered by the production, but brought in from reality — live-action footage, real photos, web/app screenshots, screen recordings, a video clip playing inside a layout template, a pasted-on logo/product image. When remaking, the user must supply the original file for these — **never write "can be AI-generated."**

**Distinct from "reference material"**: a product, logo, or person appearing in AI video, digital-avatar, or 3D footage is made of model- or renderer-produced pixels, so it doesn't count as a real asset — it's logged under `material.references`, shown in the report as "reference material to prepare for the remake," and fed to the model as a reference image (`@imageN`) when remaking. A product that appears in live-action footage is a prop and is likewise logged as reference.

## How the tool flags it

- **Whole-frame**:
  - Live action → live-action video asset
  - Screen recording/screenshot → screen-recording asset or UI screenshot (a still one counts as a screenshot)
  - A freeze frame/pan whose source is a photo or screenshot → real photo or screenshot
  - A freeze frame with an unclear source (no VLM) → "image asset (real/AI unconfirmed)," not counted as a real asset
- **Embedded** (only detected inside rendered footage — i.e. code render, screen recording, 3D, a rendered card):
  - A photo-textured region: high grayscale entropy, no flat color, and not inside a subtitle band → photo/image asset, **flagged "real vs. AI unconfirmed"** (something pasted into a template can also be an AI image). Either get the VLM to confirm it, or judge by eye yourself.
  - A light-colored panel with ≥6 small-text spots in a single frame → UI screenshot (counted as a real asset directly)
  - A region with motion while its surroundings stay still → an embedded video clip or screen recording
- **VLM**: entries under `embedded_assets` with `is_real_material=true`. When the underlying footage is a rendered layout (code, screen recording, still image), it counts as an embedded asset; something the VLM calls `real_footage` is logged as an "embedded video clip" (it could itself be an AI video). When the underlying footage is AI video / digital avatar / live action, or a product/logo/person inside 3D footage, it's logged as reference material instead.

## Known misses

- A dark, semi-transparent photo used as an ambient background (e.g. mediank's NASA background image): texture contrast is too low for deterministic detection to catch. When you see a dark-themed template, take a look at the background yourself.
- A dark-theme web screenshot: it's not a light-colored panel, so it won't be detected — check by eye for a browser frame or UI chrome.
- A small logo or product image overlaid on live-action or AI footage: the deterministic layer doesn't run embedded-asset detection on a photographic base image (too many false positives) — this relies on the VLM.

## Stock-footage filler shots

Aerial city shots, timelapses, landscapes, stock business-people, hand close-ups — live-action filler unrelated to the video's main subject — are often bought from a stock site, not shot by the creator. The tool flags these as "live-action video asset." When reviewing, note "likely stock filler" in `description`, and list it separately from the creator's own footage when delivering: for the remake, just find the same kind of clip on a stock site — there's no need to ask the user for the original file.

The same goes for film/TV footage (`film_footage`): the asset belongs to the studio — when delivering, write "need to find and cut in the matching source clip; commercial use needs licensing" rather than treating it as the user's own asset.

## How to write it up for delivery

List it item by item: shot number, time, asset type, position in frame, content description, and "you need to supply this for the remake: ..."
