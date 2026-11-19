# Feature Plan: 01 — Foundation & Shell

**Milestone:** Phase 1  
**Status:** READY TO IMPLEMENT  
**Branch:** `feat/foundation-and-shell`  
**Target:** Clean compilation, passing tests, working 5-tab shell, theme tokens, database baseline, and localization wiring.

---

## 1. Objectives & Deliverables

1. Add verified production dependencies to `pubspec.yaml`.
2. Configure tri-lingual asset folders (`assets/data/`, `assets/images/`, `assets/fonts/`).
3. Build the design system token layer in `lib/theme/` (Red Cross brand `#ED1C24`, dark/light themes, 8pt spacing grid).
4. Implement core utilities in `lib/utils/` (`AppLogger` with PII stripping, `ResponseObject`, `Formatter`, `PhoneNumberSanitizer`).
5. Wire the cross-cutting services in `lib/services/` (`ConnectivityService`, `SecureStorageService`).
6. Initialize the SQLite database in `lib/config/database/db_config.dart` with schema v1.0.0.
7. Build the 5-tab persistent bottom navigation shell using `go_router` (`StatefulShellRoute` + `IndexedStack`) and prominent red SOS button.
8. Wire `main.dart` with `MultiProvider`, `go_router`, and tri-lingual `MaterialApp` localizations.

---

## 2. File Implementation Checklist

```
lib/
├── main.dart
├── config/
│   ├── environment.dart
│   ├── api_config.dart
│   ├── router/
│   │   └── app_router.dart
│   └── database/
│       ├── db_config.dart
│       └── schema_definitions.dart
├── constants/
│   ├── app_constants.dart
│   ├── assets.dart
│   └── route_names.dart
├── theme/
│   ├── app_colors.dart
│   ├── app_spacing.dart
│   ├── app_fonts.dart
│   ├── app_theme.dart
│   └── theme_controller.dart
├── utils/
│   ├── app_logger.dart
│   ├── response_object.dart
│   ├── formatter.dart
│   └── phone_sanitizer.dart
├── services/
│   ├── connectivity_service.dart
│   └── secure_storage_service.dart
├── widgets/
│   ├── app_button.dart
│   ├── app_card.dart
│   ├── app_empty_state.dart
│   ├── app_shimmer_card.dart
│   ├── status_badge.dart
│   └── connectivity_banner.dart
└── modules/
    └── shell/
        └── presentationlayer/
            └── pages/
                ├── app_shell.dart
                └── placeholder_tabs.dart
```

---

## 3. Step-by-Step Implementation Instructions

### Step 1: Update `pubspec.yaml`
Add required dependencies:
```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  cupertino_icons: ^1.0.8
  provider: ^6.1.2
  sqflite: ^2.3.3+1
  flutter_secure_storage: ^9.2.2
  shared_preferences: ^2.3.2
  go_router: ^14.6.2
  connectivity_plus: ^6.1.0
  url_launcher: ^6.3.1
  geolocator: ^13.0.1
  permission_handler: ^11.3.1
  intl: ^0.19.0
  uuid: ^4.5.1
  http: ^1.2.2

flutter:
  uses-material-design: true
  generate: true
  assets:
    - assets/data/
    - assets/images/
```

### Step 2: Build Design System Tokens (`lib/theme/`)
*(Detailed specifications, hex codes, and widget blueprints cataloged in [ui-ux-design-specification.md](file:///l:/.claude/docs/plans/ui-ux-design-specification.md))*
* **`app_colors.dart`**:
  * Brand Red Cross Red: `static const Color primaryRed = Color(0xFFED1C24);`
  * Status & Alert Levels:
    * Critical / Evacuation / Red Flag: `#DC2626` (Container: `#FEF2F2`)
    * High Warning / Landslide Watch: `#C2410C` (Container: `#FFF7ED`, Hero Banner: `#B44214`)
    * Moderate / Caution / Road Closed: `#D97706` (Container: `#FFFBEB`)
    * Safe / Verified / Low Risk: `#16A34A` (Container: `#F0FDF4`)
    * Info / Water / Flood: `#0284C7` (Container: `#F0F9FF`)
  * Neutral Surfaces:
    * Light Theme: Canvas `#F8F6F2`, Card `#FFFFFF`, Elevated `#F0EDE6`, Border `#EBE7DF`, Text `#141414` / `#52525B`
    * Dark Theme: Canvas `#121212`, Card `#1E1E1E`, Elevated `#282828`, Border `#2D2D2D`, Text `#F5F5F5` / `#A1A1AA`
    * Floating Bottom Nav Dock: `#1A1A1A` with glowing Red Cross Red SOS center CTA
* **`app_spacing.dart`**:
  * 8pt grid: `xxs = 2.0`, `xs = 4.0`, `sm = 8.0`, `md = 16.0`, `lg = 24.0`, `xl = 32.0`, `xxl = 48.0`.
  * `AppRadius`: `sm = 8.0`, `md = 12.0`, `card = 16.0`, `modal = 24.0`, `navBar = 32.0`, `pill = 999.0`.
* **`app_fonts.dart`**:
  * Material 3 scale: `displayLarge` (32pt Bold), `headlineLarge` (26pt Bold), `headlineMedium` (22pt Bold), `titleLarge` (18pt SemiBold), `titleMedium` (15pt SemiBold), `bodyLarge` (15pt Regular), `bodyMedium` (13pt Regular), `labelLarge` (15pt SemiBold), `labelSmall` (11pt Bold Uppercase tracking +0.8).
* **`app_theme.dart`**:
  * Implements `AppTheme.light` (warm off-white `#F8F6F2`) and `AppTheme.dark` (deep OLED `#121212`) using Material 3 `ColorScheme.fromSeed(seedColor: AppColors.primaryRed)`.
  * Configures `AppBarTheme`, `CardTheme`, `ElevatedButtonTheme`, and floating capsule `BottomNavigationBarTheme`.
* **`theme_controller.dart`**:
  * `ChangeNotifier` managing `ThemeMode` (`system`, `light`, `dark`), persisting selection to `shared_preferences`.

### Step 3: Implement Utility Standards (`lib/utils/`)
* **`app_logger.dart`**:
  * `debug()`, `info()`, `warning()`, `error()`. Strips PII, exact coordinates, and tokens. Inactive in release mode (`kDebugMode`).
* **`phone_sanitizer.dart`**:
  * `isSafeToDial(String number)`: Validates phone strings against `^\+?[0-9]{3,12}$`.
* **`response_object.dart`**:
  * `ResponseObject<T>` with `bool status`, `String? message`, `T? data`, `int? statusCode`.

### Step 4: Cross-Cutting Infrastructure (`lib/services/`, `lib/config/`)
* **`secure_storage_service.dart`**:
  * Configured with `encryptedSharedPreferences: true` (Android) and `first_unlock_this_device` (iOS).
  * Storage for ICE contacts, medical notes, and DB encryption keys.
* **`connectivity_service.dart`**:
  * Listens to `connectivity_plus` and broadcasts `ConnectivityStatus` (`online`, `offline`).
* **`db_config.dart`**:
  * Manages SQLite database instance, table creation from `schema_definitions.dart`, and migration runner.

### Step 5: Persistent 5-Tab Shell Navigation (`lib/modules/shell/`)
* **`app_router.dart`**:
  * `GoRouter` configuring a `StatefulShellRoute.indexedStack` with 5 branches:
    1. `/home` → `HomePage`
    2. `/map` → `SafetyMapPage`
    3. `/sos` → SOS Quick Action Sheet
    4. `/explore` → `ExplorePage`
    5. `/alerts` → `AlertsFeedPage`
* **`app_shell.dart`**:
  * Bottom navigation bar with customized central SOS tab:
    * Red floating/prominent circular icon button.
    * Tapping SOS opens confirmation modal without disrupting bottom nav state.

### Step 6: App Entry Point (`lib/main.dart`)
* Run initialization inside `runZonedGuarded`.
* Configure root `MultiProvider`:
  * `ChangeNotifierProvider<ThemeController>`
  * `Provider<ConnectivityService>`
  * `Provider<SecureStorageService>`
* Bind `MaterialApp.router` with `AppRouter.router`, `AppTheme.light`, `AppTheme.dark`, and localization delegates.

---

## 4. Verification & Acceptance Criteria

- [ ] `flutter pub get` succeeds without version conflicts.
- [ ] `flutter analyze` passes with 0 errors and 0 warnings.
- [ ] Running app loads directly into the 5-tab shell.
- [ ] Tapping between Home, Map, Explore, and Alerts preserves child screen state.
- [ ] Tapping central SOS displays the emergency confirmation modal sheet.
- [ ] SQLite database creates `safe_travel.db` with all 13 tables verified.
- [ ] Switching system theme toggles between light and dark modes cleanly.
