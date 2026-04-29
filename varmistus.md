# Koti HIIT — Lovable Build Prompt

Build a complete HIIT workout timer app called "Koti HIIT" in React + TypeScript + Tailwind CSS. The app is a single-page application with no routing needed — screen state is managed with a single useState variable.

---

## DESIGN SYSTEM

Dark theme, near-black background (#0a0a0a). Primary accent color: red #E53935 (hsl ~3 83% 55%).
Font: Inter (Google Fonts). All text in Finnish.
CSS variables in index.css following shadcn/ui token conventions:
  --background: 0 0% 4%
  --foreground: 0 0% 96%
  --card: 0 0% 7%
  --primary: 3 83% 55%
  --primary-foreground: 0 0% 100%
  --secondary: 0 0% 11%
  --muted: 0 0% 14%
  --muted-foreground: 0 0% 50%
  --border: 0 0% 14%
  --radius: 0.5rem

---

## APP SCREENS

Four screens, controlled by a `screen` state variable: 'SETUP' | 'MOVEMENTS' | 'LIVE' | 'FINISHED'

---

## SCREEN 1: SetupScreen

Header: "HIIT TRAINER" label (primary color, small caps), h1 "Konfiguroi treeni", subtitle "Aseta intervallit ja pisteet ennen aloitusta".

Settings card (dark card, border border-white/[0.07]):
- Työ (s): segmented control, options [30, 45, 60], default 45
- Lepo (s): segmented control, options [15, 30, 45, 60], default 15
- Liikkeet / piste: segmented control, options [3, 4, 5, 6], default 4
- Kierrokset: segmented control, options [1, 2, 3, 4], default 3
- Vaihda pistettä (s): segmented control, options [45, 60, 90], default 60
Dividers between rows.

SegmentedControl: row of buttons, active = bg-primary text-white, inactive = bg-white/[0.05] text-muted-foreground. Pill/rounded-lg shape, font-bold, small text.

Below the card: 3-column station toggles grid.
Stations: JUOKSU (Activity icon, "Juoksu", sublabel "Juoksumatto"), PAINOT (Dumbbell icon, "Painot", sublabel "Käsipainot"), KEHONPAINO (PersonStanding icon, "Kehonpaino", sublabel "Oma keho").
Active station: full border-primary bg-primary/10 with icon in primary color. Inactive: border-white/[0.09] bg-white/[0.03].
At least 1 station must always remain active.

Below stations: a small strip showing "Arvioitu kesto" + "~X min" calculated from settings, and a "Historia" button (History icon) that opens a modal with saved sessions.

CTA button: full-width, bg-primary, "Siirry liikkeisiin" (Play icon). When loading shows spinner + "Luodaan treeniä...".

---

## PLAN GENERATION (generateWorkoutPlan)

First attempt AI via LLM API (see integration section). If AI fails, fall back to local shuffle.

Finnish AI prompt: Ask for a workout plan for the active stations with these rules:
- JUOKSU station: generate treadmill intervals varying between walking (6-8 km/h), light jog (9-11), run (12-14), sprint (15-16). Sprint speeds (15-16 km/h) ALWAYS have incline 0%. Other speeds can have incline 1-10%. Format: "Vauhti X km/h, kaltevuus X%"
- Other stations: concrete, safe exercises in Finnish.
Return JSON with station keys (exactly "JUOKSU", "PAINOT", "KEHONPAINO") mapping to arrays of `intervals` strings each.

Local exercise pools (Finnish):

JUOKSU: ["Vauhti 6 km/h, kaltevuus 4%", "Vauhti 7 km/h, kaltevuus 6%", "Vauhti 8 km/h, kaltevuus 8%", "Vauhti 8 km/h, kaltevuus 10%", "Vauhti 9 km/h, kaltevuus 5%", "Vauhti 10 km/h, kaltevuus 4%", "Vauhti 10 km/h, kaltevuus 6%", "Vauhti 11 km/h, kaltevuus 3%", "Vauhti 11 km/h, kaltevuus 5%", "Vauhti 12 km/h, kaltevuus 2%", "Vauhti 12 km/h, kaltevuus 4%", "Vauhti 13 km/h, kaltevuus 2%", "Vauhti 13 km/h, kaltevuus 3%", "Vauhti 14 km/h, kaltevuus 1%", "Vauhti 14 km/h, kaltevuus 0%", "Vauhti 15 km/h, kaltevuus 0%", "Vauhti 16 km/h, kaltevuus 0%"]

PAINOT: ["Goblet kyykky", "Penkkipunnerrus käsipainoilla", "Pystypunnerrus käsipainoilla", "Kulmasoutu käsipainoilla", "Askelkyykky käsipainoilla", "Hauiskääntö käsipainoilla", "Ranskalainen punnerrus", "Vipunostot sivuille", "Vipunostot taakse", "Maastaveto käsipainoilla", "Pystysoutu käsipainoilla", "Rinnalleveto käsipainoilla", "Tempaus käsipainoilla", "Lantionnosto käsipainolla", "Ojentajapotku käsipainoilla"]

KEHONPAINO: ["Burpee", "Vuorikiipeilijä", "Haarahypyt", "Kyykkyhypyt", "Punnerrukset", "Lankku", "Sivulankku", "Jalkojen nostot", "Venäläinen kierto", "Mittarimato", "Luistelijahypyt", "Polvennostojuoksu", "Kantapäät pakaroihin", "Linkkuveitset", "Karhukävely"]

---

## SCREEN 2: MovementsScreen

Back button "Takaisin asetuksiin" (ArrowLeft icon) at top.
Header: "TÄMÄN PÄIVÄN TREENI" label, h1 "Liikkeet", subtitle "Käy läpi liikkeet — muokkaa tai generoi uudelleen tarvittaessa".

For each active station render a StationSection card:
  - Header: station icon + name (UPPERCASE, tracking-widest) + "X liikettä" count on the right. If cycles > 1: "Toistetaan X kierrosta" subtext.
  - List of ExerciseRow items for each exercise:
    - Row number (dim), colored dot, exercise name.
    - On hover: show pencil (edit) button and refresh (regenerate) button.
    - Edit mode: inline text input with Enter=save, Escape=cancel, check/X buttons.
    - Regenerate: calls AI to get a new different exercise, falls back to local pool avoiding duplicates.

Below station sections: WorkoutSummary strip showing ~X min, Työ Xs, Lepo Xs, Liikkeet N, Kierrokset N.

Fixed bottom button: "Aloita treeni" (Play icon), full-width, bg-primary.

---

## SCREEN 3: LiveScreen — CRITICAL TIMER LOGIC

### Settings object shape:
```
{ work: number, rest: number, intervals: number, cycles: number, switchTime: number, activeStations: string[] }
```

### Plan object shape:
```
{ JUOKSU: string[], PAINOT: string[], KEHONPAINO: string[] }
```

### Timeline builder (buildTimeline) — exact logic, do not deviate:

Phases enum: PREP | WORK | REST | SWITCH
PREP_DURATION = 15 seconds

Build an ordered array of segment objects:

**1. Push one PREP segment** (duration: 15s, station: null, all indexes null).

**2. Loop over stations** (si = 0..stationCount-1):

  Loop over cycles (ci = 0..cycles-1):

    Loop over exercises (ei = 0..intervals-1):

      currentExercise = plan[station][ei] ?? "Liike N"

      nextExercise logic:
      - if ei < intervals-1: nextExercise = plan[station][ei+1] (next in same cycle)
      - else if ci < cycles-1: nextExercise = plan[station][0] (loop back — next cycle starts over)
      - else: nextExercise = null (last exercise of last cycle on this station)

      isVeryLast = (si === last station) && (ci === last cycle) && (ei === last exercise)

      Push **WORK** segment: `{ phase:'WORK', duration:work, station, stationIndex:si, totalStations, cycleIndex:ci+1, totalCycles:cycles, exerciseIndex:ei, totalExercises:intervals, currentExercise, nextExercise }`

      Push **REST** segment **ONLY IF NOT isVeryLast**: same fields but phase:'REST', duration:rest

    end exercise loop

  end cycle loop

  Push **SWITCH** segment **ONLY IF si < stationCount-1**: `{ phase:'SWITCH', duration:switchTime, station:null, nextStation: activeStations[si+1], fromStation: activeStations[si], stationIndex:si, ... }`

**3. Return** the timeline array.

Each segment has a unique id (e.g. "seg-0", "seg-1"...).

---

### Timer hook (useTimer):

State: `{ segmentIndex, timeLeft, running, paused, finished, segment }`
Uses setInterval (1000ms) stored in a ref. Uses a ref-based tick function to avoid stale closures.

- **start(timeline)**: initialize, set segment 0, start interval, fire onSegmentChange(seg0)
- **tick()**: if timeLeft > 1 → decrement, fire onTick(newTime, seg). If timeLeft === 1 → fire onTick(1, seg), advance to next segment.
- **advanceToSegment(index)**: if index >= timeline.length → clearInterval, setState finished, fire onFinish(). Else set new segment, fire onSegmentChange.
- **pause()**: clearInterval, set paused=true, running=false
- **resume()**: restart interval, set paused=false, running=true
- **stop()**: clearInterval, reset all state
- Callbacks (onSegmentChange, onTick, onFinish) stored in a ref so they never become stale.

onSegmentChange fires: `beepTransition()` for any phase that is NOT PREP.
onTick fires: `beepTick()` when timeLeft <= 3.
onFinish fires: `beepFinish()`, `releaseWakeLock()`, navigate to FINISHED screen.

---

### LiveScreen UI:

**Top thin progress bar** (3-4px): fills left-to-right showing `(phaseDuration - timeLeft) / phaseDuration`. Color: primary for WORK, white/30 for REST, white/20 for SWITCH/PREP. Disable CSS transition on segment change to avoid reverse animation.

**Top bar row:**
- Left: Pause/Play toggle button (icon only, small rounded square button)
- Center: StatusBar — shows phase label (WORK=primary color, REST=white, PREP/SWITCH=muted) + cycle and exercise counters (e.g. "K 2/3  L 3/4") during WORK/REST
- Right: X exit button

**Large phase timer (PhaseTimer):** centered, huge font (8xl-9xl responsive), tabular numbers. Color: primary for WORK, white for REST, muted for PREP/SWITCH. Drop shadow glow for WORK phase.

**Content area below timer:**
- PREP phase: centered text "Valmistaudu" + dim description of session (X pistettä · X kierrosta · X liikettä/piste)
- SWITCH phase: centered rotating arrow ↻ + bold "Vaihda pistettä!" text
- WORK/REST phase: render ALL active stations as StationCard components in a scrollable list

**StationCard per station:**
- Header: station icon + name label + cycle/exercise counters on right
- During WORK: exercise name in large bold white text (font-bold, text-xl)
- During REST: exercise name in dimmer text (text-white/60)
- If nextExercise exists: "SEURAAVA" label + next exercise name in small dim text (pt-3, border-t)
- Card border: primary/30 + subtle red glow shadow during WORK, dim white border during REST

**PauseOverlay:** fullscreen backdrop (bg-black/80 blur), centered card with "Jatka", "Aloita alusta", "Lopeta" buttons. "Jatka" is primary, others are ghost.

---

### Audio (Web Audio API, no external library):

`initAudio()`: create/resume AudioContext (call on first user gesture).

`beepTick()`: fires on each of the last 3 countdown seconds. Square wave + sine oscillator at 880Hz, sharp attack, fast decay (~0.22s). Same pitch every time (loud, gym-style).

`beepTransition()`: two layered "bell" sounds at 523Hz and 784Hz (slight offset), using sine partials + noise transient for a metallic bell quality. Fires on every phase change except PREP.

`beepFinish()`: ascending fanfare — 5 bells at pitches [523, 659, 784, 1046, 1318] Hz with increasing delays [0, 0.25, 0.5, 0.8, 1.15]s.

**WakeLock:** call `navigator.wakeLock.request('screen')` on workout start, release on finish/exit.

---

## SCREEN 4: FinishedScreen

Canvas confetti animation (120 gold/yellow particles, stops after 5s).
Trophy emoji in a glowing circle.
"SUORITETTU" label (yellow-400), h1 "Treeni ohi!", subtitle "Hienoa työtä. Palaudu rauhassa ja syö hyvin."
Two buttons: "Toista treeni" (RotateCcw, primary) and "Takaisin alkuun" (Home, ghost).

---

## WORKOUT HISTORY (WorkoutHistory)

Saved sessions stored in Supabase. Table: `workout_sessions`. Fields:
- `date`: string (ISO)
- `settings`: jsonb
- `plan`: jsonb
- `duration_mins`: number
- `favourite`: boolean

Trigger: a "Historia" button (History icon) in SetupScreen opens a bottom sheet / modal.
Modal header: "Treenihistoria" + X close.
Filter tabs: "Kaikki" / "⭐ Suosikit".

**SessionCard:**
- Date formatted in Finnish (day. Month year)
- ~X min duration
- Setting pills: "Työ Xs", "Lepo Xs", "X liikettä", "X kierr.", "Vaihto Xs"
- Station icons + names
- Footer: star toggle ("Lisää suosikiksi" / "Suosikki" in yellow) + "Lataa treeni →" button (primary color)

Loading a session: sets settings + plan from saved session, navigates directly to MovementsScreen (skip generation).
If session has no plan saved: regenerate using generateWorkoutPlan.

Save new session on MovementsScreen → LiveScreen transition with: date, settings, plan, duration_mins (ceil(calcTotalDuration/60)), favourite: false.

---

## LLM INTEGRATION

Use OpenAI (or your chosen provider) for AI plan generation.
The function should accept `{ prompt, response_json_schema }` and return a parsed JSON object matching the schema.
Used in two places:
1. `generateWorkoutPlan` — generates a full plan for all active stations
2. `regenerateSingleExercise` — generates one replacement exercise for a given station, avoiding duplicates

---

## COMPONENT STRUCTURE

Keep components small and focused:

```
HIITApp.tsx                  — main orchestrator, all screen state + handlers
SetupScreen.tsx
SegmentedControl.tsx
StationToggle.tsx
MovementsScreen.tsx          — ExerciseRow, StationSection, WorkoutSummary inline or as sub-files
LiveScreen.tsx
TopProgressBar.tsx
StatusBar.tsx
PhaseTimer.tsx
StationCard.tsx
PauseOverlay.tsx
FinishedScreen.tsx           — GoldConfetti canvas component
WorkoutHistory.tsx           — SessionCard as sub-component

lib/hiit/constants.ts        — STATIONS, PHASES, SCREENS, PHASE_LABELS, PREP_DURATION, DEFAULT_SETTINGS, TREADMILL, WEIGHTS, BODYWEIGHT, EXERCISE_POOLS
lib/hiit/timeline.ts         — buildTimeline, calcTotalDuration, calcEstimatedDuration
lib/hiit/planGenerator.ts    — generateWorkoutPlan, regenerateSingleExercise
lib/hiit/useTimer.ts
lib/hiit/useAudio.ts
lib/hiit/useWakeLock.ts
```

---

## CRITICAL CORRECTNESS REQUIREMENTS

1. **Timeline logic must be EXACTLY as described.** Especially:
   - REST is omitted ONLY after the absolute last WORK segment of the entire workout (last station, last cycle, last exercise).
   - nextExercise loops back to `exercises[0]` when a cycle ends but more cycles remain.
   - SWITCH only appears between stations, never after the last station.
   - cycleIndex is 1-based (`ci+1`) in segment data.

2. **StationCard shows ALL active stations simultaneously** during WORK and REST — not just the "current" station. Every station shows its own current exercise based on `exerciseIndex` from the segment.

3. **Timer uses ref-based interval** (not state-based) to avoid stale closure bugs. Callbacks always read from a ref.

4. **Audio context must be initialized on first user interaction** (`initAudio` called on "Siirry liikkeisiin" button click before async operations).

5. **beepTick fires on timeLeft values 3, 2, and 1** (checked in `onTick` callback: `if (timeLeft <= 3)`).

6. **"Aloita alusta" (restart) rebuilds the same timeline** and calls `timer.start()` again — it does NOT regenerate exercises.
