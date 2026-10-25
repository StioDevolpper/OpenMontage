---
description: Short film in one of Lemo-Opuscar's 59 code-drawn styles (watercolor, ink wash, Ukiyo-e, risograph…). Picture rendered from code by the Lemo-Opuscar engine; voice, music, mix and subtitles by OpenMontage.
argument-hint: [style] [topic]
---

Read `skills/creative/lemo-opuscar.md` (when it fits, setup, workflow, machine notes, licences), then the chosen style's `~/Projects277/lemo-opuscar/styles/<slug>/STYLE.md`, and make the film described below.

- No style named? Suggest 2–3 from `~/Projects277/lemo-opuscar/styles/README.md` that fit the topic, with the gallery link https://lemomo-ai.github.io/lemo-opuscar/, and let the user pick.
- Work in `projects/<slug>/`: voice first (its durations set the timeline), then the page (`window.DUR` / `render(t)` / `READY` / `EV`), stills + contact sheet, silent render with `--workers 2`, music/foley, `audio_mixer` `full_mix`, ffmpeg mux, self-check.
- Reference build: `projects/spring-bean-aquarelle/` (page/main.js, score.py, mix.py).

In OpenMontage this runs inside the `explainer` or `animation` pipeline as the picture engine — it is a style library + engine, not a separate pipeline.

Request: $ARGUMENTS
