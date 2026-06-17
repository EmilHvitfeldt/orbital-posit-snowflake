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

## The deck matrix (2 modules + knobs)

Everything is driven by **two** animation modules and a set of `window.ORB_*` knobs set
in a `` ```{=html} `` block at the top of each `.qmd`. The single-animation decks are the
full cross of four axes:

`{R, Python} × {non-Posit, Posit} × {roundtrip, inplace} × {horizontal, vertical}` = 16.

- `js/orbital-roundtrip.html` — the "before": data collects from the table, enters the
  workflow card, scores, and the prediction travels back to fill `.pred`.
- `js/orbital-inplace.html` — the orbital way: `orbital()` compiles the model to a SQL
  chip, the chip docks in Snowflake, `.pred` fills in place, the session winds down.

### Combined "before → after" decks (`*-combined*.qmd`)

On top of the 16, there are **8 combined decks** — `{R, Python} × {non-Posit, Posit} ×
{horizontal, vertical}` — that put the round-trip and the in-place animations in **one
deck as two slides**, so the viewer steps through "before" then arrows into "after" in one
flow. They reuse both modules unchanged: see "Two animations on one page" below.

### Knobs (read with a fallback in each module)

| knob | effect | default |
| --- | --- | --- |
| `ORB_POSIT` | nest the session *inside* a Snowflake container (Posit Team Native App) | `false` |
| `ORB_ORIENT` | `'vertical'` → session on top, data on bottom, portrait 820×1080 stage (use a 900×1200 deck) | horizontal |
| `ORB_MODEL_LABEL` | model-card code (`workflow()` / `Pipeline`) | `workflow()` |
| `ORB_SESSION_LABEL` | session header text | `R session` / `Posit Workbench` |
| `ORB_SESSION_LOGO` | session logo chip: a short string (`R` / `Py`) renders as a letter chip, an image path (`.svg`/`.png`/…, e.g. `js/posit-workbench.svg`) renders as an `<img>`. Posit decks use the Workbench logo. | `R` |
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
  inner stage size from `ORB_ORIENT`. CSS targets the stage via `[id^="orb-stage"]` so it
  covers both the lone `#orb-stage` and the combined deck's `#orb-stage-rt` / `#orb-stage-ip`.

## Two animations on one page (combined decks)

The modules are **parametrized** so more than one can run on a page (the combined decks):

- Each module is an init function — `ORB.initRoundtrip(opts)` / `ORB.initInplace(opts)` —
  taking `{stageId, markerClass, fragmentIds}`, all defaulting to the standalone values.
  Standalone decks self-init at the bottom of the module **unless `window.ORB_COMBINED`**
  is set; combined decks set that knob and call the init functions themselves (in
  `js/orbital-combined-init.html`), once per stage.
- `ORB.build()` takes `opts.stageId` (default `'orb-stage'`) and caches per stage id in
  `ORB._ctx[stageId]`, so each stage gets its own independent context.
- The two modules' default `fragmentIds` differ (`orb-rt-*` vs `orb-compile`/…), so a
  combined deck only overrides `stageId` + `markerClass`, not the fragment ids.
- A combined deck = one knob block (`ORB_COMBINED` + the usual knobs) + two `.orb-slide`
  sections, each with its own stage `<div>` and marker class, and its own
  `include-after-body` listing both modules then `js/orbital-combined-init.html`.

## Deck navigation overlay

`js/orbital-nav.html` (loaded for every deck via `_quarto.yml`) adds a completion CTA that
fades in on the deck's **last** slide once every fragment is shown (`Reveal.isLastSlide()`
gates it so combined decks don't surface it between their two sections). On standalone
round-trip decks the CTA links to the matching in-place deck (before → after); on in-place
decks it offers a replay of the round-trip; all decks get an "All decks" link back to the
landing page. The counterpart deck is derived from the filename by swapping
`roundtrip`↔`inplace`.

## Build / preview / publish

```bash
quarto preview <deck>.qmd   # iterate on one deck (live reload)
quarto render               # render the whole site into docs/
```

The site is a Quarto **website** (`output-dir: docs`) so GitHub Pages can serve it from the
main branch `/docs` folder. `index.qmd` is the landing page linking every deck (16 single +
8 combined); `.nojekyll`
keeps `site_libs/` intact. Non-deck markdown (`README.md`) is excluded from rendering via
the project `render:` glob.

## Layout

- `_quarto.yml` — website + format defaults; loads anime.js, `js/infra.html`, `js/orbital-base.html`, and `js/orbital-nav.html` for every deck.
- `index.qmd` — landing page (HTML) linking all decks (16 single + 8 combined).
- `*.qmd` — the decks (knob blocks at top); `*-combined*.qmd` are the two-slide before→after decks.
- `js/anime.min.js` — vendored anime.js v3.2.2 (referenced by the header include; listed under project `resources:` so it is copied into `docs/`).
- `js/infra.html` — shared `TM` helpers (copied from tidy-animations; keep in sync).
- `js/orbital-base.html` — `ORB.build(opts)`: the shared starting frame, all layout maths, the data table, and the DOM helpers, returned as a context object. **Both modules consume this — change geometry here, once.** `opts.groupSession` toggles whether the session winds down as one `.orb-rgroup` (in-place) or its panel/card dim individually (round-trip).
- `js/orbital-roundtrip.html`, `js/orbital-inplace.html` — the two animation modules, each exposing an `ORB.init*` function (see "Two animations on one page"); each owns only its phase choreography and calls `ORB.build()` lazily inside `render()`.
- `js/orbital-combined-init.html` — wiring for the combined decks: binds each module to its own stage/slide.
- `js/orbital-nav.html` — the before→after completion CTA (loaded for every deck).
- `css/demos.css` — deck styles (SCSS-layered), including the nav overlay; `css/index.css` — landing-page styles.
