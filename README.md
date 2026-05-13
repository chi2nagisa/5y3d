# 5 Years of Dan, 3 Mocks per Rank (5y3d)

> Taiko no Tatsujin Dan Dojo Theoretical Clear Calculator

[English](README.md) | [中文](README.zh.md)

A static web tool that calculates theoretical clear results (Gold Clear / Red Clear / Fail / Incomplete) for each Dan rank across all versions of *Taiko no Tatsujin* based on the player's historical best scores. Also determines Perfect / FullCombo / Normal frame status.

---

## Tech Stack

- **Pure static single-file**: All logic in `index.html`, zero build tools, zero frameworks
- **Data-driven**: Dan configurations and player scores loaded entirely from JSON
- **Browser storage**: Score data and manual overrides cached in `localStorage`

---

## Data Specs

### Dan Data `dani_datas/dani_data.json`

Root: `{ "versions": [...] }`, ordered chronologically.

Each `version`:
- `id`: version slug, e.g. `green`, `blue`, `nijiiro_2025`
- `name_zh`: Chinese version name
- `danis`: array of dan objects

Each `dani`:
- `id`: dan slug
- `sort_order`: sort index (0~24)
- `total_notes`: total note count across 3 songs
- `songs`: 3 exam songs with `song_id`, `name`, `name_zh`, `difficulty`, `stars`, `notes`
- `clear_conditions`: array of clear conditions

**Adding a new version**: append to the `versions` array; no code changes required.

### Extra Dan Data `dani_datas/gaiden_data.json`

Same schema as `dani_data.json`, but `danis` order follows the raw JSON order (no `sort_order`).

### Score Data Format

Array of objects:
- `song_no` (Integer): song ID, maps to `song_id` in dan data
- `level` (Integer): difficulty, `1~5` maps to Easy/Normal/Hard/Oni/Oni(Ura)
- `high_score` (Integer): total song score
- `high_score_result` (Integer[5]):
  - `[0]` Perfect count
  - `[1]` Good count
  - `[2]` Bad count
  - `[3]` Drumroll count
  - `[4]` Max combo

**Data sources**:
- donder API: `https://hasura.llx.life/api/rest/donder/get-score?id={id}`
- Local JSON file upload


---

## Core Logic

### Clear Calculation

`computeDaniResult(dani)` matches scores for each exam song, then aggregates by condition type:

| Condition | shared | per_song |
|-----------|--------|----------|
| `good` | Sum of `[0]` | Per-song `[0]` |
| `ok` | Sum of `[1]` | Per-song `[1]` |
| `bad` | Sum of `[2]` | Per-song `[2]` |
| `roll` / `rolls` | Sum of `[3]` | Per-song `[3]` |
| `combo` | Sum of `[4]` | Per-song `[4]` |
| `hits` | `good + ok + roll` total | Per-song |
| `score` | Sum of `high_score` | Per-song `high_score` |
| `gauge` | **Display only, not calculated** | **Display only, not calculated** |

### Frame Judgment

- **Perfect**: All 3 songs have `perfect count === note count`
- **FullCombo**: All 3 songs have `max combo === note count`, but not all perfect
- **Normal**: Neither of the above

### Manual Score Editing

- Entry: "Edit" button next to each exam song in the detail panel
- Storage: `localStorage['5y3d_manual_scores']`, physically isolated from real scores
- Read priority: Manual > Real (API/JSON) > Example data
- Reset: "Reset Manual Scores" button in the Import Data panel

### Bilingual Song Names

- Default: Chinese (`songs[].name_zh`)
- Fallback to Japanese (`songs[].name`) when `name_zh` is null or empty
- Preference persisted in `localStorage['5y3d_lang']`

---

## Local Dev

```bash
python -m http.server 8080
# Open http://localhost:8080
```

**File structure**:

```
.
├── index.html                 # Entry (HTML + CSS + JS)
├── dani_datas/
│   ├── dani_data.json        # Main dan config (v3.0)
│   └── gaiden_data.json      # Extra dan config
└── dani_score_logo/           # Clear status icons (WebP)
```

---

## Version History

| Version | Date | Highlights |
|---------|------|------------|
| v1.5.1 | 2026-05-11 | Two-row song list layout to prevent overlap |
| v1.5.0 | 2026-05-11 | Bilingual song name switching (zh/ja) |
| v1.4.0 | 2026-05-11 | Manual score editing & what-if analysis |
| v1.3.1 | 2026-05-11 | Song name column width fix |
| v1.3.0 | 2025-04-19 | Score import page (donder API + JSON upload) |
| v1.2.0 | 2025-04-19 | Extra dan panel actual rendering |
| v1.1.1 | 2025-04-19 | `score` condition support, extra data fixes |
| v1.1.0 | 2025-04-17 | Main/Extra tab switching |
| v1.0.0 | 2025-04-09 | Initial: matrix display, clear calc, detail panel |

---

## Known Issues / TODO

- [ ] Extra mode detail panel may scroll horizontally on narrow screens

- [ ] Many Color Pop songs have `song_id = -1`, unmatchable
- [ ] Manual editing doesn't support `high_score` override (incomplete for `score` condition what-if)
