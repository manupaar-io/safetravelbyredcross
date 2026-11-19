# PROGRESS & PROJECT ROADMAP: Safe Travel Sri Lanka

<!--
This is the single source of truth for project status, active roadmap, pending work, and decision logs.
Read this at the start of every session to immediately know:
1. What has been completed
2. What is actively in progress
3. What is coming next (the roadmap)
4. Open architectural/technical decisions
DO NOT modify the root TODO file (reserved for manual user tracking).
-->

**Last Updated:** 2026-11-19 (Post Architecture & System Design Phase)  
**Current Phase:** Phase 1 — Foundation & Navigation Shell  
**Active Working Branch:** `main` (feature branches per milestone)  

---

## 🚦 Quick Executive Dashboard

```
Overall Progress: [████░░░░░░░░░░░░░░░░] 20% (Architecture & Design 100% | Code Implementation 0%)

[COMPLETED]       Session 0: Project Setup & Tooling Configuration
[COMPLETED]       Session 0.5: Security Hardening & Performance Budgets
[COMPLETED]       Session 1.0: Full System Design, ERD, Schema DDL & Feature Breakdown
[READY TO START]  Phase 1: Foundation, Design Tokens, Database Baseline & 5-Tab Shell
[UPCOMING]        Phase 2: Offline Core (First Aid, Safety Topics, Emergency Numbers, SOS, Onboarding)
[UPCOMING]        Phase 3: Online & SWR Features (Weather, Alerts, Explore, Safety Map)
[UPCOMING]        Phase 4: User Features (Trip Planner, Go-Bag Checklist, Profile & ICE Contacts)
[UPCOMING]        Phase 5: Enhancements (FCM Siren Alerts, Delta Sync, Cloud Backup)
[UPCOMING]        Phase 6: Polish, Tri-Lingual Audio/Text Verification, Hardening & Store Release
```

---

## 📋 Comprehensive Phase Roadmap & Feature Status

### ✅ COMPLETED PHASES

#### Phase 0: Project Initialization & Configuration
- [x] Initialized Flutter 3.x project (`safe_travel_sri_lanka`).
- [x] Established project rules: `CLAUDE.md`, `AGENTS.md`.
- [x] Documented architectural guidelines in `.claude/docs/` (`architecture.md`, `project-brief.md`, `project-structure.md`, `tech-stack.md`, `coding-patterns.md`).
- [x] Configured Git and baseline directory structure.

#### Phase 0.5: Security & Performance Baseline
- [x] Authored project-wide `SECURITY.md` (Threat Model, TLS 1.3, Cert Pinning, Anti-Tampering, Zero cleartext HTTP).
- [x] Authored `.claude/docs/security-agent-guidelines.md` (Pre-commit quality gates, PII logging ban, parameterized SQL).
- [x] Formulated Life-Critical Performance Budgets (<1.2s cold start, 0ms SOS RAM cache, SWR caching, Isolate.run for >50KB).
- [x] Locked Anti-Overengineering Principle (KISS/YAGNI, 60-second file readability).

#### Phase 1.0: System Design, ERD & Technical Specifications
- [x] Authored **System Design Document (SDD)** ([`system-design-document.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/system-design-document.md)) with full Mermaid architecture & sequence diagrams.
- [x] Authored complete **Entity-Relationship Diagram (ERD)** modeling 13 SQLite tables + SecureStorage key-value pairs.
- [x] Authored **Database Schema & DDL** ([`database-schema.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/database-schema.md)) with indexes, migration runner, and seed data loader.
- [x] Authored **Feature Analysis & Module Breakdown** ([`feature-analysis.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/feature-analysis.md)) categorizing P0/P1/P2 priorities across all 10 modules.
- [x] Authored **UI/UX Design Specification & Component Inventory** ([`ui-ux-design-specification.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/ui-ux-design-specification.md)) analyzing 33 prototype screenshots for exact color hex codes, M3 type scales, 8pt spacing grid, zero-spinner skeletons, and 14 screen-by-screen widget blueprints.
- [x] Authored **Open Questions Register** ([`open-questions.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/open-questions.md)) tracking 26 external API and backend dependencies with safe defaults.
- [x] Authored **Phase 1 Feature Plan** ([`01-foundation-and-shell.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/features/01-foundation-and-shell.md)).

---

### ⏳ IN PROGRESS / NEXT UP: Phase 1 — Foundation & Navigation Shell

**Detailed Plan:** [`01-foundation-and-shell.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/features/01-foundation-and-shell.md)  
**Primary Goal:** Transform the blank Flutter counter app into the fully wired, themed 5-tab application shell.

- [ ] **Milestone 1.1: Dependencies & Asset Layout**
  - [ ] Add production dependencies to `pubspec.yaml` (`provider`, `sqflite`, `flutter_secure_storage`, `shared_preferences`, `go_router`, `connectivity_plus`, `url_launcher`, `geolocator`, `intl`, `uuid`, `http`).
  - [ ] Create `assets/data/`, `assets/images/`, `assets/fonts/` folders and configure in `pubspec.yaml`.
  - [ ] Configure `analysis_options.yaml` with strict linter rules.
- [ ] **Milestone 1.2: Design System & Theme Tokens (`lib/theme/`)** *(Follow [`ui-ux-design-specification.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/ui-ux-design-specification.md))*
  - [ ] `app_colors.dart`: Red Cross brand red (`#ED1C24`), warm off-white canvas (`#F8F6F2`), OLED dark canvas (`#121212`), alert levels (`#DC2626`, `#C2410C`, `#D97706`, `#16A34A`, `#0284C7`), and floating dock (`#1A1A1A`).
  - [ ] `app_spacing.dart`: 8pt grid (`xs: 4`, `sm: 8`, `md: 16`, `lg: 24`, `xl: 32`, `xxl: 48`) and border radii (`card: 16`, `modal: 24`, `navBar: 32`, `pill: 999`).
  - [ ] `app_fonts.dart`: Material 3 type scale (displayLarge 32pt to labelSmall 11pt tracking +0.8).
  - [ ] `app_theme.dart`: Material 3 `AppTheme.light` and `AppTheme.dark`.
  - [ ] `theme_controller.dart`: `ChangeNotifier` with `shared_preferences` persistence.
- [ ] **Milestone 1.3: Core Utilities (`lib/utils/`, `lib/constants/`)**
  - [ ] `app_logger.dart`: PII-stripped, release-gated logging.
  - [ ] `phone_sanitizer.dart`: Emergency phone regex validator (`^\+?[0-9]{3,12}$`).
  - [ ] `response_object.dart`: Universal service response wrapper.
  - [ ] `formatter.dart` & `parser_utils.dart`.
  - [ ] `app_constants.dart`, `assets.dart`, `route_names.dart`.
- [ ] **Milestone 1.4: Infrastructure & Services (`lib/services/`, `lib/config/`)**
  - [ ] `connectivity_service.dart`: Network status stream.
  - [ ] `secure_storage_service.dart`: KeyStore (AES-256-GCM) / Keychain wrapper.
  - [ ] `environment.dart`: `--dart-define` config reader.
  - [ ] `api_config.dart`: REST endpoints & timeout configurations.
  - [ ] `database/db_config.dart`: SQLite initialization & schema v1 creation.
- [ ] **Milestone 1.5: 5-Tab Navigation Shell (`lib/modules/shell/`)**
  - [ ] `app_router.dart`: `go_router` with `StatefulShellRoute.indexedStack`.
  - [ ] `app_shell.dart`: Persistent bottom nav bar with prominent elevated red center SOS button.
  - [ ] Tab state preservation verification across all 5 branches.
- [ ] **Milestone 1.6: Core Shared Widgets (`lib/widgets/`)**
  - [ ] `AppButton`, `AppCard`, `AppEmptyState`, `StatusBadge`, `ConnectivityBanner`.
- [ ] **Milestone 1.7: Tri-Lingual Localization Baseline**
  - [ ] Set up `l10n.yaml`, `app_en.arb`, `app_si.arb`, `app_ta.arb`.
- [ ] **Milestone 1.8: `main.dart` Composition**
  - [ ] Wire `MultiProvider`, `MaterialApp.router`, `ThemeController`, and error safety net.

---

### 🔮 UPCOMING PHASES ROADMAP

#### Phase 2: Offline Safety Core (Life-Critical Features)
- [ ] **Bundled Content JSON Seed Files:** Build offline asset JSONs for safety topics, first aid, emergency hotlines, and go-bag items.
- [ ] **Emergency Hotlines & Pre-Warming (`lib/modules/sos/`):** 0ms RAM cache initialization at boot, one-tap dialer confirmation sheet.
- [ ] **First Aid Manuals (`lib/modules/first_aid/`):** Step-by-step swipeable instructions, severity triage, offline search.
- [ ] **Safety Topics (`lib/modules/safety_topics/`):** 8 categories (Floods, Landslides, Coastal, Wildlife, etc.) with detail views.
- [ ] **Onboarding Flow (`lib/modules/onboarding/`):** Welcome, feature carousel, location/notification permission requests, all-set confirmation.

#### Phase 3: Online & SWR Caching Features
- [ ] **Home Dashboard (`lib/modules/home/`):** Greeting header, live weather card (SWR), active hazard alert banner, 2x2 quick-access grid, explore preview.
- [ ] **Context-Aware Dynamic UI:** Weather-warning adaptation (hazard tone shifts, indoor/safe shelter prioritization).
- [ ] **Real-Time Alerts Feed (`lib/modules/alerts/`):** Chronological feed, severity filtering, SWR background sync.
- [ ] **Traveler Explore Directory (`lib/modules/explore/`):** Categories (Hotels, Clinics, Tourist Police), distance sorting, weather-adaptive indoor mode.
- [ ] **Interactive Safety Map (`lib/modules/safety_map/`):** Google Maps integration, hazard overlay polygons/circles, Red Cross points, static offline vector fallback map.

#### Phase 4: User Personal Features
- [ ] **Emergency Contacts & Profile (`lib/modules/profile/`):** ICE contacts, medical info (blood group, allergies) stored strictly in `flutter_secure_storage`.
- [ ] **Trip Planner (`lib/modules/trip/`):** Local itinerary CRUD, trip dates, notes.
- [ ] **Go-Bag Survival Checklist (`lib/modules/trip/`):** Interactive checklist with persistent check states per trip.
- [ ] **Language Switcher:** In-app toggle between English, Sinhala, and Tamil.

#### Phase 5: Enhancements & Integrations
- [ ] **"I'm Safe" Broadcast:** GSM SMS generation to ICE contacts + async REST dispatch.
- [ ] **Push Notifications:** Firebase Cloud Messaging (FCM) high-priority disaster warnings.
- [ ] **Incremental Content Delta Sync:** `GET /api/v1/content/sync?since=timestamp`.
- [ ] **Backend Auth & Cloud Sync (Optional):** Wire JWT authentication when .NET API docs arrive.

#### Phase 6: Hardening, Polish & Release
- [ ] Accessibility review (48dp touch targets, high-contrast text, screen reader labels).
- [ ] Battery & GPS optimization tests.
- [ ] Obfuscated release builds (`--obfuscate --split-debug-info`).
- [ ] Android APK & iOS IPA store packaging.

---

## 🔒 Confirmed Architectural Decisions (D0 — D11)

| ID | Decision | Confirmed Outcome |
|:---:|---|---|
| **D0** | Core Stack | Flutter 3.x / Dart 3, `provider` for DI, `sqflite` for DB, hand-rolled `http` client. |
| **D1** | Offline-First Core | Bundled JSON assets seeded into SQLite; critical safety works 100% without internet. |
| **D2** | Modular Layering | Feature modules under `lib/modules/<feature>/` split into `datalayer/`, `domainlayer/`, `presentationlayer/`. |
| **D3** | Authentication Scope | No login required for core safety features; app is immediately accessible to anyone. |
| **D4** | Platform Priority | Android-first, iOS second. |
| **D5** | Primary Navigation | 5-tab persistent bottom navigation (`Home`, `Map`, `SOS` [Red center], `Explore`, `Alerts`). |
| **D6** | 3-Tier Storage Matrix | 1. `flutter_secure_storage` (PII/tokens/keys)<br>2. `sqflite` + SQLCipher (offline content/user trips)<br>3. `shared_preferences` (UI flags only). |
| **D7** | Cleartext Traffic Ban | Strict HTTPS only (`cleartextTrafficPermitted="false"`, `NSAllowsArbitraryLoads=false`). |
| **D8** | Performance Budgets | Cold start <1.2s, SOS dialer 0ms RAM lookup, SWR zero-spinner caching, `Isolate.run` for >50KB JSON. |
| **D9** | Anti-Overengineering | Pragmatic simplicity (KISS/YAGNI). Zero speculative abstractions. 60-second file readability. |
| **D10**| Tri-Lingual Support | Native English (`en`), Sinhala (`si`), and Tamil (`ta`) supported from the start. |
| **D11**| Routing Engine | `go_router` using `StatefulShellRoute.indexedStack` for nested tab navigation. |

---

## ❓ Open Decisions & External Dependencies (Summary)

*Full 26-item registry in [`open-questions.md`](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/open-questions.md).*

| Question ID | Topic | Pending External Input | Provisional Client Default (Unblocked) |
|---|---|---|---|
| **OQ-01** | Backend Auth Scheme | .NET Core JWT / OAuth2 contract | Stateless Bearer token header in `SecureHttpClient`. |
| **OQ-06** | API Base URL & Version | Production & Staging hostnames | `https://api.safetravel.redcross.lk/api/v1/`. |
| **OQ-11** | Weather Data Source | Backend aggregator vs Direct 3rd Party | .NET backend aggregates DMC data; client calls backend. |
| **OQ-12** | Hazard Alert Ingestion | Push notification payload contract | FCM high-priority topic subscription + REST polling fallback. |
| **OQ-16** | "I'm Safe" Broadcast | GSM SMS vs Server endpoint | Dual mode: instant SMS to ICE contacts + async REST call. |
| **OQ-21/22** | Cloud Keys & FCM | Firebase config & Google Maps API key | Compile-time injection via `--dart-define`. |
