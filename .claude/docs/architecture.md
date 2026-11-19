# Architecture

<!-- Reflects the CURRENT state of the system. Update in place as decisions change. -->

## Overview

Flutter app organized as feature **modules** (`lib/modules/<feature>/`), each split into
three layers: **datalayer** (DTO/model/mapper/repository), **domainlayer** (service —
talks to the REST API or local DB), and **presentationlayer** (ChangeNotifier controller +
pages/widgets). Cross-cutting services live under `lib/services/` and `lib/config/`.
Dependency wiring is centralized in `main.dart` via `provider`'s `MultiProvider`.

## Offline-First Design

The app has two data modes:

### Bundled Offline Content (always available)
- Safety guides (8 categories: floods/landslides, beach safety, health, first aid,
  roads/transport, wildlife, personal security, go-bag/emergency numbers)
- First-aid step-by-step instructions (IFRC-aligned)
- Emergency phone numbers (police, ambulance, tourist police, DMC, Red Cross)
- Safety topic detail content

These are stored as structured data in the app's assets and loaded into SQLite on first
launch. They update when the app updates or via a background content sync when online.

### Online Features (require connectivity)
- Live hazard alerts from national authorities
- Weather data (temperature, conditions)
- Interactive safety map with real-time hazard zones
- Explore directory (live business listings, ratings, distance)
- "I'm Safe" broadcast
- Push notifications for emergency alerts

### Connectivity Handling
- `ConnectivityService` monitors network state
- Online features degrade gracefully: cached data shown with a "last updated" timestamp
- Offline indicator in the header ("✓ Offline ready" or "⚠ Limited — no connection")
- No feature crashes or shows an error screen just because the network is unavailable

## Key Decisions

- **Layered-per-module over layered-per-app**: each feature module owns its full stack
  (data → domain → presentation). Keeps a feature's code together; new features follow
  the same three-folder shape.
- **DTO → Mapper → Model separation** for API-sourced data: `*Dto.fromJson` does raw
  parsing; `*Mapper.toModel()` applies display-formatting; `*Model` is what the UI renders.
- **SQLite models use `fromMap`/`toMap` directly** — no DTO layer for local-only data.
- **Provider for DI, not global state management**: `provider` hands services down to
  controllers; controllers are plain `ChangeNotifier`s created per-page, not globally.
- **No login required for core safety features**: the app works without authentication.
  Profile/settings use local storage. If a backend account is added later, it's opt-in
  for enhanced features only.
- **Context-aware UI adaptation**: when a hazard warning is active, the UI dynamically
  shifts (warning colors, indoor-first explore recommendations, prominent alert banners).

## Modules

| Module | Owns |
|---|---|
| `lib/modules/onboarding/` | Welcome, onboarding carousel, permission setup, setup confirmation |
| `lib/modules/home/` | Home dashboard — greeting, alert banner, quick-access grid, explore preview, safety topics grid |
| `lib/modules/safety_map/` | Interactive hazard/safe-zone map, place detail, map filters |
| `lib/modules/explore/` | Business directory, normal and warning-state variants |
| `lib/modules/safety_topics/` | Safety topic listing, topic detail, offline content |
| `lib/modules/first_aid/` | First-aid index, step-by-step guides |
| `lib/modules/sos/` | SOS emergency screen, one-tap dialer, emergency numbers |
| `lib/modules/trip/` | Trip planning, go-bag checklist |
| `lib/modules/alerts/` | Live alerts feed, alert detail |
| `lib/modules/profile/` | Profile, settings, emergency contacts, language, permissions |
| `lib/services/` | Cross-module: ConnectivityService, LocationService, WeatherService, OfflineContentService, NotificationService |
| `lib/config/` | Environment, API config, database config |
| `lib/theme/` | Design-system tokens: AppColors, AppSpacing, AppRadius, AppTextStyles, AppTheme |
| `lib/utils/` | AppLogger, Formatter, parser utilities |
| `lib/widgets/` | Shared UI components (AppCard, AppButton, etc.) |
| `lib/constants/` | App-wide constants, assets paths, route names |

## Navigation

Bottom navigation bar with 5 items:
1. **Home** — Home dashboard
2. **Map** — Safety map
3. **SOS** — Emergency screen (prominent red center button)
4. **Explore** — Business directory
5. **Alerts** — Live alerts feed

Top-level navigation uses `GoRouter` or manual `Navigator` (TBD). Each bottom-nav
destination has its own navigation stack.
