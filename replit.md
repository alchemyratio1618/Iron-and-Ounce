# Iron & Ounce

## Overview

Iron & Ounce is a nutrition-first fitness tracking mobile app built with Expo (React Native). It follows a "Burn-to-Earn" calorie model where exercise earns additional calorie budget. The app tracks food intake, workouts, hydration, weight progress, and streaks. It features an onboarding flow, a dashboard with a calorie ring visualization, macro tracking with a "Gold" protein status, and a gamified badge/milestone system.

The app uses a hybrid architecture: an Expo/React Native frontend for cross-platform mobile and web support, paired with an Express.js backend server. Data is currently stored client-side using AsyncStorage, with a PostgreSQL database schema defined via Drizzle ORM (partially integrated — the server has a schema and in-memory storage stub but the app primarily uses local storage).

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend (Expo / React Native)
- **Framework**: Expo SDK 54 with expo-router for file-based routing
- **Routing structure**: Tab-based navigation with 4 tabs: Dashboard (`index`), Food, Workout, Progress
- **State management**: React Context (`DataContext`) provides all app state — profile, daily data, weight log, streaks, computed macros/calories
- **Local storage**: AsyncStorage handles all persistent data client-side (profile, daily logs, weight history, streaks, onboarding state)
- **Styling**: Dark theme throughout, defined in `constants/colors.ts`. Uses the Rubik font family loaded via `@expo-google-fonts/rubik`
- **Key UI components**: `CalorieRing` (SVG ring chart), `MacroBar` (progress bars), `HydrationCard` (water tracking), `StreakBadge`
- **Onboarding**: Multi-step flow in `app/onboarding.tsx` collecting body stats, calorie budget, and macro targets before entering the main app
- **Platform support**: iOS, Android, and Web (with platform-specific adjustments for haptics, keyboard handling, and safe areas)

### Backend (Express.js)
- **Server**: Express 5 in `server/index.ts` with CORS configured for Replit domains and localhost
- **Routes**: Defined in `server/routes.ts` — currently a stub with no API endpoints implemented. All routes should be prefixed with `/api`
- **Storage layer**: `server/storage.ts` defines an `IStorage` interface with a `MemStorage` implementation (in-memory Map). This is a placeholder — the storage interface pattern allows swapping to database-backed storage
- **Static serving**: In production, serves a landing page from `server/templates/landing-page.html`

### Database (PostgreSQL via Drizzle ORM)
- **Schema**: Defined in `shared/schema.ts` — currently only has a `users` table with `id`, `username`, `password`
- **ORM**: Drizzle ORM with `drizzle-zod` for schema validation
- **Config**: `drizzle.config.ts` reads `DATABASE_URL` env var, outputs migrations to `./migrations`
- **Push command**: `npm run db:push` to sync schema to database
- **Current state**: The database schema is minimal and the app doesn't actively use server-side storage — all data lives in AsyncStorage on the client. The database infrastructure is in place for future server-side features

### Data Flow
- The app currently operates as a "local-first" app. The `DataContext` loads from and saves to AsyncStorage
- `lib/query-client.ts` has React Query setup with `apiRequest` helper pointing to the Express server, but it's not actively used by the main data flow
- The `shared/` directory is intended for code shared between frontend and backend (currently just the DB schema)

### Onboarding Flow
- **File**: `app/onboarding.tsx` — multi-step goal-setting flow shown on first launch
- **Steps**: 4 steps — Body (current/goal weight), Calorie Budget, Macro Targets (protein/carbs/fats), Hydration (water goal)
- **Storage**: `@iron_ounce_onboarding_complete` flag in AsyncStorage tracks completion
- **Routing**: `app/_layout.tsx` checks `onboardingComplete` from DataContext — shows `OnboardingScreen` if false, `Stack` with tabs if true
- **No hardcoded defaults**: `DEFAULT_PROFILE` has all zeros. All values must be set by the user during onboarding. This prevents accidental badge/milestone awards
- **Goal editing**: Users can change goals later via the Settings panel in the Progress tab

### Key Design Decisions
1. **Local-first storage**: All user data (meals, workouts, weight, streaks) stored via AsyncStorage for offline-first experience. Server/DB integration is scaffolded but not wired up yet
2. **Context-based state**: Single `DataContext` provides all data and mutations to the component tree, keeping data logic centralized
3. **Common foods/exercises catalog**: Predefined lists in `lib/types.ts` (`COMMON_FOODS`, `COMMON_EXERCISES`) for quick-add functionality
4. **Dark-only theme**: No light mode — the app uses a consistent dark color palette with gold accents
5. **Gamification**: Streak tracking (logging + hydration), badge system, weight milestones, and protein "Gold" status
6. **User-set goals only**: No hardcoded health goals — all values set by user during onboarding to avoid false badge awards

### Build & Development
- **Dev mode**: Run `npm run expo:dev` (Expo) and `npm run server:dev` (Express) concurrently
- **Production build**: `npm run expo:static:build` for static web export, `npm run server:build` + `npm run server:prod` for server
- **Proxy setup**: In development, the Express server proxies Expo's Metro bundler requests

## External Dependencies

### Core Framework
- **Expo SDK 54** with React Native 0.81.5 and React 19.1
- **expo-router** for file-based navigation with typed routes
- **React Native Gesture Handler**, **Reanimated**, **Screens**, **Safe Area Context** for native navigation primitives

### Data & Storage
- **AsyncStorage** (`@react-native-async-storage/async-storage`) — primary client-side persistence
- **Drizzle ORM** with PostgreSQL dialect — server-side database (requires `DATABASE_URL` env var)
- **React Query** (`@tanstack/react-query`) — API data fetching (scaffolded, not yet primary)

### Server
- **Express 5** — HTTP server
- **pg** — PostgreSQL client driver
- **http-proxy-middleware** — dev proxy for Metro bundler

### UI & UX
- **Rubik font** via `@expo-google-fonts/rubik` (Regular, Medium, SemiBold, Bold)
- **react-native-svg** — SVG charts and visualizations
- **expo-haptics** — tactile feedback on interactions
- **expo-blur** / **expo-glass-effect** — tab bar visual effects
- **expo-linear-gradient** — gradient backgrounds
- **@expo/vector-icons** (Ionicons, Feather, MaterialCommunityIcons) — icon sets

### Build Tools
- **esbuild** — server bundling
- **tsx** — TypeScript execution for dev server
- **drizzle-kit** — database migration tooling
- **patch-package** — post-install patches