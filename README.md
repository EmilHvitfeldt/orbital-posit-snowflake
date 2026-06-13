# orbital + Snowflake animations

Self-contained Quarto RevealJS decks contrasting how a tidymodels (or scikit-learn) model gets scored against data in Snowflake. They tell opposite stories about how predictions get into the `.pred` column, across a 2×2 of framings:

|              | non-orbital (the "before")   | orbital (in place)        |
| ------------ | ---------------------------- | ------------------------- |
| **non-Posit** | `orbital-roundtrip.qmd`      | `orbital-inplace.qmd`     |
| **Posit**     | `posit-roundtrip.qmd`        | `posit-inplace.qmd`       |

That's the **R** cut. There's a matching **Python** (scikit-learn) cut of every deck, prefixed `python-` (e.g. `python-orbital-roundtrip.qmd`, `python-posit-inplace.qmd`). And every deck has a **vertical** (portrait) variant with `-vertical` appended (e.g. `orbital-roundtrip-vertical.qmd`, `python-posit-inplace-vertical.qmd`).

So the full set is 4 axes — `{R, Python} × {non-Posit, Posit} × {roundtrip, inplace} × {horizontal, vertical}` = **16 decks**, all driven by two animation modules via `window.ORB_*` knobs:

- **Posit** decks set `window.ORB_POSIT = true` — nests the session *inside* a Snowflake container (the Posit Team Native App in Snowpark Container Services), so the data never leaves Snowflake.
- **vertical** decks set `window.ORB_ORIENT = 'vertical'` (portrait 900×1200 canvas) — session on top, data table on the bottom, flow running vertically.
- **Python** decks set `ORB_MODEL_LABEL = 'Pipeline'`, `ORB_SESSION_LOGO = 'Py'`, a Python-blue accent, and a Python session label.

The choreography is identical across all of them; only labels, colours, and layout change.

Built with Quarto RevealJS + [anime.js](https://animejs.com/) v3, driven by RevealJS fragments. See [orbital](https://orbital.tidymodels.org/) for the real package.

`index.qmd` is a landing page linking every deck; the whole thing renders into `docs/` as a Quarto website, ready to serve via **GitHub Pages** (main branch `/docs`).

## non-orbital — the "before" (round-trip)

`js/orbital-roundtrip.html`, used by `orbital-roundtrip.qmd` (R session beside Snowflake) and `posit-roundtrip.qmd` (Posit Workbench session nested inside Snowflake).

- **Stage 0** — session (pulsing "running", trained `workflow()` card) + Snowflake table with an empty `.pred` column.
- **Stage 1** — the data rows "collect" into a packet, which travels to the session and **enters the top of the workflow**; a gear spins while it computes.
- **Stage 2** — the prediction **emerges from the bottom of the workflow**, travels back, and lands on the `.pred` column, filling it in.

In the Posit cut both panels live inside one Snowflake container, with a dashed runtime-boundary divider: the data never leaves Snowflake, but still has to cross into a live runtime to be scored.

## orbital — the orbital way (in place)

`js/orbital-inplace.html`, used by `orbital-inplace.qmd` and `posit-inplace.qmd`.

- **Stage 0** — same starting frame.
- **Stage 1** — `orbital()` compiles the model card into a `SELECT … AS .pred` SQL chip, in place (no data moves).
- **Stage 2** — the SQL chip travels into Snowflake and docks over the table (the query goes to the data); the session winds down.
- **Stage 3** — the query runs in place and `.pred` fills; the session's indicator is grey: **running → not needed**.

In all decks, forward nav, back nav, and direct-link/reload arrival all resolve to the correct visual state (the render is idempotent).

## Run them

```bash
quarto preview orbital-roundtrip.qmd   # non-Posit, the "before"
quarto preview orbital-inplace.qmd     # non-Posit, the orbital way
quarto preview posit-roundtrip.qmd     # Posit, the "before"
quarto preview posit-inplace.qmd       # Posit, the orbital way
quarto preview orbital-inplace-vertical.qmd   # any deck + "-vertical" = portrait
quarto preview python-posit-inplace.qmd       # any deck + "python-" prefix = Python
quarto render                          # render the whole site (16 decks + landing page) into docs/
```

Navigate with the arrow keys: each press advances one fragment (stage).

## Layout

- `_quarto.yml` — website + format defaults; loads anime.js and the shared `js/infra.html` for every deck. Renders into `docs/`.
- `index.qmd` — the landing page (HTML) linking all 16 decks.
- `*.qmd` — the 16 decks (see the table above). Per-deck labels/colours/framing are set in a top-of-file `{=html}` block (`window.ORB_*`).
- `js/infra.html` — shared helpers under the `TM` namespace (`TM.onReveal`, `TM.gate`, `TM.anime`, `TM.pause`).
- `js/orbital-roundtrip.html`, `js/orbital-inplace.html` — the two animation modules; each drives all of its R/Python × non-Posit/Posit × horizontal/vertical decks via the `ORB_*` knobs.
- `css/demos.css` — deck styles (loaded as a reveal theme, so it is SCSS-layered with a `/*-- scss:rules --*/` boundary); `css/index.css` — landing-page styles.

## Configurable knobs

Set in the `{=html}` block at the top of each `.qmd`:

| global | meaning | default |
| --- | --- | --- |
| `window.ORB_POSIT` | nest the session inside a Snowflake container (Posit framing) | `false` |
| `window.ORB_ORIENT` | `'vertical'` stacks session over data (use a portrait canvas); else horizontal | horizontal |
| `window.ORB_MODEL_LABEL` | code shown on the model card (`workflow()` for R, `Pipeline` for Python) | `workflow()` |
| `window.ORB_SESSION_LABEL` | session header label | `R session` (or `Posit Workbench` when `ORB_POSIT`) |
| `window.ORB_SESSION_LOGO` | session logo chip (`R` / `Py`) | `R` |
| `window.ORB_SNOWFLAKE` | Snowflake accent colour | `#29B5E8` |
| `window.ORB_RSESSION` | session accent colour | `#276DC3` |

## Development

These animations follow the **tidy-animations** framework — the `TM` helpers, the
idempotent `render(stage)` pattern, the segment-by-segment build workflow, and the
GIF/MP4 capture tooling. For conventions and the full set of gotchas, see that repo and
its `CLAUDE.md`:

<https://github.com/EmilHvitfeldt/tidy-animations>

Repo-specific notes (the knob system and a couple of anime.js gotchas) live in this
repo's [`.claude/CLAUDE.md`](.claude/CLAUDE.md). Adding a new deck is normally just a new `.qmd` with a
different `window.ORB_*` knob block — don't fork the modules.
