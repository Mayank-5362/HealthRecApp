# HealthRecApp

HealthRecApp is a friendly personal health tracker built with React Native. It is designed to make daily logging feel quick and calm, not like homework. You can track your sleep, water, mood, diet, symptoms, and activity in one place, even when you are offline. The app stores your data locally first and syncs it to Supabase when you are online, so your entries are not lost if the network is slow or unavailable. Android is the current focus, and the app includes a native step counter that keeps updating your daily steps without keeping the app open.

This repository contains everything needed to run the app: UI screens, reusable components, data services, and the native Android and iOS projects. Navigation is tab based, so each main feature has its own space. State is shared through context providers, which makes it easy for screens to read and update data without passing long chains of props. Services keep the core logic in one place, so the UI stays clean and readable. Overall, the project is compact, modular, and easy to extend.

## What you can do with the app

- Log sleep, hydration, mood, diet, and symptoms in under a minute.
- Track steps on Android using the device step counter sensor.
- Record timed activities like running, cycling, or swimming.
- Estimate calories from both steps and timed workouts.
- Scan food images for quick nutrition hints (helper feature).
- Get lightweight insights about trends, averages, and streaks.
- Keep logging even when offline, then sync later.

## Tech stack (simple and practical)

HealthRecApp uses a familiar React Native stack with a few focused extras:

- **React Native + TypeScript** for a fast mobile UI with safer data models.
- **React Navigation** for bottom tabs and a root stack that gates auth.
- **SQLite (react-native-sqlite-storage)** for offline-first local storage.
- **Supabase** for authentication and cloud sync.
- **Android native Kotlin service** for reliable step tracking.
- **Jest** for tests, plus ESLint and Prettier for clean code.

Key configuration lives in [package.json](package.json), [babel.config.js](babel.config.js), [metro.config.js](metro.config.js), and [tsconfig.json](tsconfig.json). The Supabase schema is defined in [supabase/schema.sql](supabase/schema.sql), and secrets are loaded from a local file based on [services/appSecrets.example.ts](services/appSecrets.example.ts).

## Main modules (explained in plain language)

### Authentication and session

This module handles sign-in, sign-up, and session persistence. The UI is in the login screen, while the logic is in [context/AuthContext.tsx](context/AuthContext.tsx). The context exposes simple actions like log in and sign up, and it decides when the app should show the login flow versus the main tabs. It also handles the common case where a user needs to confirm their email before a full session is available. By keeping auth state in one place, the rest of the app can stay focused on features.

### Daily health logging

Daily logging is the heart of the app. Users can quickly record sleep, hydration, mood, diet, and symptoms in the logging screen. Input controls are built for speed: selectors, cards, and minimal typing. All logging actions flow through [context/HealthDataContext.tsx](context/HealthDataContext.tsx), which acts as the single source of truth for the UI. Each entry is stored in SQLite with timestamps and sync status so it can be pushed to Supabase when the user is online.

### Activity and calorie tracking

Timed activities like running or cycling are recorded as sessions with a duration. The app calculates an estimated calorie burn based on duration and body weight, using logic in [services/calorieEstimate.ts](services/calorieEstimate.ts) and [services/activityCalories.ts](services/activityCalories.ts). The daily total is a combination of timed activity calories and step-based calories, which helps avoid double counting. The UI stays simple, while the calculation logic is centralized and easy to update.

### Step tracking (Android)

Android step tracking is built with a native foreground service that listens to the step counter sensor. It stores step counts in shared preferences, and a boot receiver ensures tracking continues after a restart. The React Native side reads this data through [services/androidStepCounter.ts](services/androidStepCounter.ts), and the hook in [hooks/useStepCounter.ts](hooks/useStepCounter.ts) gives screens a clean API. This feature is Android only, and emulator support can be limited, so real device testing is recommended.

### Nutrition scan

The nutrition feature lets users take a food image and receive quick nutrition hints. Network calls are encapsulated in [services/fatSecret.ts](services/fatSecret.ts) and [services/geminiNutri.ts](services/geminiNutri.ts), which keeps the UI clean and allows the API layer to change without rewriting screens. This feature is meant to be a helper rather than a strict tracker, so it is best used as guidance rather than exact measurement.

### Symptom checker

The symptom checker provides a calm, lightweight interface for users to enter symptoms and get a response from a hosted model service. The focus is on clarity and reassurance rather than diagnosis. The module keeps the interaction simple and avoids medical claims. It is designed as a tool for awareness and tracking, not a replacement for professional care.

### Insights and trends

Insights are generated locally from synced data and stored as snapshots. The logic in [services/insights.ts](services/insights.ts) creates friendly summaries like streaks, averages, and changes over time. Because insights are calculated on device, the app can show them even when offline. Snapshots use user-scoped IDs to avoid cross-user collisions in Supabase. The Insights screen displays these summaries as cards you can scan quickly.

### Sync and data layer

The app uses a queue-based sync model. Changes are saved locally first, then uploaded when the network is available. The sync engine lives in [services/sync.ts](services/sync.ts), while the data mapping and persistence are handled in [repositories/healthRepository.ts](repositories/healthRepository.ts). App data is camelCase in TypeScript and converted to snake_case when stored in SQLite or Supabase. This keeps the UI readable and the storage layer consistent with SQL conventions.

## Project structure (simple view)

Here is the project layout in plain terms:

- [App.tsx](App.tsx) and [index.js](index.js) start the app.
- [navigation/](navigation/) defines the root stack and bottom tabs.
- [screens/](screens/) contains each feature screen (dashboard, log, activity, etc.).
- [components/](components/) holds shared UI elements like cards and charts.
- [context/](context/) stores global state for auth and health data.
- [repositories/](repositories/) handles SQLite persistence and mapping.
- [services/](services/) contains domain logic, APIs, and utilities.
- [types/](types/) defines shared TypeScript models.
- [theme/](theme/) contains color tokens and theme helpers.
- [android/](android/) and [ios/](ios/) are the native projects.
- [supabase/](supabase/) stores the SQL schema and policies.

## How data moves through the app

The data flow is designed to be predictable and offline friendly:

1. A screen collects input from the user.
2. A context provider updates local state and calls a repository method.
3. The repository writes to SQLite and marks the record as pending sync.
4. The sync service uploads pending changes to Supabase when online.

This pattern keeps the app fast on-device while still keeping a cloud backup for the user. It also keeps UI code clean, because screens focus on layout and user input while the data layer handles persistence and sync.

## Getting started

Install dependencies, start Metro, and run the Android app:

```
npm install
npx react-native start
npm run android
```

If you want to use Supabase, apply the schema in [supabase/schema.sql](supabase/schema.sql) using the Supabase SQL editor. Then create a local secrets file based on [services/appSecrets.example.ts](services/appSecrets.example.ts). This secrets file is intentionally not committed. For step tracking, test on a real Android device if possible, because emulators often do not have a step counter sensor.

## Notes and limitations

HealthRecApp is a prototype and learning project, not a medical device. Some features rely on external APIs and require local secrets. Android is the primary focus, and iOS support may need extra verification. The symptom checker and nutrition scan are meant to guide awareness, not provide medical advice. Even with those limitations, the project offers a clean base for building a full-featured health tracking app.
