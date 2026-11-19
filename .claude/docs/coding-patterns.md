# Coding Patterns — Dart / Flutter

<!--
This file pins the global `coding-standards` skill to this project's actual stack.
The global skill defines universal principles (clear naming, no unnecessary comments,
avoid deep nesting, etc.) — this file defines the CONCRETE idiom for this
language/framework.

Keep this current: if a new pattern is introduced or an existing one changes, update
this file as part of that same change, with the user's confirmation.
-->

## Stack

Dart 3 / Flutter, `provider` for DI, `sqflite` + `flutter_secure_storage` for local
persistence, hand-rolled `http` REST client (no codegen, no `dio`, no `retrofit`).

## Core Principle: Pragmatic Simplicity & Anti-Overengineering (KISS / YAGNI)

- **Build what is actually needed now**: Avoid speculative complexity, unneeded abstractions,
  generic wrapper layers, or architectural patterns introduced "just in case".
- **Immediate readability**: Any developer opening any file must be able to understand what it
  does in 60 seconds without mental gymnastics. Clear, explicit Dart over clever, indirect code.
- **Save resources, time, and clarity**:
  - Don't add extra classes, interfaces, or layers when a direct function or straightforward
    class solves the problem cleanly.
  - Follow the 3-layer module convention (`datalayer/`, `domainlayer/`, `presentationlayer/`),
    but only create the folders a module actually uses (e.g. no DTOs for local-only SQLite features).
  - Prefer flat, simple widgets over deeply nested widget hierarchies.
  - If something feels over-engineered, simplify it immediately.

## Naming Conventions

- Files: `snake_case.dart`. Classes: `PascalCase`. Members/variables/functions:
  `lowerCamelCase` — standard Dart convention, followed consistently.
- Suffix by role: `*Dto` (raw parse), `*Model` (UI-facing), `*Mapper` (DTO→Model),
  `*Repository` (local DB access), `*Service` (REST/business logic), `*Controller`
  (`ChangeNotifier` presentation state), `*Page` (route-level widget).
- Constants: `AppConstants.kXxx` (`k`-prefixed) for app-wide constants
  (`kDefaultFallbackText`, `kDefaultStatusColor`, etc.).

## Module / Folder Organization

See `.claude/docs/project-structure.md` for the concrete tree. Every feature module under
`lib/modules/<feature>/` follows `datalayer/ -> domainlayer/ -> presentationlayer/`; only
create the subfolders a module actually needs (e.g. no `dto/` for SQLite-only modules).

## Layering Pattern

`Page (widget) -> Controller (ChangeNotifier) -> Service (domainlayer) -> Dto/Model +
Mapper (datalayer) -> Repository (SQLite) or http (REST)`.

- **Page**: builds UI, listens to controller via `ChangeNotifierProvider`/`Consumer` or
  is constructed with the controller directly; no business logic.
- **Controller**: holds UI state (loading flags, form controllers, error messages),
  orchestrates calls to one or more services, calls `notifyListeners()` after state
  changes, disposes its own `TextEditingController`/`FocusNode`/`AnimationController`
  instances in `dispose()`.
- **Service**: makes the REST call or coordinates repositories; returns `ResponseObject`
  for expected outcomes (success/business-failure); lets real exceptions propagate to be
  caught by the controller.
- **Dto**: `factory Dto.fromJson(Map<String, dynamic>)` — tolerant parsing only, with
  inline `??` fallbacks for alternate JSON shapes. Keeps `rawData` for anything not
  modeled — **except** when the source response carries credential/secret fields.
- **Mapper**: static `toModel(dto)` — applies display fallbacks
  (`AppConstants.kDefaultFallbackText`) and formatting (`Formatter.*`). No parsing logic
  here, no fallback logic in the DTO.
- **Model**: plain data holder consumed by widgets; for SQLite-backed data, the model
  itself exposes `fromMap`/`toMap` instead of going through a DTO.

## Conditional Dispatch Idiom

Default to Dart's `switch` expression for anything beyond 2–3 branches over long
`if/else if` chains.

## Error Handling Pattern

- **Expected/business failures** (bad network response, validation): service returns
  `ResponseObject(status: false, message: '...')`; controller reads `.status`/`.message`,
  no exception thrown.
- **Unexpected failures** (network error, timeout, parse error): `try/catch` at the
  service boundary, converted into a `ResponseObject` with a user-facing message — don't
  let raw exceptions reach the controller/UI.
- **Global safety net**: `runZonedGuarded` in `main.dart` catches uncaught async errors;
  `FlutterError.onError` catches widget/rendering errors. Both log via `AppLogger.error`.
  Don't rely on these for expected control flow.
- All error/info logging goes through `AppLogger` (`lib/utils/app_logger.dart`), not
  `print` or bare `developer.log`.

## Dependency Injection / Composition Style

- Root services (`ConnectivityService`, `LocationService`, `WeatherService`,
  `OfflineContentService`) are constructed once in `main.dart`'s `MultiProvider`.
- Controllers are **not** provided globally — they're constructed per-page (typically in
  a `StatefulWidget.initState` or via `ChangeNotifierProvider(create: ...)` scoped to that
  route), pulling already-provided services via `context.read<T>()`/constructor params.
  **Exception**: `ThemeController` is provided globally since `MaterialApp` itself needs it.
- No service/controller should instantiate a dependency it could instead receive —
  constructor injection is the norm.

## UI Theme / Color Palette

- **Design-system tokens**: `AppColors.seed` (Red Cross red `#ED1C24`) feeds the app's
  `ColorScheme`. Prefer `Theme.of(context).colorScheme.*` over importing `AppColors`
  directly in a widget. `AppColors` also holds status colors for hazard levels.
- Spacing/radius from `AppSpacing`/`AppRadius` (8pt-grid fixed dp values).
- Shared widgets (`AppCard`, `AppButton`, `AppEmptyState` in `lib/widgets/`) are the
  reuse layer — no screen hand-rolls its own button/card styling.
- **Text**: use `Theme.of(context).textTheme` for all text styles.
- **Dark mode**: support both light and dark themes from the start
  (`ThemeMode.system` default).

## Comment & Docstring Conventions

Inherits the global "WHY not WHAT, mandatory only for non-obvious cases" rule. Use `///`
doc comments sparingly on public service methods — brief, one line, describing intent
rather than restating the signature.

## Testing Conventions

Follow standard Flutter convention: `test/` mirroring `lib/` structure, `*_test.dart`
naming. Widget tests for key UI components, unit tests for services and controllers.

## Performance Engineering Standards (Life-Critical Latencies)

In an emergency safety app, slowness is a critical safety hazard. Every agent and developer
must enforce these rules:

1. **Non-Blocking Startup (< 1.2s Cold Start)**:
   - `main()` must do the bare minimum before `runApp()`. Heavy services (SQLite DB, bundled JSON,
     location warmup) initialize asynchronously *after* the initial frame renders.
2. **Main Thread Protection (`Isolate.run`)**:
   - Never decode or transform JSON payloads larger than 50KB on the root UI isolate. Offload to
     `Isolate.run()` to prevent frame drops and touch latency.
3. **Zero-Spinner Strategy (Stale-While-Revalidate)**:
   - Never show a blank screen or blocking full-page spinner if cached data exists in SQLite.
   - Render cached weather/alerts immediately with a "Last updated" indicator; fetch fresh data
     silently in the background.
4. **Pre-Warmed SOS & Emergency Hotlines (0ms Latency)**:
   - Critical emergency numbers (119, 110, 1990, DMC 117, Red Cross) are loaded into synchronous
     RAM cache at boot. Tapping SOS must never wait on disk I/O or network.
5. **Widget Rebuild Minimization**:
   - Use `const` constructors on every immutable widget and style object.
   - Use scoped, granular `Consumer` / `Selector` widgets at leaf nodes; do not wrap whole `Scaffold`s.
   - Wrap pulsating or animated widgets (SOS rings, flashing alert badges) in `RepaintBoundary`.
   - Preserve bottom-nav tab state with `IndexedStack` or `PageStorage`.
6. **Battery & Radio Conservation**:
   - Geolocation uses `LocationAccuracy.medium` and `distanceFilter: 250` for background/weather;
     only use high accuracy when actively viewing the map or dispatching SOS.
   - Exponential backoff on network failures (5s → 15s → 45s → 2m) to avoid draining battery in
     disaster zones.
7. **Image RAM Bounds**:
   - Specify `cacheWidth` or `cacheHeight` on `Image.asset` to prevent full-resolution GPU texture
     bloat and OOM crashes on low-to-mid range devices.
