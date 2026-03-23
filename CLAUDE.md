# DontSkip — Fitness Tracking App

## Purpose
Mobile-first fitness tracking web app for personal use. Tracks workouts, meals, water intake, and body progress. Features **Jarvis**, an intelligent local assistant with rule-based engine for coaching guidance.

## User Profile
- Felipe, 34 years old, 72kg, 1.74m, ectomorph (skinny-fat tendency)
- Goal: Body recomposition (lose abdominal fat + gain muscle)
- Gym: Building gym, Mon–Fri 6:05–7:05 AM, Utah timezone (MST/MDT)
- Units: Thinks in kg, gym equipment in lbs — app supports toggle
- Nutrition targets: 150g protein, 2,200 kcal/day, 9 glasses water (2,250 ml)
- Supplementation: Whey post-gym (27g protein), creatine 5g daily, protein bar afternoon

## Architecture

### Stack
- **Frontend**: React 18 (CDN, no build tools) + htm (ES modules) — same pattern as enzo projects
- **State**: Local state in App component + localStorage persistence for daily data
- **Styling**: Tailwind CSS (CDN) + inline styles via theme tokens
- **Fonts**: Inter (headings: 700/800/900) + Plus Jakarta Sans (body: 400-800), Google Fonts CDN
- **No build step**: Single HTML file, `python3 -m http.server 8080` or Firebase Hosting
- **AI**: Jarvis — local rule-based engine with keyword intent detection (no external API)

### Design Language
- **Theme system**: Dual light/dark mode via `THEMES` object, global `let T` variable, `applyThemeCSS()` function
- **Light accent**: `#274472` (navy blue)
- **Dark accent**: `#6b9ed6` (soft blue)
- **Palette**: Each theme has 20+ tokens: bg, card, elevated, text, sub, muted, accent, green, amber, red, water, etc.
- **Style**: Minimalista premium, clean cards (`ds-card` class), subtle shadows
- **Optimized for**: iPhone (safe areas, viewport units, touch targets)
- **Animations**: fadeUp, tabEnter, slideUp (Jarvis sheet), celebrate, glowPulse, typingDot, msgFadeIn
- **Icons**: Emoji for stat cards + Lucide-style SVG inline for navigation/actions

### Navigation — 5 Tabs + Jarvis FAB
1. **Hoy** (Today) — Daily dashboard: stat rings, workout card, meals timeline, water counter ✅
2. **Semana** (Week) — Placeholder
3. **Comidas** (Meals) — Meal tracking, food verifier, food library ✅
4. **Progreso** (Progress) — Placeholder
5. **Perfil** (Profile) — Placeholder
6. **Jarvis FAB** — Floating button → bottom sheet with AI assistant ✅

### Key Files
- `index.html` — Full app (~2750 lines: CSS, React components, everything in one file)
- `CLAUDE.md` — This file (project context for AI sessions)

### localStorage Keys
- `ds_theme` — `'light'` or `'dark'` (persists theme preference)
- `ds_streak` — `{ date, count }` (daily streak tracker)
- `ds_workout_{todayKey}_{date}` — exercise states (sets, weights, reps, done) per day
- `ds_water_{date}` — water ml consumed per day
- `ds_meals_{date}` — meals with confirmed status per day

## Components Implemented

### Theme System
- `THEMES` constant with `light` and `dark` palettes (20+ tokens each)
- `getInitialTheme()` — reads from localStorage, defaults to light
- `applyThemeCSS(mode)` — sets CSS variables on `:root` for ds-card, ds-check, ds-input, etc.
- `ThemeToggle({ dark, onToggle })` — sun/moon icon button in header

### Today Tab (`TodayTab`)
- Header: greeting by time of day + "Felipe" in Inter 30px/800, date, routine pill
- `DashboardStats` — 2x2 grid of `StatCard` components (protein 🥩, calories 🔥, water 💧, gym 🏋️)
- `StatCard` — SVG ring (52px) with emoji center + label + value/max
- `WorkoutCard` — full routine with exercise items, set tracking, rest timer, cardio GIF
- `ExerciseItem` — expandable card with animated GIF (ExerciseDB), muscle SVG icon, sets grid (weight/reps/checkbox), rest timer, YouTube tutorial link
- `RestTimer` — 90s circular countdown timer between sets
- `MealsTimeline` — vertical timeline of daily meals with confirm/edit
- `MealItem` — individual meal with macro breakdown, confirm button, edit mode
- `WaterCounter` — SVG glass with animated fill (celeste #38bdf8), water drop indicators, quick-add buttons (150/250/500/750 ml), custom amount input with ml/oz toggle

### Comidas Tab (`ComidasTab`)
- Three sections: Hoy (today's meals), Verificador (food checker), Biblioteca (food library)
- `FoodVerifier` — search food by name, shows calorie/protein semaphore (green/amber/red)
- `FoodLibrary` — browsable food database with search, add custom foods
- `FOOD_LIBRARY_DEFAULT` — 14 pre-loaded foods with protein/cal data

### Jarvis (`JarvisFAB`)
- **UI**: Bottom sheet (70vh), slides up with animation, backdrop blur
- **Header**: "J" avatar in accent gradient, "Jarvis", "Tu asistente de fitness", close button
- **Welcome message**: Dynamic greeting by time of day + brief day summary
- **Quick buttons**: 2x3 grid (🍕 ¿Puedo comer algo? / 💪 ¿Qué peso uso hoy? / 🏃 No pude ir al gym / 📖 Explícame un ejercicio / 💊 Suplementos de hoy / 📊 ¿Cómo voy hoy?) — hidden after first message
- **Chat**: User bubbles (right, accent color), Jarvis bubbles (left, elevated bg, "J" avatar)
- **Loading**: 3 animated dots (600ms simulated delay)
- **Auto-scroll**: scrolls to bottom on new messages
- **Reset**: History clears when sheet closes

#### Jarvis Engine (local, no API)
- `FOOD_LIBRARY` — ~31 foods with full macros (cal, protein, carbs, fat)
- `STARTING_WEIGHTS` — real weights for all 24 exercises (kg + lbs)
- `MOTIVATION_MSGS` — 6 personalized messages (August goal, baby, consistency)
- `buildDayContext()` — gathers real-time data: routine, exercises done, protein, calories, water
- `detectIntent(text)` — keyword-based intent classification (8 intents)
- `jarvisRespond(intent, text, ctx)` — generates contextual responses

#### Intent Detection (priority order)
| Intent | Triggers | Response |
|--------|----------|----------|
| `food` | Any FOOD_LIBRARY key in text | Semaphore 🟢🟡🔴 based on remaining cal/protein |
| `missed_gym` | "no pude", "no fui", "salté"... | Active rest reassurance + tomorrow's routine |
| `exercise_form` | "cómo hago", "explícame"... | Exercise description + sets/reps + YouTube link |
| `weight_suggest` | "qué peso", "peso inicio"... | Table of today's exercises with starting weights |
| `pain` | "me duele", "dolor", "lesión"... | Safety advice, DOMS vs acute pain |
| `supplements` | "creatina", "whey"... | Protocol by day type (training vs rest) |
| `motivation` | "cansado", "no tengo ganas"... | Random motivational + progress data |
| `summary` | Default fallback | Full day summary with encouragement |

### Placeholder Tabs
- `PlaceholderTab({ name, icon, dark, onThemeToggle })` — used for Semana, Progreso, Perfil

### App Component
- Holds all state: theme, tab, waterMl, exerciseStates, meals, unitPref
- Passes props down to all child components
- No context/reducer — flat prop drilling
- localStorage auto-save for exerciseStates, waterMl, meals (keyed by date)

### Exercise Data Structure (in ROUTINES)
Each exercise object has: `id`, `name`, `nameEn`, `desc`, `sets`, `reps`, `muscle`, `imageUrl` (ExerciseDB GIF), `videoId` (YouTube)

## UI Patterns (CRITICAL)

### React + htm (same as enzo projects)
- Uses `<script type="module">` with htm from esm.sh CDN
- **NEVER** escape backticks in htm templates
- **ALL hooks must be called before any early returns** — React error #310
- SVGs with `class` attribute must use `dangerouslySetInnerHTML`
- `window.onerror` creates overlay div outside React root

### Mobile Optimization
- `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, user-scalable=no">`
- `<meta name="apple-mobile-web-app-capable" content="yes">`
- Safe area insets: `env(safe-area-inset-top)`, `env(safe-area-inset-bottom)`
- Bottom tab bar height: 64px + safe area bottom
- Jarvis FAB: `bottom: calc(76px + var(--safe-bottom))`
- Touch targets: minimum 44x44px
- No hover-dependent interactions

### Tailwind via CDN
- `<script src="https://cdn.tailwindcss.com"></script>`
- Custom config: Plus Jakarta Sans as default sans font
- Use utility classes inline, avoid custom CSS when possible

### CSS Classes
- `.ds-card` — card with bg, border-radius 20px, border, shadow, theme transitions
- `.ds-check` — custom checkbox with green check animation
- `.fade-up` / `.tab-enter` / `.stagger` — entrance animations
- `.jarvis-sheet` / `.msg-enter` / `.typing-dot` — Jarvis animations
- `.no-scrollbar` — hidden scrollbar for chat areas
- `.bottom-spacer` — spacer for content above tab bar

## Workout Routines

### Weekly Split
| Day | Routine | Focus |
|-----|---------|-------|
| Monday | Upper A | Pecho + Hombros + Bíceps (strength) |
| Tuesday | Lower A | Cuádriceps + Glúteos + Core (strength) |
| Wednesday | Descanso | Recuperación activa |
| Thursday | Upper B | Espalda + Tríceps (hypertrophy) |
| Friday | Lower B | Posterior + Glúteos + Core (hypertrophy) |
| Saturday | Cardio | LISS o HIIT (opcional) |
| Sunday | Descanso | Descanso total |

### Upper A (Monday) — Pecho + Hombros + Bíceps
| # | Exercise | Sets x Reps | Starting Weight |
|---|----------|-------------|-----------------|
| 1 | Press Pecho (Bench Press) | 4x8-10 | 20kg / 44lbs |
| 2 | Press Inclinado (Incline DB Press) | 4x10 | 16kg / 35lbs |
| 3 | Press Militar (Overhead Press) | 3x10 | 14kg / 31lbs |
| 4 | Elevaciones Laterales (Lateral Raises) | 3x12 | 6kg / 13lbs |
| 5 | Curl Bíceps (Bicep Curl) | 3x12 | 10kg / 22lbs |
| 6 | Curl Martillo (Hammer Curl) | 3x12 | 10kg / 22lbs |

### Lower A (Tuesday) — Cuádriceps + Glúteos + Core
| # | Exercise | Sets x Reps | Starting Weight |
|---|----------|-------------|-----------------|
| 1 | Sentadilla (Squat) | 4x8 | 20kg / 44lbs |
| 2 | Prensa Piernas (Leg Press) | 4x10 | 45kg / 99lbs |
| 3 | Extensión Cuádriceps (Leg Extension) | 3x12 | 22kg / 48lbs |
| 4 | Curl Femoral (Leg Curl) | 3x12 | 22kg / 48lbs |
| 5 | Elevación Pantorrillas (Calf Raises) | 4x15 | 20kg / 44lbs |
| 6 | Plancha (Plank) | 3x45s | Peso corporal |

### Upper B (Thursday) — Espalda + Tríceps
| # | Exercise | Sets x Reps | Starting Weight |
|---|----------|-------------|-----------------|
| 1 | Jalón al Pecho (Lat Pulldown) | 4x10 | 30kg / 66lbs |
| 2 | Remo Polea (Seated Cable Row) | 4x10 | 28kg / 62lbs |
| 3 | Remo Mancuerna (Dumbbell Row) | 3x10 c/l | 12kg / 26lbs |
| 4 | Vuelos Posteriores (Rear Delt Fly) | 3x15 | 6kg / 13lbs |
| 5 | Extensión Tríceps (Tricep Pushdown) | 3x12 | 15kg / 33lbs |
| 6 | Press Francés (Skull Crushers) | 3x10 | 10kg / 22lbs |

### Lower B (Friday) — Posterior + Glúteos + Core
| # | Exercise | Sets x Reps | Starting Weight |
|---|----------|-------------|-----------------|
| 1 | Peso Muerto (Deadlift) | 4x6 | 40kg / 88lbs |
| 2 | Sentadilla Búlgara (Bulgarian Split Squat) | 3x10 c/l | 10kg / 22lbs |
| 3 | Curl Femoral Sentado (Seated Leg Curl) | 3x12 | 22kg / 48lbs |
| 4 | Hip Thrust | 3x10 | 20kg / 44lbs |
| 5 | Elevación Pantorrillas (Calf Raises) | 4x15 | 40kg / 88lbs |
| 6 | Crunch | 3x20 | Peso corporal |

### Cardio (Saturday)
- Option A: 30 min LISS (incline walk, bike)
- Option B: 20 min HIIT (intervals)
- Option C: Active recovery (stretching, mobility)

### Progression Rule
Increase weight when all reps completed with good form. Prioritize progressive overload. Form before weight.

## Meal Plan Template

### Office Days (Mon/Wed/Fri) — ~2,100 kcal, ~155g protein
- **7:05 AM** — Whey Post-Gym (130 cal, 27g P)
- **9:30 AM** — Pan + Jamón + Leche Nurri (480 cal, 24g P)
- **12:30 PM** — Pollo + Arroz + Ensalada (620 cal, 44g P)
- **3:30 PM** — Protein Bar Built (200 cal, 17g P)
- **7:00 PM** — Pollo + Ensalada (510 cal, 37g P)

### Packed Days (Tue/Thu) — ~1,800 kcal, ~140g protein
- **7:05 AM** — Whey Post-Gym (130 cal, 27g P)
- **9:30 AM** — Pan + Jamón (240 cal, 18g P)
- **12:30 PM** — Ensalada con Atún (420 cal, 32g P)
- **3:30 PM** — Protein Bar Built (200 cal, 17g P)
- **7:00 PM** — Cena variable (600 cal, 35g P)

### Rest Days (Sat/Sun) — ~1,900 kcal, ~110g protein
- **9:30 AM** — Desayuno (460 cal, 25g P)
- **12:30 PM** — Almuerzo (550 cal, 34g P)
- **3:30 PM** — Merienda (250 cal, 15g P)
- **7:00 PM** — Cena (400 cal, 28g P)

## Development Phases

### Phase 1 — Foundation + Today Tab ✅ COMPLETE
- Navigation shell (5 tabs, bottom bar)
- Today tab: stat rings, workout card with exercise tracking, meals timeline, water counter
- Dark mode with toggle (light/dark themes)
- UI polish: Inter headings, SVG water glass, custom stat cards
- Local state only, kg/lbs toggle

### Phase 2 — Comidas Tab ✅ COMPLETE
- Comidas tab with three sections: Hoy, Verificador, Biblioteca
- Food verifier with calorie/protein semaphore
- Food library with search and custom food addition
- Meal confirm/edit functionality

### Phase 3 — Jarvis AI ✅ COMPLETE
- Local rule-based engine (no external API)
- 8 intent detection with keyword matching
- FOOD_LIBRARY (31 foods), STARTING_WEIGHTS (24 exercises), MOTIVATION_MSGS
- Bottom sheet UI (70vh) with quick buttons, chat bubbles, loading animation
- Context-aware responses using real-time day data

### Phase 3.5 — Exercise Media + Persistence ✅ COMPLETE
- Animated anatomical GIFs from ExerciseDB CDN (`static.exercisedb.dev/media/{id}.gif`)
- Each exercise has `imageUrl` (GIF) and `videoId` (YouTube) in ROUTINES
- GIF display: 200x200px, `objectFit: contain` (native resolution 180x180)
- YouTube tutorial button per exercise (direct link via videoId)
- Cardio day shows treadmill GIF; rest days show mountain SVG (non-cardio only)
- localStorage persistence: workout progress, water, meals — all keyed by date, auto-reset daily

### Phase 4A — Tab Semana (next)
- Weekly adherence view
- Daily completion indicators
- Volume tracking
- Streak display

### Phase 4B — Tab Progreso
- Weight/measurement tracking
- Progress photos
- Strength progression charts
- Body composition estimates

### Phase 5 — Tab Perfil + Settings
- Settings, goals configuration
- Unit converters
- Data export

### Phase 6 — Firebase Persistence
- Firebase auth (anonymous or Google)
- Firestore CRUD for workouts, meals, water
- Real-time sync
- Offline support

### Phase 7 — Polish Final
- PWA (service worker, install prompt)
- Notifications
- Performance optimization

## Data Model (Future — Firebase)

### Collections
```
users/{userId}
  - profile: { name, age, weight, height, bodyType, goal, unitPref }
  - settings: { darkMode, notifications }

users/{userId}/workouts/{date}
  - day: 'monday' | 'tuesday' | ...
  - exercises: [{ id, name, sets: [{ weight, reps, done }] }]
  - startedAt, completedAt

users/{userId}/meals/{date}
  - meals: [{ name, time, items: [{ name, cal, protein, carbs, fat }], confirmed }]

users/{userId}/water/{date}
  - ml: number (target: 2250)
  - log: [{ time, amount }]

users/{userId}/progress/{date}
  - weight, bodyFat, measurements: { waist, chest, arms }
  - photos: { front, side, back }
  - notes
```

## Related Projects
- `enzo-hchb-sync` and `enzo-sync-ingestion` — same React+htm+single-HTML pattern
- Design patterns carried over: Cards, animations, mobile-first approach
