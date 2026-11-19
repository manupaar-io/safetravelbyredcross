# Safe Travel Sri Lanka 🇱🇰

An offline-first mobile safety companion and emergency response platform developed in affiliation with the **Sri Lanka Red Cross Society**.

> **Humanitarian Design Ethos:** *"Useful under stress, simple under calm."*

[![Architecture Review](https://img.shields.io/badge/Architecture-Review%20Ready-green.svg)](file:///l:/SE/vscode/safetravelbyredcross/SYSTEM_DESIGN_REVIEW.md)
[![Platform](https://img.shields.io/badge/Platform-Flutter%20%7C%20Android%20%26%20iOS-blue.svg)](https://flutter.dev)
[![Offline Core](https://img.shields.io/badge/Offline--First-100%25%20Bundled-red.svg)](file:///l:/SE/vscode/safetravelbyredcross/SYSTEM_DESIGN_REVIEW.md#4-database-design--entity-relationship-diagram-erd)
[![Design Prototype](https://img.shields.io/badge/Prototype-Live%20Preview-orange.svg)](https://safetravelbyredcross.netlify.app/)

---

## 📌 Technical Architecture & System Design Document
For the complete technical specification and architecture details, please see:
👉 **[SYSTEM_DESIGN_REVIEW.md](file:///l:/SE/vscode/safetravelbyredcross/SYSTEM_DESIGN_REVIEW.md)**

It includes:
- **System Topology Diagram**: Cross-platform Flutter client ↔ .NET Core Backend ↔ National Disaster Authorities (NBRO, DMC, Met Dept).
- **Clean 3-Layer Client Architecture**: `presentationlayer/` → `domainlayer/` → `datalayer/` with route-scoped `ChangeNotifier` controllers.
- **Entity-Relationship Diagram (ERD)**: Complete SQLite relational model (bundled offline assets, SWR dynamic cache, SQLCipher encrypted private user tables).
- **Theoretical Standard Flows**: Stale-While-Revalidate (SWR) for real-time weather & hazard alerts, and low-latency pre-warmed emergency SOS dialer sequence.
- **Service Interfaces & API Contracts**: Decoupled domain interfaces and target endpoint specifications for flexible, maintainable integration.

---

## 📱 Key Features & Modules

| Module | Core Functionality | Execution Tier | Priority |
|---|---|:---:|:---:|
| **Emergency SOS Hub** | Low-latency pre-warmed hotline dialer (119, 1990, 110, 117), regex security guard against USSD injection, and GSM "I'm Safe" broadcast. | Offline (RAM / Tel / SMS) | **P0** |
| **First Aid & Safety Guides** | Step-by-step swipeable emergency manuals (IFRC aligned) covering 8 core hazards (landslides, floods, riptides, snakebites). | Offline (SQLite B-Tree) | **P0** |
| **Real-Time Hazard Alerts** | Instant local hazard notifications triaged by severity (Critical, High, Moderate), sourced from NBRO & DMC. | SWR Hybrid Cache | **P1** |
| **Adaptive Weather Dashboard** | Live meteorological updates and warning banners. Severe weather automatically shifts the UI to highlight emergency shelters. | SWR Hybrid Cache | **P1** |
| **Interactive Safety Map** | Google Maps overlays of danger zones, evacuation centers, safe hospitals, and district Red Cross branches (with offline vector fallback). | Online + Offline Fallback | **P1** |
| **Trip Go-Bag Checklist** | Interactive emergency packing and survival supplies tracker with persistent local state. | Encrypted SQLite (AES-256) | **P1** |
| **Tri-Lingual Parity** | Native English (`en`), Sinhala (`si`), and Tamil (`ta`) across all guides, alerts, and navigation. | Offline Bundled | **P0** |

---

## 🏗️ Architectural Overview

```
lib/
├── config/              # Environment variables, Router (go_router), SQLite init & migrations
├── constants/           # Route names, asset paths, application constants
├── theme/               # Design tokens (Red Cross #ED1C24, 8pt spacing grid, M3 typography)
├── utils/               # AppLogger (PII-stripped), PhoneSanitizer, ResponseObject wrapper
├── services/            # Cross-cutting: ConnectivityService, SecureStorageService
├── widgets/             # Core shared UI components (AppButton, AppCard, StatusBadge)
└── modules/             # Feature-isolated 3-layer architecture
    ├── shell/           # 5-Tab Persistent Navigation (StatefulShellRoute + IndexedStack)
    ├── sos/             # Pre-warmed emergency hotlines & confirmation sheets
    ├── first_aid/       # Offline procedural emergency medical cards
    ├── safety_topics/   # 8 categorized disaster preparedness guides
    ├── home/            # Weather card, hazard alert banner, quick access grid
    ├── alerts/          # Chronological disaster warning feed
    ├── explore/         # Verified facilities directory with weather-adaptive mode
    ├── safety_map/      # Google Maps + offline district fallback map
    ├── trip/            # Itinerary management & Go-Bag survival checklist
    └── profile/         # In Case of Emergency (ICE) contacts & language switch
```

---

## 🚀 Getting Started

### Prerequisites
- [Flutter SDK](https://docs.flutter.dev/get-started/install) (`>= 3.10.4`)
- Dart SDK (`>= 3.4.0 < 4.0.0`)
- Android Studio / Xcode / VS Code with Flutter extensions
- Physical mobile device or emulator

### Installation & Run

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/safetravelbyredcross.git
   cd safetravelbyredcross
   ```

2. **Install dependencies:**
   ```bash
   flutter pub get
   ```

3. **Verify linting & static analysis:**
   ```bash
   flutter analyze
   ```

4. **Launch on connected device:**
   ```bash
   flutter run
   ```

---

## 🔒 Security & Performance Constraints
- **Performance:** Cold launch to interactive `< 1.2s`, SOS dialer `< 0ms` RAM lookup, tab switching `< 16ms` (60fps).
- **Data Protection:** Zero cleartext HTTP traffic allowed. User ICE contacts and medical data are encrypted on device via SQLCipher and Android KeyStore / iOS Keychain (`flutter_secure_storage`).
- **Telemetry:** Zero PII logging in production builds.

---

## 📚 Technical Documentation Index
- **[SYSTEM_DESIGN_REVIEW.md](file:///l:/SE/vscode/safetravelbyredcross/SYSTEM_DESIGN_REVIEW.md)** — Comprehensive architecture review document with full Mermaid diagrams, ERD, and open questions.
- **[.claude/docs/plans/database-schema.md](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/database-schema.md)** — Complete SQLite DDL and migration strategy.
- **[.claude/docs/plans/open-questions.md](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/open-questions.md)** — Full 26-item open questions register.
- **[.claude/docs/plans/feature-analysis.md](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/plans/feature-analysis.md)** — Module-by-module breakdown.
- **[.claude/docs/PROGRESS.md](file:///l:/SE/vscode/safetravelbyredcross/.claude/docs/PROGRESS.md)** — Project roadmap and confirmed architectural decisions (D0–D11).
