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
  "description": "string — tooltip text",
  "link":        "string — optional URL. Renders as an Open ↗ link in the role drawer (opens when the column header is clicked). Useful for pointing to a tracking ticket, spec doc, benchmarks, or any external context for the role."
}
```

**Section object** (one per row group):

```jsonc
{
  "id":   "string — short key, used as candidate.section",
  "name": "string — section header label",
  "link": "string — optional URL. Renders as a small ↗ next to the section title. Useful for sections that map to a single tracked thing (a hiring ticket, an open issue, a project tracker). Pool / status sections that don't map to one anchor can omit it."
}
```

**`locationTiers` object** (optional, only meaningful when candidates are people and physical location matters for the role):

```jsonc
"locationTiers": {
  "label":     "string — row label shown in the drawer (default: 'Location tier')",
  "modifies":  ["roleId", ...],   // optional — see below
  "1":         "string — description for Tier 1",
  "2":         "string — description for Tier 2",
  "3":         "string — description for Tier 3"
}
```

When present, candidates with a `locationTier` value (1, 2, or 3) get a colored tier pill in their drawer. Common pattern: tier 1 = local / preferred, tier 2 = manageable / direct flight, tier 3 = stretch / connecting flight or international. The descriptions are free-form so you can frame the tier rule for your situation (travel access, timezone overlap, in-person availability, etc.).

**`modifies` field** — list of role IDs that this `locationTier` should *floor* on the fit tier. When set, the tool computes `effectiveTier = max(meritTier, locationTier)` for each named role at init, mutates the fit cell, and preserves the original under `fit._meritTier`. The candidate drawer surfaces this as `merit Tier X → floored to Tier Y by locationTier Z` on affected fits. Useful when location is a hard constraint on a specific role (e.g. an on-site hire, a regional partnership) and you want the matrix to reflect that reality without manually editing every fit cell. Roles not in `modifies` are unaffected.

The locationTier section is also suppressed in the candidate drawer when the candidate is closed or declined for *every* role in `modifies` — showing a top-tier pill on someone you've already rejected is a false signal. Drop a role from `modifies` to keep the pill visible regardless of fit status.

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
  "name":         "string — display name",
  "section":      "string — must match a section.id in config",
  "location":     { "country": "...", "state": "...", "city": "..." },  // optional, see below
  "locationTier": 1,                                                    // optional, 1/2/3, requires locationTiers config
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

**`location` field** (optional, candidate-level):

Object with optional `country`, `state`, `city` string fields. When present, the candidate drawer shows a Location row joined as `city, state, country` skipping any empty fields. If the field is present but every subfield is empty (e.g. `"location": {}`), the row renders as `?` to flag missing-but-expected info — useful for hiring use cases where every candidate should have location captured. Omit the field entirely to suppress the row (e.g. for non-people use cases like library evaluation).

**`locationTier` field** (optional, candidate-level):

Integer 1, 2, or 3. Only renders if `locationTiers` is configured in matrix-config.json. The tier description comes from the config; the pill color reuses the matrix tier palette (Tier 1 = primary, Tier 2 = secondary, Tier 3 = caution).

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

Created with the assistance of [Claude](https://www.claude.com/).
