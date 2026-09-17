# Aura — Kotlin Multiplatform Productivity App

[![Kotlin](https://img.shields.io/badge/Kotlin-multiplatform-7F52FF?logo=kotlin&logoColor=white)](https://kotlinlang.org/)
[![Compose Multiplatform](https://img.shields.io/badge/Compose-Multiplatform-4285F4?logo=jetpackcompose&logoColor=white)](https://www.jetbrains.com/lp/compose-multiplatform/)
[![Android](https://img.shields.io/badge/Android-first-3DDC84?logo=android&logoColor=white)](https://developer.android.com/)
[![Firebase](https://img.shields.io/badge/Firebase-integrations-FFCA28?logo=firebase&logoColor=202124)](https://firebase.google.com/)
[![Gradle](https://img.shields.io/badge/Gradle-build-02303A?logo=gradle&logoColor=white)](https://gradle.org/)

Aura is a Kotlin Multiplatform productivity application built with Compose Multiplatform and an Android-first Firebase architecture. It brings together authentication, onboarding, dashboard workflows, todos, habits, journaling, Pomodoro state, notifications, remote configuration, and experiment-aware product behavior.

> This repository is a portfolio fork of a collaborative project. It is maintained here to document the mobile architecture, project scope, and the areas I personally contributed to.

The project demonstrates breadth beyond web development: shared Kotlin domain and presentation code, Android platform integration, Firebase authentication and data services, local preferences, background work, notification delivery, feature flags, and recovery of an application after reconnecting it to a separate Firebase environment.

## About the Project

Aura follows a Kotlin Multiplatform structure with shared feature code and platform-specific implementations. The current product path is Android-first: Android contains the concrete Firebase, Credential Manager, WorkManager, notification, and Remote Config integrations used by the runnable experience, while iOS targets and source sets are present but not represented as a claim of feature-complete iOS parity.

The repository includes common feature modules for authentication, home/dashboard, todos, habits, journal, Pomodoro, settings, onboarding, notifications, and experiments. Firebase-backed user data is scoped below `users/{userId}` in Firestore, while experiment events use Realtime Database. DataStore holds local preferences and workflow state; background work and notifications are handled through Android platform services.

## Core Features

### Authentication and onboarding

- Firebase Authentication with Google Sign-In.
- Android Credential Manager flow using the generated OAuth web client ID.
- Authorized-account selection with account-picker fallback and explicit error states.
- Session-aware navigation, sign-in/sign-out state handling, and localized onboarding.

### Productivity workflows

- Dashboard/home experience with user context and feature-gated sections.
- Todo creation, editing, completion, due dates, and Firestore-backed reactive updates.
- Habit creation, streak/completion tracking, and Firestore completion records.
- Journal entries stored and observed through the signed-in user's Firestore subtree.
- Pomodoro state and settings persisted through local DataStore-backed preferences.

### Notifications and background work

- Firebase Cloud Messaging integration and Android notification channels.
- Runtime notification permission handling and notification-facing UI states.
- WorkManager jobs for daily summaries, Pomodoro-related work, and experiment heartbeat behavior.
- Local scheduling paths for application reminders and workflow notifications.

### Remote configuration and experiments

- Firebase Remote Config defaults, fetch/activate, and real-time update listening.
- Feature flags with a polling fallback when real-time updates are unavailable.
- Free/Premium gates and experiment variants represented in shared application logic.
- Realtime Database event logging under the authenticated user's experiment path.

### Product foundation

- Compose Multiplatform UI with shared design tokens, typography, components, and screen shells.
- Navigation and dependency injection with Kotlin serialization and Koin.
- Localization assets and translation workflow support.
- Common tests plus Android instrumented and visual-fixture coverage for critical UI behavior.

## Technical Architecture

```text
commonMain
  shared models, repositories, use cases, ViewModels, navigation, UI
       │ expect/actual boundaries
       ▼
androidMain
  Firebase Auth/Firestore/RTDB/Remote Config/FCM
  Credential Manager, WorkManager, notifications, DataStore
       │
       ├── Firestore: users/{uid}/todos, habits, completions, journals
       └── RTDB: users/{uid}/experiments/events

iosMain
  platform target and source-set boundaries; selected integrations remain partial
```

The project separates shared feature logic from platform services through interfaces and Android implementations. Firebase initialization, Google credential exchange, Firestore listeners, Remote Config, FCM, WorkManager, and notification channels are wired in the Android source set. The repository's [KMP architecture notes](./docs/KMP_ARCHITECTURE.md) and [Firebase integration notes](./docs/FIREBASE_IN_KMP.md) document those boundaries.

Room and SQLite dependencies are present in the Gradle configuration, but the current checkout does not contain active Room entities, DAOs, or a Room database implementation for the product flows. The current user-data path is Firestore plus DataStore; this README does not present Room as an implemented persistence layer.

## My Role & Contributions

My contributions cover mobile UI and feature work, authentication and Firebase integration, experiment infrastructure, Android build configuration, and the later reconnection of the application to a separate Firebase environment.

### Direct product and platform contributions

- **Authentication:** implemented and strengthened the authentication state model, platform error handling, Google Sign-In integration, Credential Manager request flow, OAuth client wiring, authorized-account fallback, and sign-in transition behavior. The current `MainActivity` and auth state code preserve that work; see [`238f7dd`](https://github.com/sebas123312231/aura-app/commit/238f7dd) and the authentication work in [PR #12](https://github.com/Alee053/aura-app/pull/12).
- **Remote Config and experiments:** contributed the Remote Config service, feature-flag handling, experiment models/repositories/use cases, A/B variant behavior, Free/Premium gates, RTDB experiment-event logging, and related heartbeat/notification paths. The integration direction is visible in [PR #12](https://github.com/Alee053/aura-app/pull/12) and the subsequent [PR #13](https://github.com/Alee053/aura-app/pull/13).
- **Notifications:** implemented and debugged Android notification channels, permission handling, FCM-facing behavior, and notification-related UI states, including the platform edge cases represented by the current Android source.
- **Application features:** contributed across onboarding/session navigation, dashboard/home, habits, todos, journal, Pomodoro, settings/theme behavior, localized strings, and the shared Compose design system. The current fork history contains the corresponding feature, UI, and interaction commits rather than only isolated configuration changes.
- **Testing and build readiness:** added or extended common domain/ViewModel/mapper/manager tests, Android instrumented UI fixtures, operation-oriented UI checks, and the Gradle/build configuration needed to run Android debug and test tasks.

The project was built collaboratively. The product feature list describes the application as a whole; the bullets above identify areas supported by my commits and the collaborative PR history, without claiming sole authorship of every screen or service.

## Environment Reconstruction & Runtime Recovery

The later infrastructure work was more involved than changing a Firebase URL. I decoupled the portfolio/demo checkout from its inherited Firebase attachment and reconstructed the configuration required for the application to operate against a separate project.

That recovery work included:

- Creating the repository-side Firebase project binding and aligning the Android application with the new `google-services.json` configuration and project metadata.
- Restoring the checked-in Firestore rules and composite indexes for user-scoped todos, habits, completions, and journals.
- Reconnecting Firebase Authentication and Google OAuth wiring, including the generated web client ID consumed by Credential Manager and the Android-side error/fallback paths.
- Re-establishing the Remote Config, feature-flag, Firestore, Realtime Database, FCM, and notification integration points after the infrastructure change.
- Updating and validating Gradle/build configuration so the Android target could be assembled and its test tasks could run against the reconstructed project setup.
- Debugging broken or incomplete Credential Manager/Google authentication paths and restoring the session-dependent modules that rely on a valid signed-in user.

The repository intentionally does not expose SHA-1/SHA-256 fingerprints, OAuth secrets, Firebase credentials, or hosted-service claims. SHA registration, authorized OAuth clients, Firebase Console state, and real-device verification remain external setup requirements. The reconstruction demonstrates infrastructure recovery and integration debugging; it is not a claim of production deployment.

## Technical Highlights

- **Shared/mobile boundary:** common Kotlin code owns feature behavior while Android actuals handle Firebase, credentials, background work, notifications, and platform permissions.
- **User-scoped cloud data:** Firestore rules and repository paths keep application data below the authenticated user's document tree.
- **Resilient sign-in flow:** Credential Manager requests, authorized-account selection, account-picker fallback, and explicit state/error handling are coordinated instead of relying on a single happy path.
- **Runtime configuration:** Remote Config and feature-flag managers support defaults, fetch/activate, real-time updates, and a fallback polling path.
- **Experiment instrumentation:** A/B decisions and experiment events have explicit models and RTDB persistence paths.
- **Android operational behavior:** WorkManager, FCM, notification channels, permissions, and local state persistence are treated as product behavior rather than afterthoughts.

## Tech Stack

| Area | Technologies |
| --- | --- |
| Language and UI | Kotlin, Kotlin Multiplatform, Compose Multiplatform |
| Android | Android SDK, Credential Manager, Google Identity Services, WorkManager, notification APIs |
| Cloud services | Firebase Authentication, Firestore, Realtime Database, Remote Config, Cloud Messaging |
| Local state | DataStore, in-memory/session state, platform scheduling |
| Architecture | Shared feature layers, repository/use-case boundaries, ViewModels, Koin, Kotlin serialization |
| Quality and tooling | Gradle, Kotlin tests, coroutines test utilities, Android instrumented UI tests, visual fixtures |

## Local Setup

The application requires Android tooling and an authorized Firebase project for real authentication and cloud-backed behavior. The checked-in Firebase configuration is project-specific; replace it with an authorized configuration when working in another environment and never commit secrets.

### Requirements

- JDK 17 or newer supported by the current Android Gradle Plugin line; this checkout was inspected with JDK 21.
- Android SDK with API 36 installed. The app declares min SDK 24 and target/compile SDK 36.
- Android Studio or an equivalent Android SDK/emulator setup.
- The Gradle wrapper included in the repository (`gradle-9.4.1`).
- A Firebase project with Google Authentication, Firestore, Remote Config, Realtime Database, and FCM enabled as needed by the flow being tested.

### Build and test

```bash
./gradlew :composeApp:assembleDebug
./gradlew :composeApp:testDebugUnitTest
./gradlew :composeApp:connectedAndroidTest
```

On Windows, use `gradlew.bat` instead of `./gradlew`. The Android tests and visual fixtures can exercise local UI behavior without representing a live OAuth or production Firebase validation. Real sign-in, Firestore, Remote Config, RTDB, FCM, and physical-device notification checks require the external Firebase project and its console-side configuration.

Translation synchronization is optional and requires the repository's `LOCO_API_KEY`; it is not required for the standard Android debug build. The optional `functions` directory also requires its own Node/Firebase CLI setup and authorized project configuration.

## Current Status

Aura is an Android-first Kotlin Multiplatform portfolio project with an active shared codebase and substantial Firebase integration. The Android path is the primary runnable surface; the iOS target and selected platform integrations remain partial. This repository makes no claim of production deployment, production user volume, or complete cross-platform parity.

## Credits & Collaboration

- Original collaborative repository: [`Alee053/aura-app`](https://github.com/Alee053/aura-app)
- Portfolio fork: [`sebas123312231/aura-app`](https://github.com/sebas123312231/aura-app)
- Selected evidence: [Firebase project reconnection](https://github.com/sebas123312231/aura-app/commit/6f1496c), [Firestore rules and indexes](https://github.com/sebas123312231/aura-app/commit/b9441e2), [Credential Manager/auth hardening](https://github.com/sebas123312231/aura-app/commit/238f7dd), [integration PR #12](https://github.com/Alee053/aura-app/pull/12), and [merged integration PR #13](https://github.com/Alee053/aura-app/pull/13).

