# Project: Safe Travel Sri Lanka

This file is the project-level index. It follows the global configuration at
`~/.claude/CLAUDE.md` (rules, security, and skills that apply everywhere) and adds
this project's specific context on top.

**Read this file first in any new session, then load the referenced docs below as
relevant to the task at hand — not all of them, just what's needed.**

## What this is

**Safe Travel Sri Lanka** — a Flutter/Dart mobile application developed in affiliation
with the Sri Lanka Red Cross Society. Provides real-time hazard warnings, offline-first
safety guides, emergency SOS dispatch, trip planning, and trusted local business
recommendations for international tourists and local travelers in Sri Lanka.

**Offline-first architecture**: critical safety content (first-aid guides, emergency
numbers, safety topics) is bundled locally and works without connectivity. Online
features include live hazard alerts, interactive safety maps, weather data, and
explore/business directory.

The product principle: **useful under stress, simple under calm.** The app must work
when connectivity fails, when the user is panicking, when they don't speak Sinhala.
No feature complexity at the cost of reliability or accessibility.

## Project Docs

All project docs live in `.claude/docs/` (not a root-level `docs/`):

- `.claude/docs/project-brief.md` — what this project is, its nature, and its actual goals
- `.claude/docs/tech-stack.md` — languages, frameworks, key dependencies, infra
- `.claude/docs/architecture.md` — current architecture and key structural decisions
- `.claude/docs/project-structure.md` — file/folder layout, kept current as the project
  changes
- `.claude/docs/coding-patterns.md` — Dart/Flutter coding conventions for this project
  (pins the global `coding-standards` skill to this stack)
- `.claude/docs/security-agent-guidelines.md` — mandatory security rules, storage matrix, dialer checks, and pre-commit security gates for agents
- `SECURITY.md` — project-level security policy, threat model, network defense, and vulnerability reporting
- `.claude/docs/PROGRESS.md` — session-by-session status and decision log (read first
  when resuming)

## Stack-Based Skill Selection

Based on `.claude/docs/tech-stack.md`, apply the appropriate stack-specific conventions from
`.claude/docs/coding-patterns.md` alongside the global skills in `~/.claude/skills/`. All global
skills (brainstorming, architecture, planning, dev-loop, documentation, ui-ux,
coding-standards) apply regardless of stack — `docs/coding-patterns.md` is what adapts
`coding-standards` to this project's actual language/framework.

## Working Process

- **Pragmatic simplicity (Anti-Overengineering)**: Always write clean, direct, readable Dart.
  Never add speculative complexity, redundant wrapper classes, or generic abstractions. Focus
  strictly on what is actually needed now to preserve performance, resources, and clarity.
- Small reviewable chunks: implement, self-review (security + performance included),
  confirm it meets the actual requirement, present. Then wait for the user; they commit.
- Stay strictly in scope. Spotted something else worth fixing? Flag it separately, don't
  fold it in.
- Genuinely debate technical decisions rather than agreeing by default; once a clear
  instruction is given, execute it.
- **Keep docs current in the same task.** If a change makes anything in `.claude/docs/`,
  `.claude/AGENTS.md`, `README.md` or this file inaccurate, update it in place (no
  versioned copies). New standards go into the relevant doc, and if a standard changes,
  every doc that states the old one changes with it.

## Sandbox and Security Rules

- This project directory is the boundary for all work in this session.
- Never commit or push unless the user says so for that change.
- Ask before running any build, test, or inspection command.
- Never touch production systems or real databases.
- Do not delete or rewrite existing user work without asking.
- No secrets, keys, `.env` files or real data in code, config, tests or CI. Secrets come
  from the environment or `--dart-define`.
- Feature branch per feature. Never commit to `main` directly.
- Never copy code from reference projects verbatim. Fresh implementation only.
- Stay in scope and flag other findings separately.

## Doc Map (`.claude/docs/`)

| Doc | Read when |
|---|---|
| `project-brief.md` | Understanding what the app does |
| `tech-stack.md` | Adding a dependency or changing a version |
| `architecture.md` | System shape, modules, offline/online split, state management |
| `project-structure.md` | Deciding where a file goes |
| `coding-patterns.md` | Writing or reviewing any Dart/Flutter code |
| `security-agent-guidelines.md` | Every feature session: storage choices, logging rules, dialer checks, SQL queries |
| `plans/ui-ux-design-specification.md` | Implementing or styling ANY UI screen, widget, theme token, skeleton, or micro-interaction |
| `PROGRESS.md` | Resuming work, checking what's done |
| `features/` | Planning or picking the next task |
| `plans/` | Architectural decisions and design documents |
| `archive/` | Superseded docs (reference only) |

## Notes

- This project directory is the boundary for all work in this session.
- Keep `.claude/docs/project-structure.md` and other structural docs current as the
  project changes.
