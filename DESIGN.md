# Gutey Guesser — Design System

## Aesthetic Direction

**Dark football ops room.** Think the feel of a war room on draft night — deep forest green, stadium gold accents, clean condensed type. The interface is serious but not corporate. It earns its dark palette through the subject matter, not trend-chasing.

The design avoids:
- Purple gradients, glassmorphism, neon
- Generic sans-serifs (Inter, Roboto)
- Overuse of emojis or icons as decoration
- Rounded-everything softness

The design embraces:
- High contrast gold on deep green
- Condensed display type for headers
- Tight information density — this is a scorecard, not a landing page
- Subtle borders rather than shadows

---

## Color Palette

All colors defined in the `G` object at the top of the component.

| Token | Hex | Usage |
|-------|-----|-------|
| `G.bg` | `#0A1A0C` | Page background — deepest green |
| `G.surface` | `#111E13` | Header, tab bar, inputs |
| `G.card` | `#182118` | Card backgrounds |
| `G.border` | `#243824` | All borders, dividers |
| `G.gold` | `#FFB612` | Primary accent — headings, active states, scores, winner highlights |
| `G.green` | `#1E5C33` | Button backgrounds, positive badge backgrounds |
| `G.midGreen` | `#2D6B3E` | Button borders |
| `G.lightGreen` | `#5BA877` | Position badges, commissioner text, success states |
| `G.text` | `#E8F0E9` | Primary body text |
| `G.muted` | `#7A9B80` | Secondary text, labels, placeholders |
| `G.dim` | `#465A4C` | Tertiary text, round labels, decorative elements |
| `G.red` | `#C0392B` | Error states, locked badges |

### Score status colors

Used in the scoring breakdown pills and history table highlights.

| Status | Color | Meaning |
|--------|-------|---------|
| `perfect` | `#FFB612` (gold) | Right player, right round |
| `player` | `#5BA877` (light green) | Right player, different round |
| `position` | `#5B9BD5` (blue) | Right position, right round |
| `partial` | `#9B7ECC` (purple) | Right position, different round |
| `miss` | `#4A5E4E` (dark muted) | No match |
| `pending` | `#3A4E3E` | Awaiting actual pick |
| `none` | `#2A3A2E` | No prediction entered |

---

## Typography

Three fonts from Google Fonts, loaded via `useEffect` link injection.

```
https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Barlow+Condensed:wght@400;600;700&family=Barlow:wght@300;400;500&display=swap
```

| Token | Family | Usage |
|-------|--------|-------|
| `T.heading` | Bebas Neue | Page titles, year numbers, score totals, section headers |
| `T.sub` | Barlow Condensed 600 | Tab labels, round labels, badge text, table headers, button text |
| `T.body` | Barlow | Body copy, player names, form inputs, notes |

### Size scale

| Use | Size |
|-----|------|
| Page title (header) | 20px Bebas Neue |
| Section title | 28px Bebas Neue |
| Year in history | 22px Bebas Neue |
| Score total | 24–26px Bebas Neue |
| Tab labels | 12px Barlow Condensed |
| Round labels, badges | 11–13px Barlow Condensed |
| Body text | 13–14px Barlow |
| Small/dim text | 10–12px Barlow or Barlow Condensed |

---

## Component Patterns

### Cards

All cards use the same base style from `s.card`:
```js
{
  background: G.card,
  border: `1px solid ${G.border}`,
  borderRadius: 10,
  padding: 16,
  marginBottom: 8,
}
```

Special-purpose variants:
- **Gold border card**: `borderColor: G.gold + "33"` — used for auto-scored R1 picks, invite code section
- **Error border card**: `borderColor: G.red + "55"` — used on pick rows with validation errors
- **Hit border card**: `borderColor: G.green + "66"` — used on draft board rows once a pick is entered

### Score pills

Inline colored badges for showing point values:
```js
s.pill(status) = {
  display: "inline-block",
  padding: "1px 7px",
  borderRadius: 4,
  fontSize: 11,
  fontWeight: 700,
  background: STATUS_COLORS[status] + "22",  // 13% opacity fill
  color: STATUS_COLORS[status],
  border: `1px solid ${STATUS_COLORS[status]}33`  // 20% opacity border
}
```

### Lock badges

Small status indicators for day/round lock state:
- Green (`#1E5C33` bg) when open
- Red (`#C0392B` bg) when locked

### Buttons

Three variants from `s.btn(variant)`:
- `"gold"`: `G.gold` background, dark text — primary action
- `"green"`: `G.green` background, light text — secondary/save actions
- `"ghost"`: transparent with border — cancel/dismiss actions

All buttons use `Barlow Condensed 700`, uppercase, `letterSpacing: 1`.

### Form inputs

Two variants:
- `s.inp(hasError)`: full-weight input for required fields (player, position). Red border if error state.
- `s.inpSoft`: same style but `color: G.muted` and subtler placeholder — for optional fields (school)

### Position badges

Small inline labels showing normalized position abbreviations:
```js
{
  background: G.surface,
  border: `1px solid ${G.border}`,
  borderRadius: 3–5,
  padding: "1–3px 5–8px",
  color: G.lightGreen,
  fontFamily: Barlow Condensed,
  fontSize: 10–12px,
}
```

---

## Layout

### Header (sticky, 54px)

Logo | Title + subtitle | [User name + score] or [Submit Picks button]

### Tab bar

Scrollable horizontal row of tab buttons. Active tab has `2px solid G.gold` bottom border. Tab labels are `Barlow Condensed`, uppercase.

Five tabs for participants: My Picks · Leaderboard · Draft Board · History · Admin  
Three tabs for public: Leaderboard · Draft Board · History

### Content area

`maxWidth: 700px`, centered with `margin: 0 auto`, `padding: 16px 14px`.

### Modal overlay

Used for the code entry and name entry flows — keeps the user on the current tab without a full-screen redirect:
```js
position: fixed, inset: 0
background: "#0A1A0Ccc"  // 80% opacity bg
display: flex, alignItems: center, justifyContent: center
zIndex: 200
```

Clicking the backdrop dismisses the modal.

---

## Admin Tab — Pick Slot Management

The top section of the admin tab (commissioner-only) manages pick slots before and during the draft:

**Slot list** — shows all current slots. Each has two actions:
- **Trade Away / Restore** — toggles `traded` flag. Traded slots show as "Pick traded away" to participants and are excluded from scoring and pick entry.
- **Remove** — permanently deletes the slot from the list.

**Add Pick Slot** — round selector + button. Generates a new slot with an auto-incremented id (e.g. a second Round 4 pick becomes `"4b"`). Slots are sorted by round then id after adding.

## Admin Tab — Actual Pick Entry

One card per active (non-traded) slot. Five fields per row:
- **#** (52px) — overall pick number, e.g. `41`. Display only, no scoring effect.
- **Player name** (flex) — required for scoring to register
- **Pos** (90px) — position selector
- **School** (90px) — optional
- **Save** (60px) — writes to shared storage immediately

The confirmed pick is shown inline in the card header once saved: `Player (#41) (POS)`.

Each year card shows in its collapsed state:
- **Year** (Bebas Neue, gold) + **game name** (Barlow Condensed, muted) + **location** (Barlow Condensed, dim) on one line
- Winner and score on the line below, or "Results not recovered" for 2018

No editorial notes or blurbs — the data tells the story.

## Draft Board

Each slot renders as a card. If `traded`, it shows at reduced opacity with "Pick traded away." Otherwise it shows the round label, overall pick number (if entered), player name, school, and position badge. Pending picks show an italicized "Pending" placeholder.

**No Round 1 in 2026** — the Packers traded away their first-round pick. There is no auto-score row anywhere in the app.

## History Tab — Expanded Year View

When a year card is expanded it shows two sections in order:

### Packers Picks
Always shown, even for 2018 (which has actual picks but no participant scores). Each row: `Rd N (#overall) Player POS School`.

### Final Scores (per-person expandable)
Each participant row shows rank, name, and total score. If that person has picks recorded, the row has a chevron and is clickable. Clicking expands an inline panel below their row showing each of their guesses with:
- Round predicted, player name, position
- Scoring outcome label (e.g. "Right round" / "Picked in Rd 3 (#84)") in the appropriate status color
- Point pill: gold = 15, green = 10, blue = 5, purple = 1, muted = 0

Only one person expands at a time. `expandedHistoryPerson` state tracks `"YEAR-Name"` as a string key. 2018 shows the Packers Picks section but no Final Scores (scores not recovered).

---

## Responsive behavior

The layout is single-column and mobile-first. No breakpoints — everything is fluid. Key patterns:
- `flexWrap: "wrap"` on multi-item rows (lock badges, all-time wins)
- `overflowX: "auto"` on the picks table in history
- Tab bar scrolls horizontally on small screens
- Input grids use `gridTemplateColumns: "1fr 120px"` (player + position) — collapses naturally

---

## Logo

The Packers "G" oval logo is embedded as a base64-encoded PNG with transparent background (background removed via edge flood-fill from the original JPEG). Sizes used:
- 30×30px in header
- 52–90px in modal/login views
