# Guard the Sky

**A retro air-traffic-control typing game about emissions and air traffic.**

You are the controller on duty at a sustainability-focused airport. Two 737-class
aircraft are closing on each other in your sector. Read the instruction, type it
exactly, transmit it — before they collide. Do it 10 times per shift, across five shifts,
spend your credits wisely, and clock out with a climate grade.

One file. No build step. No dependencies beyond Google Fonts. Open
`guard-the-sky v3.html` in any modern browser and play.

---

## The Point of the Game

Guard the Sky is an **educational game combining airport laws and typing minigames**

> **Every second a player hesitates is a second an aircraft holds, burns fuel,
> and emits CO₂.**

It uses a simple core loop — read a radio instruction, type it correctly,
beat the clock — but the design is tied to an emissions model:

- **Fast, accurate typing saves simulated CO₂.** Response speed converts into
  "avoided holding time," which converts into kilograms of CO₂ not emitted.
- **Mistakes and missed deadlines cost CO₂.** A collision adds five simulated
  minutes of holding for *both* aircraft.
- **Upgrades are a green investment.** Spending credits on control systems makes
  your response windows more forgiving and reduces the emission factor — the
  game rewards investing in efficiency, not hoarding.
- **The final grade is a climate grade.** Your score blends accuracy, response
  time, and completion, so "greenest controller" and "best controller" are the
  same thing.

The fun comes from the pressure: a shrinking timer, converging radar blips, a
typing challenge that punishes sloppiness, and a 15-level upgrade curve that
makes you feel the airport getting greener under your hands.

---

## How to Play

1. **START CONTROL SHIFT** → the game enters a *standby* state.
2. Optionally **UPGRADE** your control systems (costs `UR 150`, max level 15).
3. Press ** PLAY** to receive the first instruction and start the timer.
4. **Type the instruction exactly** as shown and press **Enter** (or **TRANSMIT**).
   - Case does **not** matter.
   - Extra/missing whitespace does **not** matter.
   - Spelling and punctuation **do** matter.
5. Repeat for **10 rounds** to finish the shift. After shift 5, press **CLOCK OUT**
   for your overall report.

**Fail state:** you get **2 warnings per shift**. Each missed deadline is an airprox
event — the radar flashes red, the aircraft collide, and emissions spike. Two
collisions ends the session (credits and upgrades already earned are kept, and the
final report still shows what you accomplished).

**Anti-cheat:** paste, drag-and-drop, and copy/cut on the prompt are all disabled.
You have to actually type it.

---

## Features

### Core Gameplay

| Feature | Detail |
|---|---|
| **Typing challenge** | 32 unique ATC instructions spanning separation, efficiency, runway, landing, radio, and holding categories |
| **Case-insensitive input** | Submission is normalized (`trim` + collapse whitespace + `toUpperCase`) before comparison; the input field displays in caps |
| **Per-round timer** | Starts at 24 s, shrinks with round number, shift number, and upgrade level; floors at 9 s |
| **Speed bonus** | Credits scale with how fast you transmit: `25 + (timeLimit − response) × 2 + (upgrade − 1) × 5` |
| **Collision detection** | Timer expiry = collision event, with red screen flash, scaled `✖` marker, and emissions penalty |
| **Warning system** | 2 warnings per shift, each with a 1 s recovery pause and a fresh instruction |
| **Anti-cheat guards** | `paste`, `drop`, `copy`, `cut` all intercepted on the relevant elements |

### Progression

| Feature | Detail |
|---|---|
| **5 shifts** | `TOTAL_SHIFTS = 5` — shifts are the outer structure |
| **10 rounds per shift** | `ROUNDS_PER_SHIFT = 10` — 50 rounds per session |
| **Persistent credits** | Starting balance `UR 500`; earnings carry across all five shifts |
| **Upgrade system** | `MAX_UPGRADE = 15`, `UPGRADE_COST = 150` UR per level |
| **Upgrade effects** | Better timer floors (via `computeTimeLimit`), lower emission factor, higher credit gain per command |
| **Shift report** | Per-shift grade, rounds, accuracy, avg response, emissions, credits delta, upgrade level |
| **Clock Out report** | Overall grade (S/A/B/C/D/F), shifts cleared, overall accuracy, avg response, total CO₂ emitted/saved, final credits, rounds cleared X/50 |

### Radar & Visuals

| Feature | Detail |
|---|---|
| **Converging aircraft** | Two planes interpolate from random start positions toward the sector centre, driven by `progress = 1 − remain/timeLimit` |
| **Progressive conflict alert** | Radar flips from `SECTOR CLEAR` (green `✓`) to `CONFLICT ALERT` (red `!`) at 50% progress |
| **Collision animation** | `crashFlash` keyframes pulse red into the radar; collision ring scales to 1.5× with a red `✖` |
| **Live labels** | Flight-ID tags track each aircraft's interpolated position |
| **Radar sweep** | Pure-CSS conic-gradient sweep, radial rings, crosshairs, and N/E/S/W bearings |
| **Purple galaxy theme** | Deep-space backgrounds, purple/magenta/pink accent ramp, violet radar glow, pixel grid overlay |
| **Retro typography** | `Press Start 2P` for headings and UI labels, `VT323` for body text |
| **Three overlays** | Start modal, per-shift report, session/Clock Out report |

### The Emissions Model

```js
const FUEL_BURN_KG_PER_HOUR  = 2400;  // 737-class cruise fuel burn
const JET_FUEL_CO2_FACTOR    = 3.16;  // ICAO kg CO₂ per kg jet fuel
const SIM_MIN_PER_REAL_SEC   = 1;     // 1 real second = 1 simulated flight minute
const AIRCRAFT_COUNT         = 2;
