# Flutter Craft

**A Codex skill for building maintainable Flutter apps with GetX, consistent architecture, efficient scrolling, and explicit memory management.**

Flutter Craft turns detailed engineering conventions into a reusable workflow for Flutter and Dart development. Build new Android and iOS applications, extend existing features, or review architecture and performance with a single skill.

## What Flutter Craft covers

| Area | Engineering guidance |
|---|---|
| Flutter architecture | Feature isolation, shared services, typed data models, datasources, and clear folder ownership. |
| GetX state management | Page controllers, Bindings, StateMixin, scoped Obx updates, and predictable resource lifecycles. |
| UI structure | One class per file, a 200-line UI limit, meaningful composition, and strict separation of UI and logic. |
| Design systems | Central design tokens, component themes, light and dark modes, and platform-specific presentation. |
| Scrolling performance | Lazy ListView, GridView, and sliver builders; accurate item extents; bounded viewports and retained state. |
| Long-form reading | Lazy article blocks that preserve content order, typography, line spacing, and paragraph layout. |
| Navigation and memory | PageView-based navigation-bar destinations with explicit cleanup when pages are left. |
| Search and computation | Debounced requests, stale-response protection, suitable isolate work, and bounded caches. |
| Application quality | Authentication, permissions, typed failures, offline behavior, accessibility, and localization. |
| Android and iOS delivery | Compatible stable SDKs, reproducible toolchains, behavioral testing, and release verification. |

## Install

Clone the skill into your Codex skills directory:

```sh
git clone https://github.com/mohammedjaf96/flutter-craft.git ~/.codex/skills/flutter-craft
```

If you use a custom `CODEX_HOME`, install it under that directory's `skills` folder. Keep the entire skill folder so the linked references remain available.

## Use

Invoke `$flutter-craft` and describe the implementation or review you need:

```text
Use $flutter-craft to build a product catalog with search, filters,
and paginated loading using this project's existing design system.
```

```text
Use $flutter-craft to review this Flutter app's GetX architecture,
controller lifecycles, list performance, and memory retention.
```

```text
Use $flutter-craft to optimize a very long article without changing
its content order, line spacing, paragraph spacing, or typography.
```

## How the skill works

`SKILL.md` identifies the relevant workflow and routes the assistant to focused references. The references preserve all 38 agreed sections while keeping conditional guidance separate from the entry point.

| File | Purpose |
|---|---|
| `SKILL.md` | Skill discovery, workflow, key constraints, and reference routing. |
| `agents/openai.yaml` | Display name, short description, and invocation example. |
| `references/architecture.md` | Architecture and code conventions, sections 1-25. |
| `references/project-foundations.md` | Project discovery and missing infrastructure, section 26. |
| `references/quality-and-delivery.md` | Review checklist, linting, tests, security, accessibility, and release delivery, sections 27-33. |
| `references/performance.md` | Lists, search, computation, articles, memory, and navigation, sections 34-36 and 38. |
| `references/platform-versions.md` | Stable Android/iOS SDK and build-tool selection, section 37. |
| `scripts/check_english.py` | Repository language check for file names and text content. |

## Conventions with explicit boundaries

- This is an opinionated Flutter/GetX workflow, not a claim that every Flutter project must use the same architecture.
- Controllers, bindings, themes, and helper APIs must be verified in the actual project before use.
- Newest SDK numbers are not frozen into the skill. Check official platform requirements and plugin compatibility when choosing toolchains.
- A newer build SDK does not automatically require dropping older supported operating systems.
- Leaving a navigation destination releases page-owned resources, not durable user data or application-wide services.
- Examples are illustrative. This repository does not ship a Flutter application, backend, signing credentials, or installed Dart dependencies.

## Repository language

All repository documentation, examples, comments, file and folder names, commit messages, and repository metadata must be in English. Application localization requirements described in the skill remain separate from the repository's authoring language. Keep repository examples in English.

Run the repository language check before committing:

```sh
python3 scripts/check_english.py
```

The check rejects non-ASCII letters in names and text, including Arabic script. Standard punctuation and diagram symbols are allowed. It runs in GitHub Actions for pushes and pull requests.

## Discoverability

Focus areas: Flutter, Dart, GetX, Codex skills, AI coding agents, mobile app architecture, state management, design tokens, lazy loading, scrolling performance, memory management, Android, and iOS.
