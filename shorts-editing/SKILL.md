---
name: shorts-editing
description: Turn product images and a voiceover into a vertical Shorts video that does not look like a static slideshow, using ffmpeg.
---
# Skill 1 — Shorts Editing Skill

Turn product images + a voiceover into a vertical Shorts video that
does not look like a static slideshow. This is the exact v2 pipeline
we used for our 4th Shorts (Classroom Games Mega Pack, 28.9 s,
published November 2026). v1 was a static slideshow with one push-in
and no subtitles; it looked cheap. v2 fixed that.

## What you need

- `ffmpeg` + `ffprobe` installed.
- Product preview images (portrait-ish, at least 640 px wide; we used
  real renders of the product pages, never mock content).
- A narration MP3 (we used a CLI text-to-speech tool, voice "Aria";
  any TTS works). Write the script first — the video is cut to the
  audio, not the other way round.
- Optional but recommended: the `ffmpeg-skill` package by kajisho5
  (MIT) for its probe → edit → verify flow. It is a separate tool,
  not part of this pack.

## The v2 recipe (six ingredients, all required)

1. **Hook card, first 1.5 seconds.** A full-screen card: dark green
   background (`#0E3B2E`), thin gold frame, big cream serif text, and
   a small gold brand line under it. The hook is a problem or a
   number, never the product name. Real examples we used:
   - "TEACHERS: STOP MAKING GAMES FROM SCRATCH"
   - "Stop buying Christmas games one at a time."
   - "3 games your Christmas party actually needs."
   The hook overlays the first 1.5 s of the video and then disappears.
2. **Burned-in subtitles for every segment.** The spoken line is
   drawn on screen in 42 pt white text with a dark box, so a muted
   viewer still gets the message. One or two lines per segment.
3. **Crossfades between segments.** 0.4 s `xfade` (fade) between
   every segment. No hard cuts.
4. **Rotating camera moves.** Four moves cycle in order, one per
   segment: zoom in (6%), zoom out (6%), pan right, pan left. Two
   segments in a row must never use the same move.
5. **Progress bar.** A gold bar, 8 px tall, along the bottom edge,
   growing left to right with playback time.
6. **Adaptive title size.** The big title on each segment shrinks
   automatically until it fits the frame (start 66 pt, shrink by 2
   until it fits 980 px wide, floor 30 pt). Never ship a cut-off
   title.

## Technical spec (the exact numbers we shipped)

- Canvas: **1080×1920, 30 fps, H.264, yuv420p**.
- Segment length = narration duration ÷ number of images. Our
  videos ran 16–29 s with 4–7 images.
- Foreground image: scaled to 1020×1320, centered on a blurred,
  darkened copy of itself filling the frame (blur sigma 32,
  brightness −0.35). Title drawn at y=110, subtitles at y≈1505–1560.
- Encode: `libx264, preset medium, crf 20; aac 128k; +faststart`
  (faststart so the file plays while still downloading).
- Audio map index = number of image inputs + 1 if a hook card is
  present. Getting this wrong mutes the video or maps the wrong
  stream — check it before every render.

## The ffmpeg flow: probe → cut → verify

Run it in this order, every time. Do not skip the verify step.

### Step 1 — Probe

```bash
ffprobe -v error -show_entries format=duration -of csv=p=0 narration.mp3
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,r_frame_rate,codec_name -of default=noprint_wrappers=1 image-01.png
```

Write down: narration seconds, image sizes, frame rate. Segment
length = narration ÷ image count. If any image is smaller than
640 px wide, replace it — upscaling a tiny image looks worse than
using fewer images.

### Step 2 — Cut

Build the filter graph in this order per segment: scale + crop the
background, scale + zoompan the foreground, overlay, draw the title,
draw the subtitles. Then chain the segments with `xfade`, overlay
the hook card for `between(t,0,1.5)`, and draw the progress bar.

> **The one pitfall that cost us a day — read this twice.**
> In `zoompan`, the `d` parameter means "how many output frames to
> generate per input frame". It must be **`d=1`**. If you set `d` to
> the frame count (a natural-looking mistake), every segment blows
> up to hundreds of seconds long. If your render comes out absurdly
> long, `d` is the first thing to check.

Camera-move formulas (zoom centred, pan at 7% zoom — copy these):

- Zoom in: `z='1+0.06*on/FRAMES'`
- Zoom out: `z='1.06-0.06*on/FRAMES'`
- Pan right: `z='1.07':x='(iw-iw/zoom)*on/FRAMES'`
- Pan left: `z='1.07':x='(iw-iw/zoom)*(1-on/FRAMES)'`

with `x`/`y` centred (`iw/2-(iw/zoom/2)`, `ih/2-(ih/zoom/2)`),
`setsar=1`, output size = foreground size, `fps=30`.

### Step 3 — Verify (before you upload anything)

- `ffprobe` the output: duration matches narration ±0.5 s, resolution
  is exactly 1080×1920, codec h264, audio present.
- Watch it once, muted: every title fits, every subtitle is readable,
  the hook is legible in the first 1.5 s.
- Watch it once with sound: narration and cuts line up, no segment
  is silent for no reason.
- Check the file plays from the first second (`+faststart` was set).
- If anything fails, fix the script/graph — never ship a "good
  enough" render. A bad video costs more in channel trust than a
  missed day.

## What our video experiments proved (November 2026)

We ran five visual experiments on a spare cloud machine before
committing to a style. The verdicts — use them to avoid our dead
ends:

| Experiment | Verdict | Why |
|---|---|---|
| **A — Gradient drift.** Slow pink-purple gradient drifting behind the product, soft shadow. | **Adopt.** | Reads as motion at a glance, not a slideshow. Weakness: drift amplitude was small and the colours say nothing about the product's season. |
| **B — Real B-roll background.** A real flowing light-spot video behind the product card, text on top. | **Adopt — best of the batch, our daily default.** | Real motion, clear text, product stands out. Weakness: background theme is generic; text on black felt slightly stiff. |
| **C — Dynamic typography.** Hook words pop in one at a time, one every 0.4 s. | **Adopt after fix.** | Clearest hook delivery of all five. First version used 80 pt type and words overlapped — fixed to 64 pt. Weakness: hard cuts feel like slides, second half goes static, product ends up small. |
| **D — Montage.** Three scenes crossfaded, xfade transitions. | **Do not adopt yet.** | Transitions were smooth, but the product only appears at 7 s, the second scene's vertical crop left only a face close-up, and the three scenes had no story line. Needs a rewrite, not a tweak. |
| **E — Cinematic push-in + handheld shake.** 12% push-in, 0.012 rad shake. | **Do not adopt.** | The movement was too subtle — six sampled frames looked almost identical, i.e. a static image. Either increase the amplitude a lot or drop the idea. |

Practical rule from the experiments: **motion must be visible in a
6-frame contact sheet.** If you sample six frames across the video
and they look the same, the viewer will read it as a still image.
Use real moving backgrounds (B) or obvious typography motion (C);
do not rely on tiny push-ins (E).

Free stock for B-roll: Mixkit and Coverr offer free clips with no
registration and no key. Download the clip, check it actually shows
what its title claims before you build around it — two of our
downloads did not match their titles on visual inspection.

Next experiments on our list (not yet run): speed curves and rhythm
cuts, word-by-word highlighted subtitles, a product 3D tilt /
page-flip mockup, and free music / sound-effect design.

## Publishing rules (YouTube, verified November 2026)

These are not editing choices — break them and the channel gets
penalised regardless of how good the video is.

- **At most 1 Short per day, at least 24 hours apart.** We posted
  3 same-template Shorts within 10 minutes on day one and hit
  YouTube's inauthentic-content policy (in force since 2025-07-15:
  it penalises highly similar mass-produced template content such as
  image slideshows). Rotate the hook and the format every time.
- **Rotate three formats:** (a) product tour, (b) pain-point
  solved (problem → solution → price), (c) countdown list
  (Top 3 / 5 reasons). Never ship two videos in a row with the
  same hook, same format and same structure.
- **Links in a Short's description and comments are NOT clickable**
  (since August 2023, including pinned comments). Do not say "link
  in the description" in the voiceover — it points nowhere. The
  funnel that works: Shorts → channel homepage Shop link, and
  Shorts → related long video (needs Advanced feature verification:
  a 6-second selfie video, ID, or channel history, in YouTube
  Studio → Settings → Channel → Feature eligibility). End every
  voiceover with: "Shop link on my channel page."
- **Length:** 20–45 s for Shorts. One long video (2–5 min, full
  product demo) per main product, with the buy link in its
  description — that link IS clickable once Advanced verification
  is done.
- **Description top line (copy this):** "Get this pack: tap my
  channel name above, then tap the Shop link on my profile."
- **Review weekly:** views, subscriber conversion, and orders that
  followed each video. Three straight videos at 0 views → change
  the title/thumbnail/topic direction. A product that sells after a
  video → make two more videos in the same style within a week.

## Worked example (real, November 2026)

- Video #4, Classroom Games Mega Pack: 28.9 s, 7 product preview
  images, pain-point format, hook "TEACHERS: STOP MAKING GAMES
  FROM SCRATCH". Built with the v2 recipe above, frame-checked
  before scheduling.
- Videos #1–#3 (16–19.7 s, product-tour format) were built with
  the same recipe minus the hook-card polish; they are live and
  public. The first three were posted too close together — do not
  copy that part.
- One long video (1 min 22 s, full demo of the Christmas Mega
  Pack, 1920×1080) was built with a horizontal variant of the same
  pipeline: concatenate clips, pad to 1920×1080, draw the subtitle
  bar at the bottom.
