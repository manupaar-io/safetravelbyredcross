# System Design Document (SDD) — Safe Travel Sri Lanka

**Version:** 1.0.0-draft  
**Date:** 2026-11-19  
**Status:** DRAFT (Ready for Implementation; .NET Core API Integration Abstracted)  
**Affiliation:** Sri Lanka Red Cross Society  
**Author:** Software Architecture Team  

---

## 1. Executive Summary & Purpose

**Safe Travel Sri Lanka** (`safe_travel_sri_lanka`) is a mobile travel-safety, disaster preparedness, and emergency response platform built in Flutter/Dart. It serves international tourists, humanitarian staff, and domestic travelers navigating Sri Lanka.

### Core Architectural Mandates
1. **Offline-First Resilience:** Disaster scenarios frequently knock out cellular towers and internet backbones. Critical life-safety guides, first-aid instructions, emergency hotlines, and go-bag checklists are physically bundled inside the app client and guaranteed to operate with zero connectivity.
2. **Stress-Ready UX:** High-contrast visual tokens, minimum 48dp touch targets, zero mental friction, and a 0ms pre-warmed emergency SOS dialer.
3. **Tri-Lingual Parity (Decision D10):** Native support for English (`en`), Sinhala (`si`), and Tamil (`ta`).
4. **Pragmatic Simplicity (Anti-Overengineering / KISS):** Strict 3-layer architecture (`datalayer/` → `domainlayer/` → `presentationlayer/`), plain `ChangeNotifier` route controllers, no speculative abstractions.

---

## 2. System Context & Topology

```mermaid
flowchart TD
    subgraph MobileClient["Flutter Mobile Client (Android & iOS)"]
        UI["Presentation Layer (Widgets & Controllers)"]
        Domain["Domain Services (Offline & Online Orchestration)"]
        DataLocal["Local Storage (SQLite + SecureStorage)"]
        DataRemote["Hand-Rolled SecureHttpClient"]
        UI --> Domain
        Domain --> DataLocal
        Domain --> DataRemote
    end

    subgraph BackendAPI[".NET Core RESTful API"]
        Gateway["API Gateway / Controller Endpoints"]
        AuthSvc["Auth & Token Service (JWT)"]
        AlertsEngine["Alerts & Hazard Aggregator"]
        DirectoryEngine["Explore & Safety Directory"]
        WeatherEngine["Meteorological Ingestion Service"]
        Gateway --> AuthSvc
        Gateway --> AlertsEngine
        Gateway --> DirectoryEngine
        Gateway --> WeatherEngine
    end

    subgraph ExternalSources["National Authorities & Cloud Services"]
        NBRO["NBRO (Landslide Early Warning)"]
        DMC["Disaster Management Centre (DMC)"]
        MetDept["Department of Meteorology"]
        FCM["Firebase Cloud Messaging (FCM Push)"]
        GMaps["Google Maps Platform API"]
    end

    DataRemote <==>|HTTPS / TLS 1.3 Pinning| Gateway
    AlertsEngine <--> NBRO
    AlertsEngine <--> DMC
    WeatherEngine <--> MetDept
    Gateway -.->|Push Notifications| FCM
    FCM -.->|Emergency Siren Alerts| MobileClient
    UI -.->|Tile Rendering| GMaps
```

---

## 3. Layered Module Architecture

Each feature module under `lib/modules/<feature>/` strictly implements a 3-layer boundary:

```mermaid
graph TD
    subgraph PresentationLayer["Presentation Layer (Scoped to Page Lifecycle)"]
        Page["*Page (Route Widget)"]
        Controller["*Controller (ChangeNotifier)"]
        Widgets["*Widgets (Feature-specific leaf widgets)"]
        Page --> Controller
        Page --> Widgets
    end

    subgraph DomainLayer["Domain Layer (Business Logic & Orchestration)"]
        Service["*Service (Coordinates Cache & Remote API)"]
        Controller -->|calls| Service
    end

    subgraph DataLayer["Data Layer (Persistence & Serialization)"]
        Repo["*Repository (SQLite parameterized queries)"]
        Dto["*Dto (Raw JSON parser with ?? fallbacks)"]
        Mapper["*Mapper (Dto -> Model translation)"]
        Model["*Model (Immutable UI-facing state)"]
        HttpClient["SecureHttpClient (Hand-rolled REST client)"]

        Service --> Repo
        Service --> HttpClient
        HttpClient --> Dto
        Dto --> Mapper
        Mapper --> Model
        Repo --> Model
    end
```

---

## 4. Entity-Relationship Diagram (ERD)

The complete local relational schema executed in SQLite (`safe_travel.db`). Encrypted via SQLCipher for user personal items.

```mermaid
erDiagram
    %% =========================================================
    %% BUNDLED OFFLINE STATIC CONTENT
    %% =========================================================
    safety_topics ||--o{ safety_topic_details : contains
    safety_topics {
        int id PK
        string category_key UK "floods_landslides, beach_safety, etc."
        string title_en
        string title_si
        string title_ta
        string description_en
        string description_si
        string description_ta
        string icon_asset_path
        int sort_order
        int is_active
        string created_at
        string updated_at
    }

    safety_topic_details {
        int id PK
        int topic_id FK
        string section_title_en
        string section_title_si
        string section_title_ta
        string content_en
        string content_si
        string content_ta
        string media_type "text, image, video_link"
        string media_url
        int sort_order
    }

    first_aid_guides ||--o{ first_aid_steps : prescribes
    first_aid_guides {
        int id PK
        string guide_code UK "FA_SNAKEBITE, etc."
        string title_en
        string title_si
        string title_ta
        string severity "critical, moderate, minor"
        string search_keywords
        int sort_order
        int is_active
    }

    first_aid_steps {
        int id PK
        int guide_id FK
        int step_number
        string instruction_en
        string instruction_si
        string instruction_ta
        string image_asset_path
        string warning_text_en
        string warning_text_si
        string warning_text_ta
    }

    emergency_numbers {
        int id PK
        string service_code UK "POLICE_119, AMBULANCE_1990"
        string service_name_en
        string service_name_si
        string service_name_ta
        string phone_number
        string category "police, medical, fire, disaster, redcross"
        int is_primary
        string icon_asset_path
        int sort_order
    }

    go_bag_items ||--o{ user_go_bag_checks : checks
    go_bag_items {
        int id PK
        string item_code UK
        string item_name_en
        string item_name_si
        string item_name_ta
        string category "documents, medical, clothing, food_water, tools"
        string description_en
        int is_essential
        int sort_order
    }

    %% =========================================================
    %% CACHED ONLINE CONTENT (SWR)
    %% =========================================================
    hazard_alerts {
        string id PK "Remote Server GUID"
        string title_en
        string title_si
        string title_ta
        string description_en
        string severity "critical, high, moderate, low, info"
        string hazard_type "flood, landslide, storm, tsunami"
        string affected_area_name
        real latitude
        real longitude
        real radius_km
        string source "NBRO, DMC, SLRC"
        string issued_at
        string expires_at
        int is_active
        string fetched_at
        string raw_json
    }

    weather_cache {
        string district_key PK "colombo, galle, kandy"
        string district_name_en
        real temperature_celsius
        real feels_like_celsius
        int humidity_percent
        real wind_speed_kmh
        string condition_code
        string condition_text_en
        string icon_url
        string fetched_at
        string expires_at
    }

    explore_listings {
        string id PK "Remote Server GUID"
        string name
        string category "accommodation, food, medical, transport, embassy"
        string description_en
        string address
        real latitude
        real longitude
        string phone
        real rating
        int is_indoor
        int is_emergency_facility
        string fetched_at
    }

    map_points {
        string id PK
        string type "hazard_zone, safe_zone, redcross_point, shelter"
        string label_en
        real latitude
        real longitude
        real radius_km
        string severity
        string metadata_json
        int is_active
        string fetched_at
    }

    %% =========================================================
    %% USER PRIVATE DATA (SQLCipher Encrypted)
    %% =========================================================
    user_trips ||--o{ user_go_bag_checks : tracks
    user_trips {
        string id PK "Client UUIDv4"
        string trip_name
        string destination
        string start_date
        string end_date
        string notes
        string created_at
    }

    user_go_bag_checks {
        int id PK
        string trip_id FK "Nullable: master or trip-specific"
        int item_id FK
        int is_checked
        string checked_at
    }

    content_sync_metadata {
        string entity_type PK
        string last_synced_at
        string etag
        int record_count
        string sync_status
    }
```

---

## 5. Sequence Flows & Operational Protocols

### 5.1 Stale-While-Revalidate (SWR) Pattern for Weather & Alerts
Prevents UI spin-blocks by providing instant cached data to the screen while quietly refreshing from the .NET Core backend.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Page as HomePage UI
    participant Ctrl as HomeController
    participant Svc as WeatherService
    participant Repo as WeatherRepository (SQLite)
    participant API as .NET Core Weather API

    User->>Page: Opens Home Screen
    Page->>Ctrl: loadDashboard()
    Ctrl->>Svc: getWeatherForDistrict("galle")
    Svc->>Repo: queryCache("galle")
    Repo-->>Svc: returns CachedWeather (last updated 15m ago)
    Svc-->>Ctrl: Emit Cached Data (immediate)
    Ctrl-->>Page: Render Weather widget (zero spinner)
    
    rect rgb(240, 248, 255)
        note over Svc,API: Asynchronous Background Revalidation
        Svc->>API: GET /api/v1/weather?district=galle
        alt Network Success
            API-->>Svc: 200 OK (Fresh Payload)
            Svc->>Repo: updateCache("galle", freshPayload)
            Svc-->>Ctrl: Emit Fresh Data
            Ctrl-->>Page: Smooth leaf widget rebuild
        else Network Unavailable / Offline
            Svc->>Svc: Catch SocketException
            Note over Ctrl,Page: Continue displaying cached weather with "Updated 15m ago"
        end
    end
```

### 5.2 Zero-Latency Emergency SOS Flow
Guarantees emergency hotlines are accessible within 0ms without waiting for disk I/O, database initialization, or network availability.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant SOSBtn as SOS Button (Red Center Tab)
    participant Shell as AppShell Navigation
    participant Sheet as DialConfirmationSheet
    participant RAM as Synchronous RAM Cache (Hotlines)
    participant Sanitizer as PhoneNumberSanitizer
    participant OS as Native OS Phone Dialer (url_launcher)

    Note over RAM: Pre-warmed at main() boot: 119, 110, 1990, 117
    User->>SOSBtn: Taps Center Red SOS Button
    SOSBtn->>Shell: Trigger SOS Action
    Shell->>RAM: Read Hotline Numbers (0ms synchronous RAM read)
    RAM-->>Sheet: Instant Numbers Data
    Shell->>Sheet: Show Modal Bottom Sheet
    User->>Sheet: Taps "Call 1990 Ambulance"
    Sheet->>Sanitizer: isSafeToDial("1990")
    Sanitizer->>Sanitizer: Validate against ^\\+?[0-9]{3,12}$
    Sanitizer-->>Sheet: Regex Match Valid (Pass)
    Sheet->>OS: launchUrl("tel:1990")
    OS-->>User: Open Native Phone Dialer UI
```

---

## 6. Abstracted API Integration Layer

Because the external .NET Core API documentation is pending delivery, all network interactions are cleanly insulated behind domain interfaces:

| Expected Endpoint | HTTP Method | Client Service Method | Local Cache Repository |
|---|:---:|---|---|
| `/api/v1/alerts/active` | `GET` | `AlertsService.getActiveAlerts()` | `HazardAlertsRepository` |
| `/api/v1/weather` | `GET` | `WeatherService.getWeather(district)` | `WeatherCacheRepository` |
| `/api/v1/explore` | `GET` | `ExploreService.getListings(coords, radius)` | `ExploreRepository` |
| `/api/v1/map/points` | `GET` | `SafetyMapService.getMapOverlays()` | `MapPointsRepository` |
| `/api/v1/content/sync` | `GET` | `OfflineContentService.syncDelta()` | `ContentSyncRepository` |
| `/api/v1/safety/broadcast` | `POST` | `SosService.broadcastImSafe(telemetry)` | Transmitted live or fallback to SMS |
| `/api/v1/auth/login` | `POST` | `AuthService.login(credentials)` | `SecureStorageService` (Tokens) |
| `/api/v1/auth/refresh` | `POST` | `AuthService.refreshToken()` | `SecureStorageService` (Tokens) |

---

## 7. Performance & Quality Benchmarks

| Metric | Target Budget | Enforcement Mechanism |
|---|---|---|
| **Cold Start to First Frame** | `< 1.2s` | Zero async blocking in `main()`; DB and background sync warm up after first paint |
| **SOS Action to Dialer** | `0ms` | Pre-warmed in-memory RAM cache at startup |
| **Tab Switch Latency** | `< 16ms (60fps)` | `go_router` StatefulShellRoute + `IndexedStack` preserving sub-trees |
| **Offline Content Lookup** | `< 100ms` | Indexed SQLite B-Tree queries with limit/offset |
| **Heavy JSON Processing** | Background | `Isolate.run()` for all payloads > 50KB |
