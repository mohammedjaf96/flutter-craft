---
name: flutter-craft
description: Build, extend, or review Flutter applications using GetX, feature isolation, strict UI/controller separation, central design tokens, lazy scrolling, and explicit resource lifecycles. Use for Flutter app implementation and architecture work, especially when the user requests Flutter Craft.
---

# Flutter Craft

Apply the user's agreed Flutter/GetX conventions to the requested scope. This skill supports new applications and focused changes to existing ones; it does not authorize unrelated rewrites, account changes, or publishing.

## Read the applicable rules

- Always read [Architecture and code standards](references/architecture.md) before generating or editing Dart code. It defines feature folders, naming, one class per file, the 200-line UI limit, widget purity, GetX controllers, status rendering, localization, themes, bindings, navigation, datasources, and cleanup.
- For initial project discovery, new apps, or missing helpers, read [Project foundations](references/project-foundations.md). Verify actual files/APIs before using them; implement only missing foundations needed by the task.
- Before implementing security, session, error/offline, accessibility, environment, or release behavior, and before delivering any implementation, read the relevant sections of [Quality and delivery](references/quality-and-delivery.md). Section 27 is the final checklist and section 28 covers lints.
- For lists/grids, search, CPU-intensive work, memory, long articles, or navigation-bar pages, read the relevant sections of [Performance](references/performance.md): §34 lists/extents, §35 search/compute/rendering/memory, §36 long articles, §38 navigation page lifecycles.
- At project setup, planned toolchain changes, and release preparation, read [Platform versions](references/platform-versions.md). Check current official requirements and select a verified stable compatible toolchain; this skill deliberately contains no permanent latest-version numbers.

Section references such as §18 or §38 point to the numbered headings in these files. The architecture reference holds §1–25, foundations §26, quality §27–33, performance §34–36 and §38, platform versions §37.

## Preserve these decisions

- One controller per main page; closely related pages may share one. Never split a page controller or create controllers for individual widgets. Keep page state, variables, functions, getters, setters, and logic in the controller; data access remains in datasources.
- Widget classes contain only the constructor, constructor-backed final fields, and `build`. Follow the narrowly defined element-lifecycle exception for a genuinely necessary StatefulWidget.
- Each UI file stays at or below 200 lines. Meaningful page composition is allowed; success bodies may be extracted when warranted. Do not add responsibility-free forwarding wrappers.
- Centralize design values; extend an existing system or establish tokens and light/dark themes before screens in a new app. Preserve the required iOS/Android visual conventions.
- Use lazy builders for collection scrolling. Choose item extents only when accurate for the content and constraints; never force uniform paragraph heights.
- Long-article optimizations preserve text order, inline content, line/paragraph spacing, wrapping, shaping, and intended layout. Keep an enormous paragraph intact if splitting cannot preserve its layout.
- Navigation-bar destinations use `PageView.builder`. Release departed page content and page-owned controllers/resources after the transition settles, and rebuild on re-entry. PageView alone does not guarantee GetX cleanup. Do not delete durable user data or application-wide services when leaving a page.
- Do not reintroduce removed deployment-specific browser optimizations or deferred-loading requirements.
- Naming rules in the architecture reference govern app-owned Dart identifiers and source folders/files. This skill package uses the required `SKILL.md`, `agents/openai.yaml`, and reference filenames. Do not rename tool-required project files or existing external package APIs to fit Dart naming conventions.

## Repository authoring language

Keep this skill repository's documentation, examples, code comments, names, metadata, and commit messages in English. Do not introduce non-English sample strings into the repository. Application localization requirements in the references are separate from this repository authoring rule. Run `python3 scripts/check_english.py` before committing changes to the skill.

## Implementation and delivery

Inspect project instructions and actual dependency versions, choose the relevant references, and make the requested change. Keep any missing-design question narrow and continue independent work. Do not invent files, APIs, backend contracts, credentials, or test results.

Review the applicable checklist, run meaningful analysis/tests/builds for the requested targets, and report the changed behavior, checks actually run, assumptions, and concrete remaining setup. Do not publish or deploy beyond the user's authorization.
