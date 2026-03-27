# Gutey Guesser — Claude Context

## What this is

A family NFL draft prediction game built as a single React artifact. Originally called the **Ted Thompson Whisperer** (2017), renamed Gutey Guesser after Brian Gutekunst became Packers GM in 2018. Played every year at the NFL Draft. The family submits picks for each Packers draft selection, scored by how close they got.

## Participants

| Name   | Joined | Notes |
|--------|--------|-------|
| Riley  | 2017   | Record holder — 52 pts in 2024 |
| Jeremy | 2017   | 2-time winner (2022, 2025) |
| Dad    | 2017   | 2-time winner (2017, 2023) |
| Kelsey | 2022   | Did not participate 2025 |
| Jen    | 2022   | Did not participate 2025 |

## Scoring system

```
15 pts  — Right player, right round
10 pts  — Right player, different round
 5 pts  — Right position, right round
 1 pt   — Right position, different round
```

**Position groupings** (these normalize before scoring):
- `OL` = OT, OG, G, C, OG/C
- `DL` = DT, NT, IDL
- `EDGE` = DE
- `LB` = ILB, MLB, OLB
- `DB` = CB, S, FS, SS

## 2026 draft setup

- **No Round 1 pick** — the Packers traded away their first-round pick this year
- **Default pick slots with overall numbers**:
  - Round 2 — Pick #52
  - Round 3 — Pick #84
  - Round 4 — Pick #120
  - Round 5 — Pick #160
  - Round 6 — Pick #201
  - Round 7 — Pick #236
  - Round 7 (comp) — Pick #255
- **Lock times**:
  - Rounds 2–3 lock: Friday April 24, ~7pm CT
  - Rounds 4–7 lock: Saturday April 25, ~noon CT
- **Day 2 update rule**: After Day 2 picks, participants can revise their Day 3 slate based on what's been taken
- **Trade flexibility**: Commissioner can add/remove pick slots and mark picks as traded away at any time via the Admin tab. This does not affect scoring logic — scoring is always by round number.

## Pick slot system

Each pick slot has the shape `{id, round, label, pickNumber?, traded?}`:
- `id` — the key used in `picks` and `actualPicks` storage (e.g. `"2"`, `"7a"`, `"7b"`)
- `round` — numeric round (2–7), drives scoring comparisons
- `label` — display string (e.g. `"Round 7 — Pick 1"`)
- `pickNumber` — overall draft slot number (e.g. 52). Shown on pick cards and draft board. No scoring effect.
- `traded` — if true, the slot is shown as "Pick traded away" to participants and excluded from scoring

When the Packers acquire a pick via trade, the commissioner adds a new slot. When they trade one away, they mark it traded (or remove it). Slots are stored in shared storage under `"pick-slots"` and loaded at app start.

Each actual pick also stores `pickNumber` (the overall selection number, e.g. 41) for display, but pick number has no effect on scoring.

## Technical setup

- **Framework**: React (JSX, hooks only — no router, no build step)
- **Deployment**: Claude.ai artifact (single `.jsx` file)
- **Storage**: `window.storage` (Claude artifact persistent key-value store, shared across users)
- **Fonts**: Google Fonts — Bebas Neue (headings), Barlow Condensed (labels/subheads), Barlow (body)
- **Logo**: Packers "G" logo embedded as base64 PNG with transparent background
- **No external API dependencies** — actual picks entered manually by commissioner during the draft

## Storage keys (all `shared: true`)

| Key | Value |
|-----|-------|
| `family-members` | `string[]` — list of participant names |
| `picks-{name}` | `object` — `{ [slotId]: { player, position, school } }` |
| `actual-picks` | `object` — `{ [slotId]: { player, position, school, pickNumber } }` |
| `pick-slots` | `{id, round, label, traded?}[]` — commissioner-managed, defaults to `DEFAULT_SLOTS` |
| `invite-code` | `string` — default set privately, not in repo |

## Access model

- **Public** (no auth): Leaderboard, Draft Board, History are fully visible
- **Participants** (invite code + name): Unlocks "My Picks" and "Admin" tabs via modal overlay
- **Commissioner** (admin password — stored privately, not in repo): Can enter actual picks and change invite code

## Files in this project

- `gutey-guesser.jsx` — full React app (single file, self-contained)
- `CLAUDE.md` — this file
- `DESIGN.md` — design system documentation

## Historical champion roster

| Year | Location | Winner | Score | Participants |
|------|----------|--------|-------|-------------|
| 2017 | Philadelphia, PA | Dad    | 27    | Riley, Jeremy, Dad |
| 2018 | Arlington, TX    | —      | —     | Scores not recovered (picks data known) |
| 2019 | Nashville, TN    | Riley  | 24    | Riley, Jeremy, Dad |
| 2022 | Las Vegas, NV    | Jeremy | 38    | All five |
| 2023 | Kansas City, MO  | Dad    | 41    | All five |
| 2024 | Detroit, MI      | Riley  | 52 ⭐  | All five |
| 2025 | Green Bay, WI    | Jeremy | 33    | Riley, Jeremy, Dad |

2018 actual picks are fully documented (Jaire Alexander R1, Josh Jackson R2, etc.) but participant scores weren't preserved. The year card expands to show the Packers' picks but shows no Final Scores section.

## Key decisions and why

**Why no NFL API?** Real-time NFL draft APIs (Sportradar) require paid B2B contracts. The commissioner enters picks live on the Admin tab during the draft — faster and more reliable for a family game.

**Why a single JSX file?** Deployed as a Claude artifact, which renders a single React component. No build tooling, no hosting costs, no dependencies to manage.

**Why public leaderboard?** The family wanted to be able to share the link with anyone who wants to follow along (spouses, friends) without needing a code. Only submitting picks requires the invite code.

**Why is Round 1 auto-scored?** Per house rules, everyone submits picks starting at Round 2. The R1 pick (Micah Parsons in 2026) is pre-announced and everyone gets the 15 pts automatically.

**Why player + position required, school optional?** Player name and position are needed for the scoring engine to evaluate hits. School is display-only flavor text.

## When working on this project

- All HISTORY data is hardcoded in the JSX — parsed from Excel files uploaded during initial setup
- Each year entry has: `year`, `name`, `location`, `participants`, `scores`, `winner`, `actualPicks`, `participantPicks` — and `missing:true` for 2018
- The `note` field has been removed from all entries — history speaks for itself via the data
- The scoring function `calcScore(userPicks, actualPicks, slots)` takes a slots array — always pass `pickSlots` from state
- Position normalization happens in `normPos()` — extend this if new position abbreviations appear
- The app uses `window.storage` not `localStorage` — these are different APIs in the Claude artifact environment
- Fonts load from Google Fonts CDN — the `useEffect` that appends the link tag handles this
- The logo is embedded as a base64 PNG data URI to avoid any external image requests
- Round 1 does not exist in 2026 — the Packers traded away their first-round pick. There is no auto-score. Do not add one back.
- Pick numbers (overall draft slot, e.g. #41) are display-only. They live on `actualPick.pickNumber` and are entered by the commissioner. They have zero effect on scoring.
