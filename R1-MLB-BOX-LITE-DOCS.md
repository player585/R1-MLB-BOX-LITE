# R1 MLB BOX LITE — Project Documentation
> Complete knowledge base for the Rabbit R1 MLB Live Scores Creation
> Version: 1.2.0 | Last updated: 2026-04-08 12:43 PM PDT

---

## Project Overview

**R1 MLB BOX LITE** is a production-ready Rabbit R1 Creation that displays live MLB scores, inning-by-inning grids, and full box scores — all powered by the free MLB Stats API (no API key required). It's a single self-contained HTML file (`index.html`) designed for the R1's 240×282px portrait screen with all hardware controls wired through the official R1 Creations SDK event system.

### Repository
- **GitHub:** https://github.com/player585/R1-MLB-BOX-LITE
- **Live URL (GitHub Pages):** https://player585.github.io/R1-MLB-BOX-LITE/
- **Install:** Scan the QR code (`R1-MLB-BOX-LITE-QR.png`) with the R1's rabbit eye camera

### File Structure
```
R1-MLB-BOX-LITE/
├── index.html                  ← The entire app (single file, HTML/CSS/JS)
├── R1-Creations-Guide-full.md  ← How to build/host/install R1 Creations
├── R1-MLB-BOX-LITE-QR.png      ← Scannable QR code for R1 device install
├── R1-MLB-BOX-LITE-DOCS.md     ← This file
├── .nojekyll                   ← Prevents GitHub Pages Jekyll processing
└── README.md                   ← Repo readme
```

---

## Rabbit R1 Device — Hardware Specs & Constraints

### Screen
- **Resolution:** 240×282 pixels, portrait orientation
- **Viewport meta:** `<meta name="viewport" content="width=240, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">`
- Dark mode preferred (AMOLED-like screen, dark backgrounds save power and look sharp)

### Performance Constraints
- Very limited hardware — write optimized code
- Use hardware-accelerated CSS properties (`transform`, `opacity`)
- Minimize DOM operations
- Limit particle effects
- Use CSS transitions instead of JavaScript animations
- Only one creation can run at a time
- No large file storage

### Hardware Inputs Available
| Input | Description |
|-------|-------------|
| Scroll wheel | Physical wheel on the side, fires custom JS events |
| PTT side button | Push-to-talk button on the side, fires custom JS events |
| Touchscreen | Standard touch/click events work normally |
| Rotating camera ("rabbit eye") | Used for QR scanning, camera access via standard web APIs |
| Microphone | Standard web audio APIs |
| Speaker | Standard web audio + R1 TTS via SDK |
| Accelerometer | 3-axis, accessed via `window.creationSensors` SDK API |

---

## R1 Creations SDK — Hardware Event Reference

**CRITICAL:** The R1 does NOT fire standard keyboard events (`keydown`, `keyup`, `keypress`). The scroll wheel and PTT button dispatch **custom window events**. Standard keyboard listeners (`Enter`, `Space`, `ArrowUp`, etc.) will NOT work on the R1 device — they only work in browser testing.

### Side Button (PTT) Events
```javascript
// Single click
window.addEventListener("sideClick", () => {
  console.log("Side button clicked");
});

// Long press start
window.addEventListener("longPressStart", () => {
  console.log("Side button long press started");
});

// Long press end
window.addEventListener("longPressEnd", () => {
  console.log("Side button long press ended");
});

// Note: Double click triggers two sideClick events ~50ms apart
```

### Scroll Wheel Events
```javascript
// Scroll up (wheel rotated up/away)
window.addEventListener("scrollUp", () => {
  console.log("Scroll wheel up");
});

// Scroll down (wheel rotated down/toward)
window.addEventListener("scrollDown", () => {
  console.log("Scroll wheel down");
});
```

### Important: `scrollBy()` Does Not Work Reliably on R1
The R1 WebView does not reliably propagate `element.scrollBy()`. Instead, use:
- **Focus-navigation pattern:** Track a `focusIndex`, highlight elements with a CSS class, and call `element.scrollIntoView({ block: 'nearest', behavior: 'auto' })` to keep the focused item visible
- **Direct `scrollTop` manipulation:** `el.scrollTop = el.scrollTop + 60` works as a fallback

### Accelerometer API
```javascript
// Check availability
const isAvailable = await window.creationSensors.accelerometer.isAvailable();

// Start (normalized -1 to 1: x = tilt right/left, y = forward/back, z = up/down)
window.creationSensors.accelerometer.start((data) => {
  console.log(data.tiltX, data.tiltY, data.tiltZ);
  console.log(data.rawX, data.rawY, data.rawZ); // m/s²
}, { frequency: 60 }); // Hz

// Stop
window.creationSensors.accelerometer.stop();
```

### Storage API
```javascript
// Plain storage (unencrypted, isolated per plugin ID)
await window.creationStorage.plain.setItem('key', btoa(JSON.stringify(value)));
const val = JSON.parse(atob(await window.creationStorage.plain.getItem('key')));
await window.creationStorage.plain.removeItem('key');
await window.creationStorage.plain.clear();

// Secure storage (hardware-encrypted, Android M+)
await window.creationStorage.secure.setItem('key', btoa('secret'));
const secret = atob(await window.creationStorage.secure.getItem('key'));
// All data MUST be Base64 encoded before storage
```

### Plugin Messaging (LLM Integration)
```javascript
// Send message to server
PluginMessageHandler.postMessage(JSON.stringify({
  message: "Hello from my creation"
}));

// Request LLM response (received via window.onPluginMessage)
PluginMessageHandler.postMessage(JSON.stringify({
  message: "Tell me a fun baseball fact. Return JSON: {'fact': '...'}",
  useLLM: true
}));

// LLM speaks through R1 speaker + logs to journal
PluginMessageHandler.postMessage(JSON.stringify({
  message: "Summarize today's scores",
  useLLM: true,
  wantsR1Response: true,
  wantsJournalEntry: true
}));

// Receive messages from server
window.onPluginMessage = function(data) {
  if (data.data) {
    const parsed = JSON.parse(data.data);
    // Use parsed response
  }
};
```

### Other SDK APIs
```javascript
// Close creation and return to R1 home screen
closeWebView.postMessage("");

// Simulate touch events
TouchEventHandler.postMessage(JSON.stringify({
  type: "tap", // or "down", "up", "move", "cancel"
  x: 100,
  y: 200
}));
```

---

## MLB Stats API — Endpoints Used

All endpoints are **free, no API key required**, and served from `https://statsapi.mlb.com/api/v1`.

### Schedule + Linescore
```
GET /schedule?sportId=1&date=YYYY-MM-DD&hydrate=linescore(matchup,runners),flags
```
Returns today's games with embedded linescore data (inning-by-inning runs, current inning, outs, runners, team scores).

**Key response paths:**
- `dates[0].games[]` — array of game objects
- `game.status.abstractGameCode` — `"L"` (live), `"F"` (final), `"P"` (preview/scheduled)
- `game.teams.away/home.score` — current score
- `game.linescore.innings[]` — inning-by-inning breakdown
- `game.linescore.currentInning` — current inning number
- `game.linescore.isTopInning` — boolean
- `game.linescore.outs` — current outs (0-3)
- `game.venue.name` — stadium name

### Box Score
```
GET /game/{gamePk}/boxscore
```
Returns full box score for a specific game.

**Key response paths:**
- `teams.away/home.batters[]` — array of player IDs (batting order)
- `teams.away/home.pitchers[]` — array of player IDs (pitching order)
- `teams.away/home.players.ID{playerId}` — player data:
  - `.person.fullName` — player name
  - `.position.abbreviation` — position (SS, CF, P, etc.)
  - `.stats.batting` — game stats: `atBats`, `runs`, `hits`, `rbi`, `homeRuns`, `baseOnBalls`, `strikeOuts`
  - `.seasonStats.batting` — season stats: `avg`, `obp`, `slg`
  - `.stats.pitching` — game stats: `inningsPitched`, `hits`, `runs`, `earnedRuns`, `baseOnBalls`, `strikeOuts`, `homeRuns`, `pitchesThrown`, `strikes`
  - `.seasonStats.pitching` — season stats: `era`
- `decisions.winner/loser/save` — W/L/SV pitcher decisions

---

## App Architecture

### State Object
```javascript
let state = {
  games: [],              // Raw game data from API
  selectedGamePk: null,   // Currently selected game ID
  focusIndex: -1,         // Scores tab: which game card has focus (scroll wheel)
  boxFocusIndex: -1,      // Box Score tab: which player row has focus
  theme: 'dark',          // 'dark' or 'light'
  countdown: 30,          // Seconds until next auto-refresh
  countdownTimer: null,    // Interval ID
  currentTab: 'scores'    // 'scores' or 'boxscore'
};
```

### R1 Control Scheme (Current — v1.2.0)

**Scores Tab:**
| R1 Input | Action |
|----------|--------|
| Scroll wheel up/down | Move amber focus ring between game cards, auto-scrolls into view |
| PTT click (sideClick) | Select focused game → load box score → switch to Box Score tab |
| PTT hold (longPressStart) | Switch to Box Score tab |
| Tap game card | Select that game (touchscreen still works) |

**Box Score Tab:**
| R1 Input | Action |
|----------|--------|
| Scroll wheel up/down | Move amber highlight row-by-row through all player stat rows |
| PTT click (sideClick) | Refresh the current box score |
| PTT hold (longPressStart) | Switch back to Scores tab |
| PTT release (longPressEnd) | Refresh all scores |

**Both Tabs:**
| R1 Input | Action |
|----------|--------|
| Tap refresh button | Full manual refresh |
| Tap theme toggle | Switch dark/light mode |

**Browser Testing Fallbacks (keyboard):**
| Key | Action |
|-----|--------|
| Enter / Space | Refresh scores |
| Arrow Down/Up | Scroll main panel 60px |
| Arrow Right/Left | Switch tabs |

### Focus-Navigation Pattern
The R1 WebView doesn't reliably support `scrollBy()`. Instead, the app uses a **focus-index pattern**:

1. `state.focusIndex` tracks which game card is highlighted
2. `scrollUp`/`scrollDown` increments/decrements the index (clamped to valid range)
3. The focused card gets a `.focused` CSS class (amber border glow)
4. `scrollIntoView({ block: 'nearest', behavior: 'auto' })` keeps it visible
5. Same pattern for box score player rows via `state.boxFocusIndex`

### Auto-Refresh
- 30-second cycle with red countdown bar at footer
- On cycle completion, calls `loadAll()` to re-fetch schedule
- If a box score is loaded, it refreshes too
- Focus position is preserved across refreshes

### UI Components
- **Header:** MLB logo SVG + title + live dot (pulsing amber) + theme toggle + refresh button
- **Status bar:** Current date + game count / live count
- **Nav tabs:** Scores | Box Score (red active indicator)
- **Game cards:** Team abbreviation + full name + score + inning indicator + inning grid
- **Box score tables:** Batting (AB/R/H/RBI/HR/BB/K/AVG/OBP/SLG) + Pitching (IP/H/R/ER/BB/K/HR/PC-ST/ERA)
- **Decisions row:** W/L/SV chips
- **Footer:** API label + countdown bar + seconds remaining
- **Version watermark:** Fixed bottom-right, 45% opacity, non-interactive

### Design Tokens (Dark Theme)
| Token | Value | Usage |
|-------|-------|-------|
| `--color-bg` | `#0d0f0e` | Page background |
| `--color-surface` | `#131615` | Card/header/footer backgrounds |
| `--color-primary` | `#c8102e` | MLB red — active tabs, HR highlights, selected cards |
| `--color-win` | `#22c55e` | Winning score, RBI highlights |
| `--color-loss` | `#ef4444` | (Reserved) |
| `--color-live` | `#f59e0b` | Live badges, current inning, focused card/row highlight |
| `--color-sched` | `#5591c7` | Scheduled badges, AVG/OBP/SLG/ERA values |
| `--color-text` | `#e8ebe9` | Primary text |
| `--color-text-muted` | `#7a8078` | Secondary text |
| `--color-text-faint` | `#4a4f4d` | Tertiary text, column headers |

### Typography
| Token | Size | Usage |
|-------|------|-------|
| `--text-xs` | 0.5625rem (9px) | Badges, status bar, table headers, table data |
| `--text-sm` | 0.625rem (10px) | Team abbreviations, nav tabs, player names |
| `--text-base` | 0.6875rem (11px) | Body text |
| `--text-md` | 0.8125rem (13px) | Team scores |
| Fonts | Inter (body), JetBrains Mono (data/mono) | |

### Team Abbreviation Map
All 30 MLB teams mapped to 2-3 letter codes:
```
ATH BAL BOS CWS CLE DET HOU KCR LAA MIN NYY SEA TBR TEX TOR
ARI ATL CHC CIN COL LAD MIA MIL NYM PHI PIT SDP SFG STL WSN
```

---

## QR Code Format (R1 Install)

The R1 scans a QR code containing a JSON payload:
```json
{
  "title": "MLB Live",
  "url": "https://player585.github.io/R1-MLB-BOX-LITE/",
  "description": "Live MLB scores, box scores & inning grids — no API key needed",
  "iconUrl": "",
  "themeColor": "#c8102e"
}
```

Generated with Python `qrcode` library, error correction level L, styled with Rabbit orange (#FE5000) border and "r1 creations" label.

### Installing on R1
1. Open the **Creations card** on R1 (scroll through card stack)
2. Tap **Create** tab → **Add via QR code**
3. Point rabbit eye camera at QR code (6–12 inches)
4. Auto-installs — scroll to bottom of card stack to find it
5. Creation loads fresh from GitHub Pages URL every time (no caching)

---

## Hosting & Deployment

### GitHub Pages (Free)
- Repository: `player585/R1-MLB-BOX-LITE` (public)
- Branch: `main`, root folder
- `.nojekyll` file prevents Jekyll processing of JS
- Deploys automatically on push (~1–2 minutes)
- Live URL: `https://player585.github.io/R1-MLB-BOX-LITE/`

### Update Workflow
1. Edit `index.html` locally or via push
2. Bump version watermark (`vX.Y.Z` + date + time)
3. `git push origin main`
4. GitHub Pages rebuilds in 1–2 minutes
5. On R1: close and reopen the creation to load latest version (no cache)

---

## Version History

| Version | Date | Time | Changes |
|---------|------|------|---------|
| v1.0.0 | 2026-04-08 | 12:09 PM | Initial push — full MLB live scores, box scores, inning grids, 30s auto-refresh, dark/light theme, keyboard controls |
| v1.0.1 | 2026-04-08 | 12:24 PM | Wire R1 SDK events (`scrollUp`, `scrollDown`, `sideClick`, `longPressStart`), viewport → `width=240`, scale all UI for 240×282px screen |
| v1.0.2 | 2026-04-08 | 12:30 PM | Focus-navigation: scroll wheel moves amber ring between game cards, PTT selects game, `scrollIntoView` replaces broken `scrollBy` |
| v1.1.0 | 2026-04-08 | 12:36 PM | Add version/date watermark (bottom-right, 45% opacity) |
| v1.1.1 | 2026-04-08 | 12:37 PM | Add timestamp to watermark |
| v1.2.0 | 2026-04-08 | 12:38 PM | Scroll wheel navigates player rows on Box Score tab (amber highlight, row-by-row, auto-scroll) |

---

## Known Issues & Gotchas

1. **R1 WebView does not fire keyboard events** — always use SDK custom events (`scrollUp`, `scrollDown`, `sideClick`, `longPressStart`, `longPressEnd`)
2. **`scrollBy()` unreliable on R1** — use `scrollIntoView()` or direct `scrollTop` manipulation
3. **`scroll-behavior: smooth` may lag on R1** — use `behavior: 'auto'` for instant scrolls
4. **240px width is tight** — stat tables need horizontal scroll wrappers; keep font sizes at 9–11px for data
5. **Google Fonts load over network** — first load may be slow on R1's connection; fonts are not cached across sessions
6. **`color-mix(in oklch, ...)` CSS** — works in modern WebView but verify R1's Android WebView version supports it
7. **No offline support** — requires network for both MLB API and GitHub Pages hosting
8. **Auto-refresh continues even when no games are live** — could optimize to reduce API calls on off-days

---

## Future Enhancement Ideas

- **Offline fallback:** Cache last-known scores in `creationStorage.plain`
- **Team favorites:** Let user mark favorite teams, show those first
- **Push-to-talk game summary:** Use `PluginMessageHandler` + LLM to read a spoken game summary through R1 speaker
- **Accelerometer easter egg:** Shake to refresh
- **Yesterday/Tomorrow navigation:** PTT double-click to browse other days
- **Standings tab:** Add a third tab for division/league standings
- **Player detail popover:** PTT click on a focused box score row to show expanded stats
- **Reduce API calls:** Only auto-refresh when games are Live, extend to 5min when all Final/Scheduled

---

## R1 Creations — General Knowledge

### Three Ways to Build
| Method | Cost | Skill Level |
|--------|------|-------------|
| Speak to R1 (Intern agent) | 3 free tasks, then ~$2.33-$10/task | None |
| SDK + AI code gen + GitHub Pages + QR | Free | Basic |
| SDK + manual coding + GitHub Pages + QR | Free | Developer |

### SDK Repository
- **URL:** https://github.com/rabbit-hmi-oss/creations-sdk
- **Key folders:**
  - `plugin-demo/` — template showing all R1 hardware features
  - `plugin-demo/reference/creation-triggers.md` — full SDK event/API docs
  - `plugin-demo/js/hardware.js` — hardware event handler examples
  - `qr/` — browser-based QR code generator tool

### Creations Gallery
- **Browse:** https://rabbit.tech/creations
- **On device:** Creations card → Public tab
- Anyone with an R1 can scan a QR code and install for free

### Key Design Rules
1. Always design for **240×282px portrait**
2. Always test at that exact size before deploying
3. Use **dark theme** as default (R1 screen looks best dark)
4. Wire **all** hardware inputs through SDK custom events, not keyboard events
5. Keep JS lightweight — R1 hardware is limited
6. Use `hardware-accelerated` CSS (transform, opacity) for animations
7. Single HTML file preferred — fewer network requests on R1's connection
