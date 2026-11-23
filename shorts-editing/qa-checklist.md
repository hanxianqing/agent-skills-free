# Shorts QA checklist

Run this before every upload. One failure = do not upload; fix it
and re-check. (Checklist order follows the probe → cut → verify
flow in SKILL.md.)

## Probe

- [ ] Narration duration measured with ffprobe: ______ s
- [ ] Image count = caption count = subtitle chunk count
- [ ] Every image ≥ 640 px wide; no image is a stretched thumbnail
- [ ] Seconds per image = narration ÷ image count: ______ s

## Cut

- [ ] `zoompan` uses `d=1` (the pitfall: `d` = frame count blows
      each segment up to hundreds of seconds)
- [ ] Camera moves cycle zoom in → zoom out → pan right → pan left,
      no two adjacent segments alike
- [ ] Hook card present, first 1.5 s, text fits inside the gold frame
- [ ] Audio map index = image count + 1 (hook present) — verify the
      narration is actually on the output, not silence
- [ ] Progress bar drawn, gold, bottom edge, grows with time

## Verify (watch it twice)

- [ ] Output is exactly 1080×1920, H.264, yuv420p, audio present
- [ ] Duration matches narration ±0.5 s
- [ ] Muted watch: every title fits (no cut-off), every subtitle
      readable, hook legible in the first 1.5 s
- [ ] Sound watch: narration lines up with cuts, no dead silence
- [ ] File plays from the first second (`+faststart` set)
- [ ] 6-frame contact sheet test: sample six frames across the
      video — if they look almost identical, the motion is too
      subtle; add B-roll background or typography motion (see the
      experiments table in SKILL.md)

## Publish

- [ ] At least 24 hours since the last Short went live
- [ ] Hook and format differ from the previous video
- [ ] Title: product name + core benefit + pages/price hook
- [ ] Description top line: "Get this pack: tap my channel name
      above, then tap the Shop link on my profile."
- [ ] Voiceover ends with "Shop link on my channel page."
      (never "link in the description")
- [ ] Language: English; category: Howto & Style; audience: Not
      made for kids; visibility: Public
- [ ] After upload: confirm it is live, note the URL and the time;
      check views after 24 hours and log the result
