# Tech Stack

<!-- Filled in during project setup. Update whenever a real stack decision changes. -->

## Language(s) & Framework(s)

Dart 3 (SDK `>=3.4.0 <4.0.0`) / Flutter. Targets Android (primary) and iOS.

## Key Dependencies

- **State/DI**: `provider` — root `MultiProvider` in `main.dart` wires services into
  controllers; no other state-management library.
- **Local storage**:
  - `sqflite` — SQLite for structured offline data (safety guides, cached alerts, trip
    data, explore listings)
  - `flutter_secure_storage` — sensitive data (user preferences, emergency contacts)
  - `shared_preferences` — simple settings (theme, language, onboarding complete)
  - `hive` or `isar` — TBD: may be needed for offline content bundles (decision pending)
- **Networking**: `http` — hand-rolled REST client for online features (weather API,
  alerts API, explore API). No codegen, no `dio`, no `retrofit`.
- **Maps/location**: `google_maps_flutter` or `flutter_map` (TBD), `geolocator`,
  `geocoding`
- **Notifications**: `flutter_local_notifications` for offline alerts,
  `firebase_messaging` for push notifications (online)
- **Permissions**: `permission_handler`
- **Phone dialer**: `url_launcher` for SOS one-tap call
- **Connectivity**: `connectivity_plus` for online/offline detection
- **Internationalization**: `intl`, `flutter_localizations`
- **Lint**: `flutter_lints` (default ruleset to start)

## Infrastructure

- **Backend**: external API for online features (weather alerts, explore directory,
  hazard data). Offline content is bundled in the app.
- **Firebase**: push notifications (FCM), analytics (optional). No Firebase Auth —
  the app works without login for core features.
- **Offline content**: JSON/SQLite bundles in `assets/` for safety guides, first-aid
  instructions, emergency numbers, safety topics.

## Development Environment

- Standard Flutter tooling (`flutter run`, `flutter build apk`/`ipa`).
- Secrets/config via `--dart-define` at build time and a gitignored
  `lib/config/environment.dart` for local development.
- Environment variables: `ENV` (dev/staging/prod), `MAPS_API_KEY`,
  `WEATHER_API_KEY`, Firebase config keys.
