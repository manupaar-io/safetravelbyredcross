# Project Structure

<!-- Must stay accurate. If a file/folder moves or the structure changes, update this
     file in the SAME task that makes the change. -->

## Folder Layout

```
lib/
  main.dart                    # App entry, DI wiring (MultiProvider), lifecycle handling
  config/
    environment.dart           # AppEnvironment: build-time secrets/flags (--dart-define)
    api_config.dart            # API base URLs for online services
    database/db_config.dart    # SQLite setup/migrations
  constants/
    app_constants.dart         # App-wide constants (k-prefixed)
    assets.dart                # Asset paths
    route_names.dart           # Named route constants
  theme/
    app_colors.dart            # AppColors — Red Cross brand + status colors
    app_spacing.dart           # AppSpacing/AppRadius — 8pt-grid fixed dp scale
    app_fonts.dart             # AppFonts/AppTextStyles — type scale
    app_theme.dart             # AppTheme.light/.dark — ThemeData for both brightnesses
    theme_controller.dart      # ThemeController — app-wide ThemeMode
  services/
    connectivity_service.dart  # Network state monitoring
    location_service.dart      # GPS location
    weather_service.dart       # Weather data fetching
    offline_content_service.dart # Bundled content loading/caching
    notification_service.dart  # Local + push notifications
  utils/
    app_logger.dart            # Centralized logging (use instead of print)
    formatter.dart             # Display formatting
    parser_utils.dart          # Tolerant JSON parsing
    response_object.dart       # ResponseObject — universal service-call result type
  widgets/
    app_card.dart, app_button.dart, app_empty_state.dart, ...
                               # Shared UI components (the reuse layer)
  modules/
    onboarding/
      presentationlayer/pages/        # WelcomePage, OnboardingPage, PermissionPage, AllSetPage
    home/
      presentationlayer/{controller,pages,widgets}/
    safety_map/
      datalayer/{models}/
      domainlayer/
      presentationlayer/{controller,pages,widgets}/
    explore/
      datalayer/{dto,mappers,models}/
      domainlayer/
      presentationlayer/{controller,pages,widgets}/
    safety_topics/
      datalayer/{models,repositories}/
      domainlayer/
      presentationlayer/{controller,pages,widgets}/
    first_aid/
      datalayer/{models,repositories}/
      domainlayer/
      presentationlayer/{controller,pages,widgets}/
    sos/
      presentationlayer/{controller,pages}/
    trip/
      datalayer/{models,repositories}/
      domainlayer/
      presentationlayer/{controller,pages,widgets}/
    alerts/
      datalayer/{dto,mappers,models}/
      domainlayer/
      presentationlayer/{controller,pages,widgets}/
    profile/
      presentationlayer/{controller,pages}/
android/, ios/                  # Platform projects
assets/
  images/                      # App images, icons, illustrations
  data/                        # Bundled offline content (JSON/SQLite)
  fonts/                       # Custom fonts
.claude/
  CLAUDE.md                    # Project instructions index
  AGENTS.md                    # Agent rules and session protocol
  docs/                        # Doc set (project-brief, tech-stack, architecture, etc.)
    security-agent-guidelines.md # Agent security rules & pre-commit gates
SECURITY.md                    # Project-wide security policy & threat model
```

## Responsibilities by Folder

| Folder | Belongs there |
|---|---|
| `lib/modules/<feature>/datalayer/` | DTOs (raw JSON parsing), models (UI-facing shape), mappers (DTO→Model), repositories (local DB access) |
| `lib/modules/<feature>/domainlayer/` | Services: REST calls, business rules, returns `ResponseObject` |
| `lib/modules/<feature>/presentationlayer/controller/` | `ChangeNotifier` controllers — UI state + orchestration, constructed per-page |
| `lib/modules/<feature>/presentationlayer/pages/` | Full-screen widgets (routes) |
| `lib/modules/<feature>/presentationlayer/widgets/` | Feature-scoped reusable widgets (not shared app-wide) |
| `lib/widgets/` | App-wide shared widgets only |
| `lib/services/` | Cross-module infrastructure services (not tied to one feature) |
| `lib/config/` | Runtime/build-time configuration, not business logic |
| `lib/utils/` | Stateless helpers used across modules |
| `lib/theme/` | Design-system tokens and theme data |
| `lib/constants/` | App-wide constant values |
