# Role Alignment Matrix

A self-contained HTML tool for visualizing a pool of candidates (or items, libraries, vendors, anything) against a set of roles (or dimensions, evaluation criteria) with tiered fit scoring, per-role rollups, an action-priority view, and a written-observations panel.

No build step. No dependencies. One static HTML file plus two JSON files of your data.

## What it does

The matrix view shows a grid of candidates × roles, each cell colored by fit tier (1 / 2 / 3) with primary-lane indicators, declined and closed states, and per-cell notes that surface in a side drawer.

Two companion views ship alongside:

- **Priorities** — a three-bucket action plan (in motion / this week / on hold) for whatever next-step actions your situation calls for
- **Briefing Notes** — per-role tier rollups and free-form observations

You bring two JSON files:

- `matrix-config.json` — the canvas: title, the roles (columns), the section row groups
- `matrix-data.json` — the population: the candidates, their fits, the action plan, the observations

Edit, refresh, repeat.

## What it's good for

- Hiring panels evaluating multiple candidates against role-shapes
- Library / vendor / tool evaluation against project requirements
- Any 2D fit grid where you want a matrix view + per-role rollup + action plan + observations

The included example renders an open source library evaluation. Swap the JSON files to make it your own.

## Quick start

```bash
git clone https://github.com/dannybauman/role-alignment-matrix.git
cd role-alignment-matrix

# Copy the example JSONs to the names the HTML loads
cp matrix-config.example.json matrix-config.json
cp matrix-data.example.json   matrix-data.json

# Serve locally (any static server works)
python3 -m http.server 8000

# Open http://localhost:8000
```

Edit `matrix-config.json` and `matrix-data.json` to render your own situation.

## Why the local server

The HTML loads its data via `fetch()`. Browsers block `fetch()` from `file://` URLs for security, so a local HTTP server is required. Any of these work:

```bash
python3 -m http.server 8000
npx serve .
ruby -run -ehttpd . -p 8000
```

## File overview

```
index.html                       static shell, no data
matrix-config.example.json       example canvas: roles + sections + page metadata
matrix-data.example.json         example population: candidates + fits + rollups + actions + observations
matrix-config.json               your canvas (gitignored)
matrix-data.json                 your population (gitignored)
README.md                        this file
LICENSE                          MIT
```

The non-`.example` versions are gitignored so your private data doesn't accidentally get committed if you're working in a fork.

## Schema

### `matrix-config.json`

Top-level fields:

```jsonc
{
  "title":         "string — page title and h1",
  "eyebrow":       "string — small label above the title",
  "metaItems":     ["array", "of", "small bullet items below the title"],
  "counterLabel":  "string — label under the candidate-count / role-count display",
  "roles":         [ /* role objects, see below */ ],
  "sections":      [ /* section objects, see below */ ]
}
```

**Role object** (one per matrix column):

```jsonc
{
  "id":          "string — short key, used as the key in candidate.fits",
  "name":        "string — full label shown in the column header",
  "status":      "active | prospective",
  "shortStatus": "string — sub-label under the role name",
  "description": "string — tooltip text"
}
```

**Section object** (one per row group):

```jsonc
{
  "id":   "string — short key, used as candidate.section",
  "name": "string — section header label"
}
```

### `matrix-data.json`

Top-level fields:

```jsonc
{
  "candidates":   [ /* candidate objects, see below */ ],
  "links":        { /* per-name link maps */ },
  "roleRollups":  { /* per-role tier rollups */ },
  "outreach":     { /* three-bucket action plan */ },
  "observations": [ /* commentary entries */ ]
}
```

**Candidate object** (one per row):

```jsonc
{
  "name":    "string — display name",
  "section": "string — must match a section.id in config",
  "fits": {
    "<roleId>": {
      "tier":     1,                  // 1, 2, or 3 — fit strength. omit for closed/declined.
      "range":    "1-2",              // optional — overrides tier display when fit is uncertain
      "primary":  true,               // optional — marks the candidate's primary lane for that role
      "note":     "string",           // optional — short context shown in the drawer
      "closed":   true,               // optional — alternative to tier; renders as a closed cell
      "declined": true                // optional — candidate-side decline indicator
    }
  }
}
```

A candidate can have a fit cell for any role they apply to. Omitted role IDs render as empty cells.

**Links object** (per-candidate external links shown in the side drawer):

```jsonc
{
  "<candidateName>": {
    "site":    "https://...",
    "github":  "https://...",
    "linkedin": "https://...",
    "greenhouse": "https://..."     // any keys are accepted; UI shows them as "site", "github", etc.
  }
}
```

**roleRollups object** (per-role tier breakdown shown in the briefing view):

```jsonc
{
  "<roleId>": {
    "tier1": ["Name 1", "Name 2 (with parenthetical context)"],
    "tier2": [...],
    "tier3": [...],
    "note":  "string — commentary on pool depth and shape"
  }
}
```

**outreach object** (three-bucket action plan shown in the priorities view):

```jsonc
{
  "motion": [ { "name": "...", "tag": "...", "rationale": "..." } ],
  "week":   [ { "priority": 1, "name": "...", "tag": "...", "rationale": "..." } ],
  "hold":   [ { "name": "...", "tag": "...", "rationale": "..." } ]
}
```

`priority` on `week` items is 1-5; lower numbers render first.

**observations array** (free-form commentary shown in the briefing view):

```jsonc
[
  { "title": "string", "body": "string" }
]
```

## Customizing

- **Add or remove roles**: edit `matrix-config.json` `roles` array. The `id` field is what links a role to candidate fits, so if you rename a role you'll need to update the corresponding key in each `candidate.fits` object.
- **Add or remove sections (row groups)**: edit `matrix-config.json` `sections`. Each `candidate.section` must match a section `id`.
- **Add candidates**: append to `matrix-data.json` `candidates`. The minimum is `name`, `section`, and at least one `fits` entry.
- **Customize visible labels**: title / eyebrow / metaItems / counterLabel are all in `matrix-config.json`. Tab labels ("Matrix" / "Priorities" / "Briefing Notes") are hardcoded in `index.html` and can be edited directly if needed.

## Project structure

The HTML is a single self-contained file (CSS + JS inline). No bundler, no build step, no npm. The render code reads from globals that are populated at startup by fetching the two JSON files. The fetch flow lives at the bottom of `index.html` and is the only place data loading is wired — easy to swap out (e.g., to load from a URL parameter or a remote API) if you want.

## License

[MIT](LICENSE).

## Origin

Originally built as a private internal tool for a recruiting working session, then extracted and genericized for general use. Real data lives in a separate (private) data file; this repo carries only the example.
