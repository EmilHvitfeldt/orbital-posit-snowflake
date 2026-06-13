# orbital-posit-snowflake — project instructions

Animated explainers (Quarto RevealJS + anime.js) contrasting two ways of scoring a
model against data in Snowflake: the **round-trip** (data goes to a live session and
back) vs. the **orbital** way (the model is compiled to SQL that runs in place inside
Snowflake). See [orbital](https://orbital.tidymodels.org/) for the real package.

## Upstream framework — read this first

This repo follows the conventions of the **tidy-animations** framework:
<https://github.com/EmilHvitfeldt/tidy-animations>

Its `CLAUDE.md` (at the repo root) is the canonical reference for how these animations
are built — the `TM` namespace helpers, the idempotent `render(stage)` pattern, the
"build segment by segment" workflow, the GIF/MP4 capture tooling, and the gotchas. For
any future development here, **read that file first**; the notes below are only the
deltas specific to this repo.

## The 16-deck matrix (2 modules + knobs)

Everything is driven by **two** animation modules and a set of `window.ORB_*` knobs set
in a `` ```{=html} `` block at the top of each `.qmd`. The decks are the full cross of
four axes:

`{R, Python} × {non-Posit, Posit} × {roundtrip, inplace} × {horizontal, vertical}` = 16.

- `js/orbital-roundtrip.html` — the "before": data collects from the table, enters the
  workflow card, scores, and the prediction travels back to fill `.pred`.
- `js/orbital-inplace.html` — the orbital way: `orbital()` compiles the model to a SQL
  chip, the chip docks in Snowflake, `.pred` fills in place, the session winds down.

### Knobs (read with a fallback in each module)

| knob | effect | default |
| --- | --- | --- |
| `ORB_POSIT` | nest the session *inside* a Snowflake container (Posit Team Native App) | `false` |
| `ORB_ORIENT` | `'vertical'` → session on top, data on bottom, portrait 820×1080 stage (use a 900×1200 deck) | horizontal |
| `ORB_MODEL_LABEL` | model-card code (`workflow()` / `Pipeline`) | `workflow()` |
| `ORB_SESSION_LABEL` | session header text | `R session` / `Posit Workbench` |
| `ORB_SESSION_LOGO` | session logo chip (`R` / `Py`) | `R` |
| `ORB_SNOWFLAKE` | Snowflake accent | `#29B5E8` |
| `ORB_RSESSION` | session accent | `#276DC3` (R) / `#3776AB` (Python decks set this) |

**Adding/altering a deck is normally just a new `.qmd` with a different knob block** — do
not fork the modules. All geometry (panel/card/table/packet/chip positions, the runtime
boundary) is derived from a handful of layout constants that branch on `POSIT`/`VERT`, so
the choreography is shared. If you must change choreography, change it once in the module
and re-verify every affected deck.

## Repo-specific gotchas (in addition to the upstream ones)

- **Never seed a position with the `translate(x, y)` CSS shorthand on an element you then
  animate with anime's `translateX`/`translateY`.** anime v3 can't parse the shorthand and
  *appends* to it, doubling the offset. Build such elements without `node()`'s shorthand
  and set the start with `anime.set({translateX, translateY})` (see the packet/chip).
- **Infinite-loop animations (e.g. the thinking gear) use bare `anime(..., {loop:true})`,
  not `TM.anime`** — otherwise they pin the busy counter and the capture tool's idle
  detection never fires.
- `css/demos.css` is loaded as a reveal theme, so it needs the `/*-- scss:rules --*/`
  boundary.
- Vertical decks set their own portrait `width: 900` / `height: 1200`; the module sets the
  inner `#orb-stage` size from `ORB_ORIENT`.

## Build / preview / publish

```bash
quarto preview <deck>.qmd   # iterate on one deck (live reload)
quarto render               # render the whole site into docs/
```

The site is a Quarto **website** (`output-dir: docs`) so GitHub Pages can serve it from the
main branch `/docs` folder. `index.qmd` is the landing page linking every deck; `.nojekyll`
keeps `site_libs/` intact. Non-deck markdown (`README.md`) is excluded from rendering via
the project `render:` glob.

## Layout

- `_quarto.yml` — website + format defaults; loads anime.js and `js/infra.html` for every deck.
- `index.qmd` — landing page (HTML) linking all 16 decks.
- `*.qmd` — the decks (knob blocks at top).
- `js/infra.html` — shared `TM` helpers (copied from tidy-animations; keep in sync).
- `js/orbital-roundtrip.html`, `js/orbital-inplace.html` — the two animation modules.
- `css/demos.css` — deck styles (SCSS-layered); `css/index.css` — landing-page styles.
