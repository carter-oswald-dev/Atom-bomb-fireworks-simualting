# Nuke Brightness Simulator

A browser-based visualization of how bright a nuclear detonation would
appear against a real night sky, at any yield and distance you choose.
Renders an actual star field (Yale Bright Star Catalog) for a fixed
observer location and date, then simulates the flash's apparent magnitude
and washout effect on the visible stars.

**[Live demo](#)** — replace with your GitHub Pages URL once deployed.

---

## Contents

- [Quick start](#quick-start)
- [Using the simulator](#using-the-simulator)
- [Preset scenarios](#preset-scenarios)
- [Loading your own scenario](#loading-your-own-scenario)
- [Building a custom scenario JSON](#building-a-custom-scenario-json)
- [Adding a scenario to the built-in dropdown](#adding-a-scenario-to-the-built-in-dropdown)
- [Hosting on GitHub Pages](#hosting-on-github-pages)
- [How the brightness is calculated](#how-the-brightness-is-calculated)
- [File structure](#file-structure)

---

## Quick start

1. Open `index.html` in a browser (double-click it, or visit the hosted
   Pages URL).
2. Either pick a **preset scenario** from the dropdown on the right, or
   type a **Yield (kt)** and **Distance (km)** into the manual fields and
   click **DETONATE**.
3. Click **▶ PLAY** to run the animation. The page never autoplays —
   nothing happens until you press Play.

---

## Using the simulator

**Left sidebar**
| Control | What it does |
|---|---|
| Location / Sky date | Fixed reference sky (39.83°N 98.58°W, night of Jul 13/14 2026) — not adjustable, it's the backdrop star field |
| Event | Current yield + distance in effect |
| Mag | Live apparent magnitude of the flash at the current animation time |
| T+ | Seconds since detonation (negative = before flash) |
| Status | Playing / Paused / Stopped |
| ▶ PLAY | Starts or resumes the animation from wherever it left off |
| ⏸ PAUSE | Freezes the animation; time is preserved |
| ⟲ RESTART | Resets to T+0. If it was playing, keeps playing from the start |

If the computed peak brightness is fainter than magnitude **+6.0** (the
naked-eye visibility limit), a warning appears telling you the flash won't
be visible — try a bigger yield or shorter distance.

**Right sidebar**
| Control | What it does |
|---|---|
| Preset scenarios dropdown | Loads a built-in scenario from `scenarios/` |
| Scenario description | Shows the preset's description (if it has one) |
| LOAD SCENARIO JSON FILE | Loads a single scenario `.json` from your computer |
| Yield / Distance fields | Manual entry, used when "Custom" is selected |
| DETONATE | Applies the manually-entered yield/distance |

Loading a scenario or clicking DETONATE always resets playback to a paused
frame at T+0 — you still have to hit Play yourself.

---

## Preset scenarios

The dropdown is populated from `scenarios/index.json`, a manifest listing
which scenario files to load. Four examples ship with this project:

- Original reference case (500 kt @ 200,000 km)
- Tsar Bomba–class yield at low-Earth-orbit altitude
- 5 Mt at lunar distance
- 100 Mt (max yield) at deep interplanetary range

---

## Loading your own scenario

You don't need to touch the repo to try your own numbers:

1. Save a JSON file on your computer following the format in the [next
   section](#building-a-custom-scenario-json).
2. Click **＋ LOAD SCENARIO JSON FILE** and select it.
3. It's added to the dropdown for the rest of your session and applied
   immediately.

This works even if you just opened `index.html` straight from disk
(`file://`) — no server needed for this path.

---

## Building a custom scenario JSON

A scenario file is a single JSON object:

```json
{
  "name": "Short label shown in the dropdown",
  "description": "Optional. Max 250 characters — longer text is automatically cut off.",
  "distance_km": 200000,
  "yield_kt": 500
}
```

| Field | Required | Notes |
|---|---|---|
| `name` | No | Falls back to "Unnamed scenario" if missing |
| `description` | No | Hard-capped at 250 characters, even if the file has more |
| `distance_km` | **Yes** | Must be a positive number |
| `yield_kt` | **Yes** | Must be a positive number. Values above 100,000 (100 Mt — the largest weapon class ever designed) are silently clamped down to 100,000 |

Alternate key names `distanceKm`/`distance` and `yieldKt`/`yield` are also
accepted, in case you're generating files from another tool with different
naming conventions. `details` is accepted as an alias for `description`.

---

## Adding a scenario to the built-in dropdown

To make a scenario show up for *every* visitor (not just loaded locally by
one person), commit it to the repo:

1. Add your scenario JSON file to the `scenarios/` folder, e.g.
   `scenarios/my_scenario.json`.
2. Add its path to `scenarios/index.json`, which is just a flat array of
   file paths:

```json
[
  "scenarios/original_500kt.json",
  "scenarios/tsar_bomba_leo.json",
  "scenarios/moon_distance_5mt.json",
  "scenarios/deep_space_100mt.json",
  "scenarios/my_scenario.json"
]
```

3. Commit and push. On the next page load, it appears in the dropdown —
   no code changes needed.

`index.json` entries can also be inline objects instead of file paths, if
you'd rather keep everything in one file:

```json
[
  "scenarios/original_500kt.json",
  { "name": "Inline example", "distance_km": 1000000, "yield_kt": 2000 }
]
```

---

## Hosting on GitHub Pages

1. Push this folder structure to a repo (`index.html` and `scenarios/` in
   the same directory — either the repo root or whatever folder you point
   Pages at).
2. In repo Settings → Pages, enable Pages for that branch/folder.
3. Done. The manifest is fetched with a relative path
   (`scenarios/index.json`), so it works whether the site is served from a
   root domain (`username.github.io`) or a project subpath
   (`username.github.io/reponame/`).

**Note:** the dropdown/manifest requires the page to be served over HTTP —
GitHub Pages, or locally via `python3 -m http.server`. Opening `index.html`
directly from disk (`file://`) will skip the manifest fetch silently and
just show "Custom" in the dropdown; use the **LOAD SCENARIO JSON FILE**
button instead in that case.

---

## How the brightness is calculated

Peak apparent magnitude is derived from a simplified thermal-pulse model
rather than a naive inverse-square/linear-yield scaling:

- Total visible-light energy is assumed proportional to yield.
- The thermal pulse's characteristic duration scales with yield as
  `t ≈ 0.0417 × (yield_kt)^0.44` seconds (Glasstone & Dolan approximation),
  which means bigger detonations spread their energy over a longer pulse
  rather than just getting proportionally brighter.
- Peak power ∝ (visible energy) / (pulse duration).
- Apparent brightness then falls off with the inverse square of distance.
- The whole model is calibrated against a reference point (500 kt @
  200,000 km → magnitude −12.30, roughly full-moon brightness) so the
  scaling behaves physically rather than diverging at extreme yields or
  distances.

This is a simplified educational approximation, not a weapons-effects
reference — real thermal output depends on burst altitude, atmospheric
absorption (irrelevant here since these are space detonations), fireball
opacity, and spectral distribution in ways this model doesn't fully
capture.

---

## File structure

```
.
├── index.html              # the simulator — fully self-contained
├── README.md                # this file
└── scenarios/
    ├── index.json            # manifest: list of scenario files to auto-load
    ├── original_500kt.json
    ├── tsar_bomba_leo.json
    ├── moon_distance_5mt.json
    └── deep_space_100mt.json
```

`index.html` has no external dependencies — no build step, no npm, no
CDN links. It's a single static file plus the optional `scenarios/` data
folder.
