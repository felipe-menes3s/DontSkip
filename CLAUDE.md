# DontSkip — Fitness Tracking App

## Purpose
Mobile-first fitness tracking web app for personal use. Tracks workouts, meals, water intake, and body progress. Features **Jarvis**, an AI assistant tab with rule-based engine for coaching guidance. Full i18n support (ES/EN).

## Deployment
- **GitHub**: https://github.com/felipe-menes3s/DontSkip
- **Live URL**: https://felipe-menes3s.github.io/DontSkip/
- Deploy = `git push` (GitHub Pages auto-deploys from main branch)

## User Profile
- Felipe, 34 years old, 72kg, 1.74m, ectomorph (skinny-fat tendency)
- Goal: Body recomposition (lose abdominal fat + gain muscle)
- Gym: Building gym, Mon–Fri 6:05–7:05 AM, Utah timezone (MST/MDT)
- Units: Thinks in kg, gym equipment in lbs — app supports toggle
- Nutrition targets: 150g protein, 2,200 kcal/day, 9 glasses water (2,250 ml) — all editable via Perfil
- Supplementation: Whey post-gym (27g protein), creatine 5g daily, protein bar afternoon

## Architecture

### Stack
- **Frontend**: React 18 (CDN, no build tools) + htm (ES modules)
- **State**: Local state in App component + localStorage persistence for daily data
- **Styling**: Tailwind CSS (CDN) + inline styles via theme tokens (`T.*`)
- **Font**: **Outfit** (single font, all weights 400-900, Google Fonts CDN)
- **Icons**: Phosphor Icons React via `esm.sh` with `?deps=react@18.3.1`
- **No build step**: Single `index.html` (~6000 lines), dev server port **8085**
- **Exercise GIFs**: Local files in `gifs/` folder (24 exercises + cardio), no CDN dependency
- **AI**: Jarvis — local rule-based engine, 10 intents, bilingual keyword detection (no external API)
- **i18n**: `t()` function with EN dictionary (~200+ keys), `exName()`, `dayLabel()`, `LangToggle` component

### Design Language
- **Theme system**: Dual light/dark mode via `THEMES` object, global `let T` variable, `applyThemeCSS()` function
- **Light accent**: `#274472` (navy blue) / **Dark accent**: `#6b9ed6` (soft blue)
- **Color tokens**: T.bg, T.card, T.elevated, T.text, T.sub, T.muted, T.accent, T.accentDim, T.green, T.greenDim, T.amber, T.amberDim, T.copper, T.copperDim, T.water, T.waterDim, T.red, T.border, T.input
- **Style**: Premium minimalist, clean cards (`ds-card`), subtle shadows, no uppercase in Live
- **Emojis**: Medium-light skin tone 🏼 (U+1F3FC) for human emojis, light skin 🏻 (U+1F3FB) for body parts, male variants for athletes
- **Optimized for**: iPhone (safe areas, viewport units, touch targets)

### i18n System
- Global `LANG` variable ('es' or 'en'), mirrors `T` theme pattern
- `EN` dictionary object with ~200+ Spanish→English mappings
- `t(spanishKey)` — returns English when `LANG === 'en'`, returns key otherwise
- `exName(ex)` — uses `nameEn` field on exercise objects when English
- `dayLabel(key)` — translates day names (Lunes→Monday, etc.)
- `LangToggle` component on ALL tab headers
- All `.toLocaleDateString()` calls use `LANG`-aware locale
- Jarvis bilingual: `const L = LANG === 'en'` pattern in all intents
- **CRITICAL**: Avoid naming local variables `t` — shadows the global translation function (TDZ errors)

### Navigation — 5 Tabs (consolidated from 7 on 2026-03-23)
1. **Live** — Real-time workout session, timer, exercise tracking, mood check-in, 1RM badge per exercise
2. **Hoy** — Daily dashboard: animated StatCards, meals timeline, water counter
3. **Comidas** — Donut chart, food verifier, food library, meal builder
4. **Progreso** — Segmented control: Progreso (share card, photos, achievements) / Semana (calendar, muscle map, volume, catalog with 1RM)
5. **Jarvis** — Full-screen AI assistant tab (10 intents, 8 quick buttons)

Hidden: **Perfil** (accessed via gear icon ⚙️ in Hoy header) — settings, editable goals, editable weight, unit toggle, supplements, CSV export, notes, factory reset

### Key Files
- `index.html` — Full app (~6000 lines: CSS, React components, everything)
- `CLAUDE.md` — This file (project context for AI sessions)
- `gifs/` — 24 exercise GIFs + 1 cardio GIF (local, no CDN)
- `manifest.json` — PWA manifest (white background)
- `logo-icon.png`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` — PWA icons (white bg)

### localStorage Keys
- `ds_theme` — `'light'` or `'dark'`
- `ds_lang` — `'es'` or `'en'` (default `'es'`)
- `ds_streak` — `{ date, count }` (daily streak)
- `ds_workout_{todayKey}_{date}` — exercise states (sets, weights, reps, done, rpe, note)
- `ds_water_{date}` — water ml consumed
- `ds_water_{date}_log` — water addition history (for undo)
- `ds_meals_{date}` — meals with confirmed status
- `ds_workout_celebrated_{date}` — celebration modal shown flag
- `ds_records` — personal records per exercise `{ [id]: { weight, reps, date, unit } }`
- `ds_progress_photos` — base64 photos array (max 12)
- `ds_splash_shown` — sessionStorage, splash shown flag
- `ds_unit_pref` — 'kg' or 'lbs' (default 'lbs')
- `ds_unit_water` — 'ml' or 'floz'
- `ds_goal_protein` — editable protein goal (default 150)
- `ds_goal_calories` — editable calories goal (default 2200)
- `ds_goal_water_ml` — editable water goal in ml (default 2250)
- `ds_mood_{date}` — mood/energy emoji index for the day
- `ds_notes` — freeform notes (Perfil tab)
- `ds_food_library_custom` — user-added custom foods
- `ds_weight_log` — array of `{ date, kg }` entries (body weight history)
- `ds_profile_name` — user display name (default 'Felipe')
- `ds_creatine_{date}` — creatine supplement check
- `ds_whey_{date}` — whey supplement check
- `ds_workout_timer_{todayKey}_{date}` — workout start timestamp
- `ds_workout_paused_{date}` — paused elapsed seconds
- `ds_stats` — `{ startDate, totalDays, maxStreak, lastTrainDate }`
- `ds_warmup_{date}` — array of completed warmup exercise indices
- `ds_stretch_{date}` — array of completed stretching exercise indices

## Exercise Data Structure (in ROUTINES)
Each exercise: `{ id, name, nameEn, desc, sets, reps, muscle, gif, videoId }`
- `gif` — local path (e.g. `'gifs/bench_press.gif'`)
- `videoId` — YouTube video ID for tutorial link
- `nameEn` — English name (used by `exName()` when `LANG === 'en'`)

## Global Utilities
- `haptic(ms)` — `navigator.vibrate()` wrapper, accepts number or array pattern
- `foodIcon(name)` — regex-based keyword→emoji mapper for food items (30+ patterns, fallback 🍽️)
- `getGoal(key, def)` / `setGoal(key, val)` — localStorage goal persistence
- `useCountUp(target, duration)` — animated number hook (easeOutCubic)
- `formatVolume(v)` — formats kg/lbs volume with locale separators
- `estimate1RM(weight, reps)` — Epley formula: `weight × (1 + reps/30)`
- `calcCaloriesBurned(durationSec, bodyWeightKg)` — MET-based: `5.0 × weight × hours`
- `getUserWeightKg()` — latest body weight from weight log, default 72kg
- `t(key)` — i18n translation function
- `exName(ex)` — language-aware exercise name
- `dayLabel(key)` — language-aware day name

## Jarvis AI Assistant — 10 Intents
1. **food** — Food macro check against daily goals (green/yellow/red)
2. **missed_gym** — Encouragement + tomorrow's routine preview
3. **exercise_form** — Exercise details + description + PR + 1RM + YouTube link
4. **weight_suggest** — Per-exercise weight suggestions using actual PRs + 1RM percentages (70-80%)
5. **pain** — Safety advice for pain/injury
6. **supplements** — Daily supplement guidance (creatine + whey)
7. **motivation** — Random motivational message + current progress
8. **summary** — Full day summary (gym/protein/calories/water/streak/today's PRs)
9. **1rm** — All estimated 1RM values sorted by weight, with progressive overload tip
10. **streak** — Current streak, best streak, total days, consistency %, PR count

## Key Components
- **StatCard** — SVG progress ring (64px), Phosphor icon, useCountUp animation, sparkline, context line
- **FloatingTimer** — Global timer bar on all tabs (except Live) when workout active, play/pause/stop
- **MealBuilder** — Food search, quantity picker, custom food addition with emoji icons
- **ExerciseCatalog** — Expandable exercise browser by day with GIFs, YouTube links, and 1RM
- **WeeklyShareCard** — Week summary with navigator.share
- **CelebrationModal** — Confetti + trophy when all exercises completed (once per day)
- **LangToggle** — ES/EN language toggle button on all tab headers
- **ThemeToggle** — Light/dark mode toggle on all tab headers

## UI Patterns (CRITICAL)

### React + htm
- Uses `<script type="module">` with htm from esm.sh CDN
- **NEVER** escape backticks in htm templates
- **ALL hooks must be called before any early returns** — React error #310
- SVGs with `class` attribute must use `dangerouslySetInnerHTML`
- `window.onerror` creates overlay div outside React root for debugging
- **NEVER** name local variables `t` in scopes that call `t()` translation — causes TDZ errors
- Complex onClick handlers should be extracted to named functions (htm parser issues with inline arrow functions)

### Mobile Optimization
- Safe area insets: `env(safe-area-inset-top)`, `env(safe-area-inset-bottom)`
- Bottom tab bar height: 64px + safe area bottom
- Touch targets: minimum 44x44px
- `font-variant-numeric: tabular-nums` on body for consistent numbers

### CSS Animations
- fadeUp, tabEnter — tab transitions
- celebrate, celebrateIn — workout completion
- confettiFall — confetti particles
- splashFade, splashLogo, splashText — splash screen
- skeletonShimmer — GIF loading placeholder
- badgeUnlock — achievement unlock
- countPulse — number animation
- ring-circle — SVG stat ring stroke
- check-draw — checkmark draw
- live-dot — pulsing dot on Live tab
- slide-in-right — routine pill entrance
- pop-in — micro-feedback items
- glowPulse — accent glow effects
- typingDot, msgFadeIn, msg-enter — Jarvis chat

## Version Control
- **Git repo initialized 2026-03-23** after file destruction incident
- **GitHub repo + Pages 2026-03-24**
- CRITICAL: Always commit before running agents that edit `index.html`

## Planned Features (roadmap agreed 2026-03-24)
1. ~~1RM Estimator~~ ✅ Done
2. ~~Estimated calories burned per workout~~ ✅ Done (MET 5.0 × weight × hours)
3. Body measurements with interactive SVG body map + evolution charts
4. HIIT / interval timer (work/rest/rounds)

## Recent Features (2026-03-26)
- **Dynamic warmup** — 5 muscle-specific exercises before workout (per training day)
- **Post-workout stretching** — 5 targeted stretches after completing all exercises
- **Estimated calories burned** — MET-based calculation shown in Live tab metrics + post-workout
- **Fill sets button** — copies first set's weight/reps to remaining empty sets
- **Records fix** — saveRecord triggers on weight update for already-done sets

## Related Projects
- `enzo-hchb-sync` and `enzo-sync-ingestion` — same React+htm+single-HTML pattern
