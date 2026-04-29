# Koti HIIT – Tekninen kuvaus Lovable-siirtoa varten

## 1. Projektin yleiskatsaus

Koti HIIT on verkko- ja mobiilisovellus korkeatehoiseen intervalliharjoitteluun (HIIT). Se tarjoaa:
- Mukautettavat harjoitusasetukset (työ/lepo/kierrokset/pisteet)
- Tekoälyllä (LLM) generoidut harjoitusliikkeet eri pisteille
- Reaaliaikainen ajastin ääni-ilmoituksilla ja visuaalisella palautteella
- Harjoitushistorian tallennus ja suosikkien hallinta
- Täysin responsiivinen, mobiilioptimtu tumma UI

---

## 2. Teknologiapino

| Kerros | Teknologia |
|--------|-----------|
| Kehys | React 18.2.0 |
| Kieli | JavaScript (JSX) – ei TypeScriptiä |
| Tyylit | Tailwind CSS + shadcn/ui |
| Rakennustyökalu | Vite |
| Reititys | react-router-dom v6 |
| Datanhaku | @tanstack/react-query v5 |
| Backend | Base44 BaaS (korvattava Lovablessa) |
| Tietokanta | Base44 Entities (korvattava) |
| Todennus | Base44 Auth (korvattava) |
| LLM | Base44 Core.InvokeLLM (korvattava OpenAI/Gemini/Claude API:lla) |

---

## 3. Visuaalinen suunnittelu

### Väripaletti (Tailwind / CSS-muuttujat)

```css
:root {
  --background: 0 0% 4%;           /* Erittäin tumma tausta */
  --foreground: 0 0% 96%;          /* Lähes valkoinen teksti */
  --primary: 3 83% 55%;            /* Voimakas punainen – CTA-painikkeet */
  --primary-foreground: 0 0% 100%;
  --secondary: 0 0% 11%;
  --secondary-foreground: 0 0% 80%;
  --muted: 0 0% 14%;
  --muted-foreground: 0 0% 50%;
  --card: 0 0% 7%;
  --card-foreground: 0 0% 96%;
  --border: 0 0% 14%;
  --input: 0 0% 14%;
  --ring: 3 83% 55%;
  --radius: 0.5rem;
}
```

### Typografia
- **Fontti:** Inter (Google Fonts, painot 300–900)
- **Tyyli:** Tiivistä, kapiteeli-teksteillä (`tracking-[0.2em] uppercase`) otsikoissa
- `font-variant-numeric: tabular-nums` ajastimissa

### Yleiset UI-patternit
- Kortit: `bg-white/[0.03] border border-white/[0.07] rounded-xl`
- Painikkeet (CTA): `bg-primary font-black text-sm tracking-[0.15em] uppercase rounded-xl shadow-[0_0_32px_rgba(229,57,53,0.25)]`
- Ikonit: `lucide-react`
- Animaatiot: CSS transitions + `canvas-confetti` (FinishedScreen)
- `safe-area-inset-bottom` mobiililaitteiden turva-alueiden huomioimiseksi

---

## 4. Tiedostorakenne

```
src/
├── api/
│   └── base44Client.js         ← SDK-alustus (KORVATTAVA)
├── components/
│   ├── hiit/
│   │   ├── SetupScreen.jsx     ← Asetusnäyttö
│   │   ├── MovementsScreen.jsx ← Liikkeiden esikatselu ja muokkaus
│   │   ├── LiveScreen.jsx      ← Reaaliaikainen harjoitusnäyttö
│   │   ├── FinishedScreen.jsx  ← Harjoituksen loppunäyttö + konfetti
│   │   ├── WorkoutHistory.jsx  ← Historia-modaali (lista + suosikit)
│   │   ├── PauseOverlay.jsx    ← Taukovalikko
│   │   ├── PhaseTimer.jsx      ← Iso ajastinnäyttö
│   │   ├── TopProgressBar.jsx  ← Ohut edistymispalkki ylhäällä
│   │   ├── StatusBar.jsx       ← Vaiheen tila (WORK/REST/SWITCH)
│   │   ├── StationCard.jsx     ← Pistekortti LiveScreenissä
│   │   ├── StationToggle.jsx   ← Pisteiden valintapainike SetupScreenissä
│   │   └── SegmentedControl.jsx← Segmentoitu valintapainike asetuksille
│   └── ui/                     ← shadcn/ui-komponentit
├── lib/
│   ├── hiit/
│   │   ├── constants.js        ← Vakiot: STATIONS, PHASES, SCREENS, harjoituslistat
│   │   ├── planGenerator.js    ← LLM-pohjainen harjoitussuunnitelman generointi
│   │   ├── timeline.js         ← Aikajanan rakentaminen + keston laskenta
│   │   ├── useTimer.js         ← Ajastinhook (start/pause/resume/stop)
│   │   ├── useAudio.js         ← Ääniefektit (Web Audio API)
│   │   └── useWakeLock.js      ← Näytön hereillä pitäminen
│   ├── AuthContext.jsx         ← Todennuskonteksti (KORVATTAVA)
│   ├── PageNotFound.jsx
│   └── query-client.js
├── pages/
│   └── HIITApp.jsx             ← Päänäkymä, näyttöjen orchestrointi
├── App.jsx                     ← Router + AuthProvider + QueryClientProvider
├── index.css                   ← Globaalit tyylit, CSS-muuttujat, fontit
├── tailwind.config.js
└── main.jsx
```

---

## 5. Sovelluslogiikka ja näyttöjen kulku

```
SetupScreen → (generoi plan) → MovementsScreen → (build timeline) → LiveScreen → FinishedScreen
     ↑                                                                                  |
     └──────────────────────────────── onHome ─────────────────────────────────────────┘
     ↑
WorkoutHistory (lataa session) → MovementsScreen (suoraan, plan valmiina)
```

### Näyttöjen tilat (`SCREENS`-vakio)
```js
SCREENS = { SETUP: 'setup', MOVEMENTS: 'movements', LIVE: 'live', FINISHED: 'finished' }
```

---

## 6. Harjoituksen asetukset (settings-objekti)

```js
DEFAULT_SETTINGS = {
  work: 45,           // työn kesto sekunteina
  rest: 15,           // levon kesto sekunteina
  intervals: 4,       // liikkeiden määrä per piste per kierros
  cycles: 2,          // kierrosten määrä
  switchTime: 60,     // pisteen vaihtoaika sekunteina
  activeStations: ['juoksu', 'painot', 'kehonpaino']
}
```

### Vaihtoehdot UI:ssä
```js
WORK_OPTIONS    = [30, 45, 60]
REST_OPTIONS    = [15, 30, 45, 60]
INTERVAL_OPTIONS = [3, 4, 5, 6]
CYCLE_OPTIONS   = [1, 2, 3, 4]
SWITCH_OPTIONS  = [45, 60, 90]
```

---

## 7. Harjoituspisteet (STATIONS)

```js
STATIONS = {
  JUOKSU: 'juoksu',           // Juoksumatto / juoksu
  PAINOT: 'painot',           // Painot / käsipainot
  KEHONPAINO: 'kehonpaino'    // Kehonpainoharjoittelu
}
```

Jokainen piste näytetään omalla ikonilla:
- Juoksu → `Activity` (lucide)
- Painot → `Dumbbell` (lucide)
- Kehonpaino → `PersonStanding` (lucide)

---

## 8. Harjoitusvaiheet (PHASES)

```js
PHASES = {
  PREP: 'prep',       // 10s valmistautumisvaihe alussa
  WORK: 'work',       // Työvaihe
  REST: 'rest',       // Lepovaihe
  SWITCH: 'switch',   // Pisteen vaihtovaihe
  FINISH: 'finish'    // Harjoituksen loppu
}
```

Vaiheiden värit LiveScreenissä:
- WORK → punainen (`text-primary`)
- REST → sinertävä (`text-blue-400`)
- SWITCH → keltainen (`text-yellow-400`)
- PREP → harmaa (`text-muted-foreground`)

---

## 9. Aikajanan rakenne (timeline.js)

`buildTimeline(settings, plan)` palauttaa segmenttilistan:

```js
[
  { phase: 'prep', duration: 10 },
  // Jokaiselle kierrokselle:
  //   Jokaiselle liikkeelle:
  { phase: 'work', duration: 45, station: 'juoksu', exerciseIndex: 0, cycleIndex: 0, ... },
  { phase: 'rest', duration: 15, station: 'juoksu', exerciseIndex: 0, ... },
  //   (viimeisen liikkeen jälkeen ei lepoa)
  // Jokaisen pisteen jälkeen:
  { phase: 'switch', duration: 60, fromStation: 'juoksu', nextStation: 'painot' },
  // ... toistetaan kaikille pisteille ja kierroksille
  { phase: 'finish' }
]
```

---

## 10. Harjoitussuunnitelman generointi (planGenerator.js)

### LLM-prompti (suomeksi)
```
Luo harjoitussuunnitelma seuraaville pisteille: [pisteet].
Jokaiselle pisteelle {intervals} liikettä.
Palauta JSON-objekti muodossa: { "juoksu": ["liike1", "liike2", ...], "painot": [...], ... }
Liikkeiden tulee olla sopivia kotitreeniin.
```

### Fallback (jos LLM epäonnistuu)
Lokaalit harjoituslistat constants.js:ssä:
```js
JUOKSU_EXERCISES   = ['Juoksu', 'Sprintit', 'Sivujuoksu', 'Takaperin juoksu', ...]
PAINOT_EXERCISES   = ['Hauis-käännös', 'Ojentaja-punnerrus', 'Maastaveto', ...]
KEHONPAINO_EXERCISES = ['Burpee', 'Punnerrus', 'Kyykky', 'Askelkyykky', ...]
```

---

## 11. Ajastin (useTimer.js)

```js
// Tila
{ segmentIndex: 0, timeLeft: 45, paused: false, segment: { phase, duration, ... } }

// Metodit
timer.start(timeline)   // Käynnistää uuden aikajanan
timer.pause()           // Keskeyttää
timer.resume()          // Jatkaa
timer.stop()            // Pysäyttää ja nollaa

// Callbackit (välitetään hookille)
onSegmentChange(segment) // Kutsutaan kun vaihe vaihtuu
onTick(timeLeft, segment) // Kutsutaan joka sekunti
onFinish()               // Kutsutaan harjoituksen lopussa
```

---

## 12. Ääniefektit (useAudio.js – Web Audio API)

```js
initAudio()        // Alustaa AudioContextin (kutsuttava käyttäjäinteraktion yhteydessä)
beepTick(timeLeft) // Lyhyt piip viimeisten 3 sekunnin aikana
beepTransition()   // Kaksi lyhyttä ääntä vaiheiden välillä
beepFinish()       // 5-nuottinen fanfare harjoituksen päättyessä
```

---

## 13. Tietokantaentiteetti: WorkoutSession

### Schema
```json
{
  "name": "WorkoutSession",
  "properties": {
    "date":        { "type": "string", "description": "ISO-päivämäärä harjoituksen aloitusajankohdasta" },
    "settings":    { "type": "object", "description": "Harjoituksen asetukset (work, rest, intervals, cycles, switchTime, activeStations)" },
    "plan":        { "type": "object", "description": "Generoitu harjoitussuunnitelma { station: [exercises] }" },
    "durationMins":{ "type": "number", "description": "Arvioitu kesto minuutteina" },
    "favourite":   { "type": "boolean", "default": false }
  },
  "required": ["date", "settings"]
}
```

### CRUD-operaatiot (Base44 → korvattava)
```js
// Luo
await db.workoutSessions.create({ date, settings, plan, durationMins, favourite: false })

// Listaa (uusimmat ensin)
await db.workoutSessions.list({ orderBy: 'date', direction: 'desc', limit: 50 })

// Päivitä (esim. suosikki)
await db.workoutSessions.update(id, { favourite: true })
```

---

## 14. WorkoutHistory-komponentti

- Avautuu modaalina (koko ruutu, rounded-t-2xl mobiilissa / centered desktop)
- Suodattimet: "Kaikki" ja "Suosikit"
- Jokainen sessiokortti näyttää: päivämäärä, kesto, asetuspillerit (työ/lepo/liikkeet/kierrokset/vaihto), aktiiviset pisteet ikoneilla
- Toiminnot: "Lisää suosikiksi" (star-ikoni) + "Lataa treeni" (lataa asetukset + plan → siirtyy MovementsScreeniin)

---

## 15. Todennus

Nykyinen toteutus perustuu Base44:n `AuthContext` ja `useAuth` hookiin. Lovablessa korvattava omalla tai kolmannen osapuolen ratkaisulla (esim. Supabase Auth, Clerk, Firebase Auth).

Nykyinen autentikaatiovirtojen käsittely:
```js
const { isLoadingAuth, authError, navigateToLogin } = useAuth();
// authError.type === 'user_not_registered' → näytä UserNotRegisteredError
// authError.type === 'auth_required' → ohjaa kirjautumissivulle
```

---

## 16. Kriittiset riippuvuudet (package.json)

### Korvattava Lovablessa
| Paketti | Syy |
|---------|-----|
| `@base44/sdk` | Koko backend (DB, auth, LLM) |
| `@base44/vite-plugin` | Base44-kohtainen Vite-lisäosa |

### Siirtyy sellaisenaan
| Paketti | Käyttö |
|---------|--------|
| `@tanstack/react-query ^5` | Datanhaku ja välimuisti |
| `react-router-dom ^6` | Reititys |
| `lucide-react ^0.475` | Ikonit |
| `canvas-confetti ^1.9.4` | Konfettianimaatio FinishedScreenissä |
| `tailwindcss-animate` | Animaatiot |
| `clsx` + `tailwind-merge` | Luokkien yhdistely |
| `date-fns ^3` | Päivämäärän muotoilu |
| `framer-motion ^11` | (asennettu, vähäinen käyttö) |

---

## 17. Siirtostrategia

### Vaihe 1: Frontend (siirtyy lähes sellaisenaan)
Kaikki `components/hiit/`-komponentit ja `lib/hiit/`-moduulit ovat alustariippumattomia ja voidaan kopioida suoraan.

### Vaihe 2: Backend-abstraktio (vaatii uudelleenkirjoituksen)
Luo uusi `api/`-kerros joka korvaa `base44Client.js`:n. Tarvittavat toiminnot:
```js
// Tietokanta
createWorkoutSession(data)
listWorkoutSessions(limit)
updateWorkoutSession(id, data)

// LLM
generateExercisePlan(stations, intervalsPerStation) → { station: [exercises] }
regenerateSingleExercise(station, existingExercises) → string

// Todennus
getCurrentUser()
login()
logout()
```

### Vaihe 3: Ympäristömuuttujat
```env
VITE_LLM_API_KEY=...        # OpenAI / Gemini / Claude API-avain
VITE_DB_URL=...             # Tietokantayhteys
VITE_AUTH_SECRET=...        # Todennussalaisuus
```

---

## 18. Suositus

> **"Muokattava ennen siirtoa"**
>
> Frontend-koodi on erittäin siirrettävissä. Päätyö on Base44-backend-integraatioiden korvaaminen.
> Rakenna Lovablessa uusi `api/`-kerros ja kopioi kaikki UI- ja logiikkamoduulit sellaisenaan.
