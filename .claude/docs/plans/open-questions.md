# Open Questions Register — Safe Travel Sri Lanka

**Status:** Living Document  
**Last Updated:** 2026-11-19  
**Reference Decisions:** D0–D11 in [.claude/docs/PROGRESS.md](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/PROGRESS.md)  
**Primary Blocker:** External .NET Core REST API documentation pending.

---

## 1. Summary Dashboard

| Total Items | Open / Pending API | Client-Provisional (Unblocked) | Resolved |
|:---:|:---:|:---:|:---:|
| **26** | **20** | **4** | **2** (D10, D11) |

---

## 2. Authentication & Authorization

### OQ-01: Authentication Scheme & Token Flow
* **Category:** Authentication
* **Question:** Does the .NET Core backend use standard OAuth2 / OpenID Connect Bearer JWT tokens, session cookies, or custom API tokens?
* **Context:** Dictates client-side interceptor architecture, `flutter_secure_storage` key mapping, and 401 challenge mutex handling.
* **Recommended Default:** Stateless JWT Bearer tokens (`access_token`, `refresh_token`) stored in `flutter_secure_storage`. Client passes `Authorization: Bearer <token>` on authenticated endpoints.
* **Impact if Wrong:** Low; client HTTP abstraction (`SecureHttpClient`) can easily switch auth headers or token parsing.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** User profile cloud sync, backend "I'm Safe" broadcasting. (Does **not** block Phase 1 or offline core).

### OQ-02: User Registration & Onboarding Identity
* **Category:** Authentication
* **Question:** Is there a self-service registration flow via the app, or are credentials managed externally / pre-provisioned?
* **Context:** Determines if registration screens, password strength validators, and OTP/email verification pages are required.
* **Recommended Default:** Optional guest mode by default (no login required for core safety). Optional registration/login via email + password or phone OTP for personal trip sync.
* **Impact if Wrong:** Medium UI addition in `lib/modules/profile/`.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Cloud account registration flow.

### OQ-03: Feature Authorization Matrix
* **Category:** Authentication
* **Question:** Which exact endpoints require authentication vs. anonymous public access?
* **Context:** Ensures public safety content (weather, hazard alerts, emergency numbers, first-aid) is never blocked behind a login screen.
* **Recommended Default:** 
  * **Public:** Alerts feed, weather, public map points, explore directory, content sync.
  * **Authenticated:** Custom trip sync, user profile backup, remote "I'm Safe" broadcast log.
* **Impact if Wrong:** Minor routing/guard adjustment in `app_router.dart`.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Route guards.

### OQ-04: Token Refresh & Expiry Handshake
* **Category:** Authentication
* **Question:** What is the refresh endpoint contract (`POST /api/auth/refresh`), token TTL, and sliding expiration policy?
* **Context:** Mobile client must manage background silent token refreshes with a thread-safe mutex to avoid duplicate refresh requests.
* **Recommended Default:** Short-lived access token (15–60 min), long-lived refresh token (30 days). Standard `{ "refreshToken": "..." }` payload returning new token pair.
* **Impact if Wrong:** Internal service logic tweak in `AuthService`.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Auth token auto-refresh.

### OQ-05: Third-Party Social Identity Providers
* **Category:** Authentication
* **Question:** Are Google Sign-In or Apple Sign-In supported/mandated by the backend?
* **Context:** Requires platform native SDKs (`google_sign_in`, `sign_in_with_apple`) and developer account credentials.
* **Recommended Default:** Deferred to Phase 5. Simple email/phone auth or anonymous guest state for initial releases.
* **Impact if Wrong:** Adding third-party Flutter plugins in later phase.
* **Status:** DEFERRED [PENDING: API DOCS]
* **Blocking:** Social login UI.

---

## 3. API Contracts & Network Transport

### OQ-06: Base URL & Versioning Structure
* **Category:** API Contracts
* **Question:** What is the URL schema and API versioning convention (URI path `/api/v1/`, header-based, or query parameter)?
* **Context:** Configuration inside `lib/config/api_config.dart`.
* **Recommended Default:** Path-based versioning: `https://api.safetravel.redcross.lk/api/v1/`.
* **Impact if Wrong:** One-line configuration string change.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Live network calls.

### OQ-07: Pagination Standard
* **Category:** API Contracts
* **Question:** What pagination schema does the .NET backend implement (`pageNumber`/`pageSize`, `offset`/`limit`, or cursor/keyset)?
* **Context:** Infinite scrolling in Explore directory and Alerts archive.
* **Recommended Default:** Standard page-based: `{ "page": 1, "pageSize": 20, "totalPages": 5, "totalRecords": 95, "items": [...] }`.
* **Impact if Wrong:** Adjusting generic `PaginatedResponseDto<T>` structure.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Infinite scroll pagination logic.

### OQ-08: Error Response Specification
* **Category:** API Contracts
* **Question:** Does backend follow RFC 7807 (Problem Details for HTTP APIs) or a custom envelope?
* **Context:** Parsing structured error codes and field validation messages.
* **Recommended Default:** RFC 7807 format (`type`, `title`, `status`, `detail`, `instance`, `errors: { field: [msg] }`).
* **Impact if Wrong:** DTO parsing in `ResponseObject.fromHttpResponse()`.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Standardized server error toasts.

### OQ-09: Timestamp & Timezone Normalization
* **Category:** API Contracts
* **Question:** Are all timestamps emitted in ISO 8601 UTC with explicit `Z` offset?
* **Context:** Accurate chronological alert sorting and local time conversion in Sri Lanka (UTC+05:30).
* **Recommended Default:** Strict ISO 8601 UTC (`YYYY-MM-DDTHH:mm:ss.sssZ`).
* **Impact if Wrong:** Parser utility date-formatter tweak.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Alert relative time formatting.

### OQ-10: Multi-Language Content Delivery Architecture
* **Category:** API Contracts
* **Question:** Does the API return localized payloads based on `Accept-Language` HTTP header, or does each entity carry explicit tri-lingual fields (`title_en`, `title_si`, `title_ta`)?
* **Context:** ERD database schema and DTO serialization design.
* **Recommended Default:** Client database supports tri-lingual fields locally (`_en`, `_si`, `_ta`). API returns the requested language based on `Accept-Language: en|si|ta`, or returns full composite entity. Client repository handles fallback gracefully.
* **Impact if Wrong:** None; tri-lingual database columns support either mode.
* **Status:** PROVISIONALLY ADDRESSED [Client supports both]
* **Blocking:** None.

---

## 4. Data Sources & Real-Time Ingestion

### OQ-11: Weather Ingestion Topology
* **Category:** Data Sources
* **Question:** Does the .NET backend aggregate official Department of Meteorology / DMC data, or should the mobile client directly call a 3rd party weather provider (e.g., OpenWeatherMap, WeatherAPI)?
* **Context:** App architecture, API key placement, and rate-limiting quotas.
* **Recommended Default:** Backend acts as the aggregator (`GET /api/v1/weather?district=galle`), caching official Sri Lankan meteorological radar and forecasts. Client calls our .NET API only.
* **Impact if Wrong:** If client must call 3rd party, requires adding client API key via `--dart-define` and direct weather client.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Weather service integration.

### OQ-12: Hazard Alerts Ingestion & Real-Time Push
* **Category:** Data Sources
* **Question:** What is the notification pipeline for real-time life-critical alerts (NBRO landslides, DMC flood warnings)?
* **Context:** Background waking of app and emergency siren/heads-up banner alerts.
* **Recommended Default:** High-priority Firebase Cloud Messaging (FCM) topic subscription (`/topics/alerts_all` and district-specific topics `/topics/district_colombo`). App also polls `GET /api/v1/alerts/active` on app foregrounding.
* **Impact if Wrong:** Push payload format adaptation in `NotificationService`.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** FCM background handler payload mapping.

### OQ-13: Explore Directory Curation & Offline Seeding
* **Category:** Data Sources
* **Question:** Will explore listings (safe hotels, certified hospitals, tourist police booths) be purely online from the backend, or should a baseline directory be bundled in app assets?
* **Context:** Offline usability during disaster network blackouts.
* **Recommended Default:** Hybrid: A curated seed of critical lifeline facilities (major hospitals, district Red Cross offices, diplomatic missions) is pre-bundled in SQLite assets; commercial listings (cafes, tours) are dynamically fetched and cached.
* **Impact if Wrong:** Size of initial asset bundle.
* **Status:** PROVISIONALLY ADDRESSED [Hybrid planned]
* **Blocking:** None.

### OQ-14: Content Sync & Cache Invalidation Protocol
* **Category:** Data Sources
* **Question:** How does the app check for updated first-aid procedures and safety guide revisions (`ETag`, `If-Modified-Since`, or `GET /api/v1/content/sync?since=timestamp`)?
* **Context:** Minimizing mobile data usage for tourists on roaming.
* **Recommended Default:** Delta sync via `GET /api/v1/content/sync?since=<last_sync_timestamp>`. Table `content_sync_metadata` in SQLite tracks sync state.
* **Impact if Wrong:** Sync service network protocol adaptation.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Content synchronization service.

### OQ-15: Geospatial Hazard Zone Data Format
* **Category:** Data Sources
* **Question:** Does the backend deliver hazard zones as GeoJSON polygons / multi-polygons, or simplified circular coordinates (`latitude`, `longitude`, `radiusKm`)?
* **Context:** Map rendering library performance, polygon rendering overhead on mobile GPU.
* **Recommended Default:** Simplified format for mobile efficiency: point + radius for small warnings, GeoJSON polygon for major flood/landslide risk zones. SQLite `map_points` table stores GeoJSON string in `metadata_json`.
* **Impact if Wrong:** Map rendering layer adapts parser.
* **Status:** OPEN [PENDING: API DOCS]
* **Blocking:** Hazard map polygon rendering.

---

## 5. Functional Features & Emergency Telemetry

### OQ-16: "I'm Safe" Broadcast Dispatch Pipeline
* **Category:** Features
* **Question:** How does broadcasting "I'm Safe" propagate to family / authorities?
* **Context:** Emergency response workflow when cell networks are congested.
* **Recommended Default:** Dual mechanism:
  1. Primary: Direct SMS generation pre-filling ICE emergency numbers with current coordinates and status (works over standard GSM cellular without 4G/data).
  2. Secondary: If internet is active, async `POST /api/v1/safety/broadcast` to log status on the Red Cross response portal.
* **Impact if Wrong:** SMS vs network dispatcher switch.
* **Status:** PROVISIONALLY ADDRESSED [Dual mechanism planned]
* **Blocking:** None.

### OQ-17: Emergency Contacts (ICE) Sync
* **Category:** Features
* **Question:** Are ICE contacts stored strictly client-side for privacy, or backed up to the Red Cross traveler portal?
* **Context:** Privacy regulations (GDPR, Sri Lanka Personal Data Protection Act No. 9 of 2022).
* **Recommended Default:** Stored strictly client-side in hardware-backed `flutter_secure_storage`. Never transmitted to backend unless explicitly included in an active SOS dispatch event.
* **Impact if Wrong:** Regulatory compliance review.
* **Status:** PROVISIONALLY ADDRESSED [Privacy-first local storage]
* **Blocking:** None.

### OQ-18: Trip Planning & Go-Bag Backup
* **Category:** Features
* **Question:** Is the trip itinerary and go-bag checklist purely local or cloud-synchronized?
* **Context:** Offline-first architecture vs cross-device convenience.
* **Recommended Default:** Offline-first local storage in SQLCipher encrypted database. Optional sync endpoint `POST /api/v1/trips/sync` when backend is available.
* **Impact if Wrong:** Low.
* **Status:** PROVISIONALLY ADDRESSED [Local-first, cloud-optional]
* **Blocking:** None.

### OQ-19: Offline Map Tile Strategy
* **Category:** Features
* **Question:** Should the app bundle vector/raster offline map tiles for Sri Lanka, or rely on device OS map caching?
* **Context:** APK/bundle size limit vs disaster-zone usability without cell coverage.
* **Recommended Default:** Rely on standard map caching for dynamic maps, but bundle a static, high-contrast vector offline emergency district map as fallback in `assets/images/maps/`.
* **Impact if Wrong:** App download size (offline tiles can add 50–150MB).
* **Status:** PROVISIONALLY ADDRESSED [Static fallback asset + online cache]
* **Blocking:** None.

### OQ-20: SOS Confirmation Guard Interaction
* **Category:** Features
* **Question:** What is the UX guard on the central SOS button to prevent accidental pocket activation?
* **Context:** False alarm mitigation for emergency dispatchers (119/1990).
* **Recommended Default:** Single tap opens an immediate bottom confirmation sheet with large 1-tap call buttons and a 3-second auto-dial countdown with instant cancel.
* **Impact if Wrong:** UI refinement in `lib/modules/sos/`.
* **Status:** RESOLVED [UX pattern defined]
* **Blocking:** None.

---

## 6. Infrastructure & Deployment

### OQ-21: Firebase Project Ownership
* **Category:** Infrastructure
* **Question:** Who configures and provides `google-services.json` (Android) and `GoogleService-Info.plist` (iOS) for FCM?
* **Context:** Push notification pipeline.
* **Recommended Default:** Backend / DevOps team sets up unified Firebase project; mobile devs inject config files.
* **Status:** OPEN [Pending DevOps / Client handoff]
* **Blocking:** Phase 5 push notification testing.

### OQ-22: Google Maps API Keys & Quota
* **Category:** Infrastructure
* **Question:** Which GCP project issues the Google Maps SDK keys, and what API restrictions / HTTP referrers are applied?
* **Context:** `MAPS_API_KEY` injected via `--dart-define`.
* **Recommended Default:** Separate staging and production restricted Android/iOS package keys.
* **Status:** OPEN [Pending DevOps / Client handoff]
* **Blocking:** Live Google Map rendering on device.

### OQ-23: Code Signing & Keystore Custody
* **Category:** Infrastructure
* **Question:** Where are the production upload keystores (Android `.jks`) and Apple Distribution certificates managed?
* **Context:** CI/CD signing configuration.
* **Recommended Default:** Managed securely via GitHub Actions Secrets / secure key vault; local builds use debug signing.
* **Status:** OPEN
* **Blocking:** Production release builds only.

### OQ-24: CI/CD Pipeline Host
* **Category:** Infrastructure
* **Question:** What CI/CD platform will execute automated builds and tests (GitHub Actions, GitLab CI, Azure Pipelines)?
* **Context:** Build pipeline script creation.
* **Recommended Default:** GitHub Actions standard workflow (`flutter analyze`, `flutter test`, `flutter build apk`).
* **Status:** OPEN
* **Blocking:** Automated deployment.

### OQ-25: Crash Telemetry & Privacy
* **Category:** Infrastructure
* **Question:** Which crash monitoring tool is approved under Red Cross privacy governance (Firebase Crashlytics, Sentry, or none)?
* **Context:** Crash breadcrumb sanitization (must not log PII).
* **Recommended Default:** Firebase Crashlytics with strict PII scrubbing.
* **Status:** OPEN
* **Blocking:** Release telemetry.

### OQ-26: Map SDK Confirmation
* **Category:** Infrastructure
* **Question:** Confirm `google_maps_flutter` as primary interactive map renderer?
* **Context:** Decision locked per architecture.
* **Recommended Default:** `google_maps_flutter` with fallback to static offline map when offline.
* **Status:** CONFIRMED (Recommended)
* **Blocking:** None.
