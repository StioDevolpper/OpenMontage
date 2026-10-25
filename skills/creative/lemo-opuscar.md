# Lemo-Opuscar — film styles drawn in code (creative skill)

> Library: `~/Projects277/lemo-opuscar` (external clone, MIT code / CC BY 4.0 demos) · Picture: Canvas/WebGL page rendered frame by frame in headless Chrome · Sound, mix, subtitles: OpenMontage
> Validated on this machine with `projects/spring-bean-aquarelle/` (20 s, Watercolor Brush, cloned FR voice). Illustrated write-up (diagrams, frames, the two ffmpeg 4.4 mixer fixes): `docs/lemo-opuscar-integration.md`.

**What it is:** a library of 59 film styles (watercolor, ink wash, Ukiyo-e, risograph, blueprint, Swiss motion graphics, LiDAR survey, 16-bit pixel RPG…). Each style has a `STYLE.md` (the fixed look, colour, type, motion, camera grammar, sound palette) and a demo built on a reusable engine (brush strokes, rigs, shaders). No image or video generation: every frame is code. It is a **picture engine + style library, not a pipeline**: OpenMontage keeps the pipeline, the voice, the music, the mix and the subtitles.

## When to propose it

Offer it as one option at the idea / proposal stage (beside Remotion, HyperFrames, Ink Theater) when the brief wants:

- a strong **hand-made or print look** that the 5 playbooks in `styles/*.yaml` don't cover (watercolor journal, ink wash, woodcut, risograph, paper-cut, stained glass…);
- a **free, offline** visual with no image/video provider;
- a short hero piece (15–90 s) where the look matters more than speed.

Don't propose it for talking heads, footage edits (unless one of its Edit & Documentary styles fits), or long videos on a deadline: each film needs its own hand-written page. Say the cost up front: more tokens and iteration than a templated Remotion scene; render ≈ 2–3 min per 20 s on this machine.

Style list with gallery names: `~/Projects277/lemo-opuscar/styles/README.md` · gallery with every demo film: https://lemomo-ai.github.io/lemo-opuscar/ . Suggest 2–3 styles that fit the topic and let the user pick.

## Setup (once)

```sh
git clone --depth 1 https://github.com/lemomo-ai/lemo-opuscar.git ~/Projects277/lemo-opuscar   # if missing
cd ~/Projects277/lemo-opuscar && sh plugin/skills/lemo-opuscar/scripts/setup.sh deps          # npm + headless Chrome + .venv, ~350 MB
```

The install runs third-party code: ask the user before running it. Voice and music tiers of lemo-opuscar are not needed (OpenMontage does both).

## Workflow (division of labour)

Work in `projects/<slug>/`, never inside the lemo-opuscar clone (don't edit its `core/` or `styles/`).

1. **Script + voice (OpenMontage).** Write the lines, generate them with the usual TTS (for the user's French videos: `~/tools/my_voice.sh`, see voice notes). Measure each WAV's duration: it sets the timeline.
2. **Read the style.** `styles/<slug>/STYLE.md` is fixed: keep its look, palette, type, motion and pitfalls. Story, shots and timings are ours — write the treatment first, then open `DEMO.md` / `demo/` only to learn the engine, never to copy the demo's story.
3. **Build the page** in `projects/<slug>/page/`: copy the engine files the style needs from `styles/<slug>/demo/` (e.g. watercolor: `engine.js`, run `paper.py` for `paper.jpg`, fetch fonts into `fonts/`), then write `index.html` + `main.js` exposing the page contract:
   - `window.DUR`, `window.render(t)` (pure function of t, seeded randomness only), `window.READY = true` after fonts/images load;
   - `window.EV = [{t, type, …}]` for sound cues; subtitles burned into the page in the style's caption design (STYLE.md §4).
   Put the voice start times in one place and reuse them in the page and the mix script.
4. **Check stills** before rendering, from the lemo-opuscar root:
   `node core/render/still.mjs <page> 3 8.5 12.5` then `.venv/bin/python core/render/sheet.py sheet.jpg <stills…>` and look at the sheet (overlapping labels, notes covering the subject, unfinished effects).
5. **Render the silent picture:** `node core/render/events.mjs <page>` then `node core/render/video.mjs <page> --fps 24 --workers 2` → `<page>/out/video.mp4`.
6. **Music + foley:** synthesize from `events.json` with `core/audio/sfx.py` helpers (see `projects/spring-bean-aquarelle/score.py`), or use any OpenMontage music source. Keep to the style's sound palette.
7. **Mix (OpenMontage):** `audio_mixer` `full_mix` — speech tracks with `start_seconds`, music (`volume` ≈ 0.35, `music_volume_during_speech` ≈ 0.2), sfx, `target_duration` = `DUR`.
8. **Mux:** `ffmpeg -i page/out/video.mp4 -i audio/mix.wav -map 0:v -map 1:a -c:v copy -c:a aac -b:a 192k -ac 2 -shortest -movflags +faststart <slug>.mp4`. Subtitles file: `core/render/srt.py cues.json <slug>.srt`.
9. **Self-check:** `ebur128` loudness, `blackdetect`, Whisper on the final file, one full watch.

## Machine notes (this box)

- **2 cores:** `--workers 2`.
- **No GPU:** WebGL falls back to llvmpipe. 2D canvas styles render fine; 3D / shader-heavy styles (brick-toy, glass-product, paper-popup, lowpoly-island, hologram-hud…) will be very slow — warn before proposing them.
- **Don't use lemo-opuscar's `core/render/mux.sh` here:** with ffmpeg 4.4 its decoded-duration check under-reads by loudnorm's ~3 s look-ahead and rejects valid audio ("only 17.1 s of 20 s could be decoded"). The OpenMontage mix is already loudness-normalized; mux with ffmpeg as in step 8.
- `audio_mixer` outputs mono when the speech is mono.
- Disk is tight: delete `page/stills/` after review.

## Licences

- Code (engines, `core/`): MIT — fine to copy into projects.
- Demo films and stills: CC BY 4.0 — credit LemoLab if you reuse them; films we make with the library are ours, no credit needed.
- Never reuse the demo voice tracks of Game Show Flat, Paper-cut Lightbox, Broadcast News Package (not under CC BY).
- Fonts: Google Fonts (OFL), downloaded into the project, never hot-linked at render time.

## Decision log

Log the choice as `category: "render_runtime_selection"` with `lemo-opuscar (<style slug>)` as the selected option, Remotion / HyperFrames in `options_considered`, and the reason (look, offline, cost).
