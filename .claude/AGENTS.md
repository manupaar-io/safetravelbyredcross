# AGENTS.md: Safe Travel Sri Lanka

Rules for any coding agent working in this project. Read at the start of every session:
global rules, `CLAUDE.md`, this file, and the relevant `.claude/docs/` files.

Authority order if rules conflict: user's global rules > project rules (`CLAUDE.md`,
this file, `.claude/docs/` standards) > reference projects. If two rules still conflict,
stop and ask.

---

## 1. Mission and Principles

- Build a **mobile travel-safety and emergency readiness app** for Sri Lanka, affiliated
  with the Sri Lanka Red Cross Society. Target users: international tourists and local
  travelers.
- **Offline-first**: critical safety content works without connectivity. Online features
  enhance but are never required for core safety functionality.
- **Useful under stress**: the app must work when the user is panicking, when connectivity
  is gone, when they're in an unfamiliar place. Every UI decision serves this principle.
- **Pragmatic simplicity (Anti-Overengineering)**: Never build speculative complexity,
  unneeded wrapper layers, or convoluted abstractions. Focus strictly on what is needed now
  to save resources, time, and clarity. Code must be immediately readable and maintainable by any dev.
- Production-grade in everything shipped: tested, documented, maintainable, accessible.
- One feature per session. Sessions are independent: never rely on memory, read
  `.claude/docs/PROGRESS.md`.

## 2. Architecture Rules

- **Offline-first architecture**: safety guides, emergency numbers, first-aid manuals are
  bundled locally. Online data (weather, alerts, explore) is cached aggressively.
- **Feature modules** (`lib/modules/<feature>/`): each with `datalayer/`, `domainlayer/`,
  `presentationlayer/` as needed (same pattern as RoadRunner).
- **State management**: `provider` for DI, `ChangeNotifier` controllers per page. No other
  state management library.
- **Local storage**: `sqflite` for structured offline data, `flutter_secure_storage` for
  sensitive data, `shared_preferences` for settings.
- **No codegen REST clients**: hand-rolled `http`-based client. Clean, explicit, debuggable.
- **Context-aware UI**: the app adapts layout and recommendations based on active hazard
  warnings (normal mode vs warning mode).

## 3. Quality Bar (Every Feature)

- Follow all rules in `.claude/docs/security-agent-guidelines.md` and `SECURITY.md`.
- Follow performance rules in `.claude/docs/coding-patterns.md`: non-blocking startup, `const` widgets, zero-spinner SWR caching, `Isolate.run` for >50KB payloads.
- Pragmatic simplicity: no over-engineering, no unnecessary boilerplate, only what is actually needed.
- Every new screen handles: loading state, empty state, error state, offline state.
- Offline content is tested without connectivity.
- No secrets in code or git; only example files with fake values.
- All logging through a centralized logger (`AppLogger`), never `print` or bare logs. Zero PII, tokens, or raw GPS in logs.
- SQLite queries must be parameterized (`whereArgs`); zero string interpolation.
- Sensitive data (ICE contacts, tokens) in `flutter_secure_storage`; non-sensitive flags in `shared_preferences`.
- Ask before adding a dependency. Pin versions.

## 4. Session Protocol

**Feature session:**
1. Read the files listed above plus the feature plan in `.claude/docs/features/`.
2. Write design notes in your own words.
3. Write the feature plan to `.claude/docs/features/<NN>-<name>.md`.
4. **Stop and ask for approval.**
5. Implement on a feature branch in small commits.
6. Run build and tests, fix failures.
7. Update `.claude/docs/PROGRESS.md`.
8. Final short report: what was built, files changed, decisions, deviations, risks.

**Safety:**
- Never commit or push unless the user says so.
- Ask before running any build, test, or inspection command.
- Never touch production systems.
- Do not delete or rewrite existing user work without asking.
- Stay in scope and flag other findings separately.

**Communication:** short, simple, direct. All open questions in one batch with a
recommended default. State assumptions, flag risks early, never hide a deviation.

## 5. Done Checklist (Every Feature)

- [ ] Follows global and project rules, `SECURITY.md`, and `.claude/docs/security-agent-guidelines.md`
- [ ] Pragmatic simplicity verified (clean, readable, no over-engineering or speculative complexity)
- [ ] Performance standards respected (`const` constructors, non-blocking UI, SWR caching)
- [ ] Design notes written
- [ ] All 4 states handled (loading, empty, error, offline)
- [ ] No secrets or personal data in code or logs
- [ ] Local storage classified correctly (SecureStorage vs SQLite vs SharedPreferences)
- [ ] All database queries parameterized
- [ ] Build passes (`flutter analyze`, `flutter test`)
- [ ] `.claude/docs/PROGRESS.md` updated
- [ ] `.claude/docs/project-structure.md` updated if structure changed

## 6. Docs to Keep Current

If a change makes anything in `.claude/docs/`, `CLAUDE.md`, a README or this file
inaccurate, update it in place in the same task (no versioned copies).
`.claude/docs/PROGRESS.md` is updated at the end of every session.
