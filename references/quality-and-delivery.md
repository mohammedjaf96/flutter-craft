# Flutter Craft — Quality And Delivery

Section numbers preserve cross-references across this skill.

- 27. Summary Checklist for AI Assistants
- 28. analysis_options.yaml — Lint Enforcement
- 29. Testing and Verification
- 30. Security, Authentication, and Permissions
- 31. Failures, Connectivity, and Data Consistency
- 32. RTL, Accessibility, and Responsive Layout
- 33. Environments, Builds, and Release Delivery

## 27. Summary Checklist for AI Assistants

Before generating or editing any code in this project, confirm:

- [ ] Feature is inside `lib/features/<featureName>/` with `bindings/`, `controllers/`, `data/model/`, `data/datasources/`, `views/pages/`, `views/widgets/`
- [ ] Service used by more than one page/controller is in `core/`: one file → `core/services/`, more than one file → `core/<serviceName>/`
- [ ] No underscores or hyphens in any name — including no `_`-prefixed private/local members or builders
- [ ] No abbreviated / shortcut names — full words only
- [ ] One class per file — never two classes in one file
- [ ] No line break before or after `.` — in widget trees break only after `(`, `)`, `),`, `;` or an argument's `,`
- [ ] Every UI file is 1–200 lines; anything longer is split into **header / body / footer**, feature-named
- [ ] Meaningful extraction only: page composition is allowed; wrappers that add no responsibility are avoided (§4.2)
- [ ] Page-tree splits go in `views/pages/`; cards / spec widgets / spec elements go in `views/widgets/`
- [ ] A custom widget is its own file in `views/widgets/` — never inside the page, never a local `_` builder (small/simple pieces stay inline)
- [ ] Every widget class (page or widget) holds only: constructor + its `final` fields + `@override build()` — **no var, no function, no get, no set**, nothing else
- [ ] Every page has a controller (two related pages may share one) — all vars, functions, getters, setters live there
- [ ] `StatelessWidget` / `GetView<T>` by default; `StatefulWidget` **only at extreme need** (real element-lifecycle requirement)
- [ ] No `initState` / `dispose` in a widget — use `onInit` / `onClose` in the controller (except inside a §8.1-justified `StatefulWidget`, for lifecycle only)
- [ ] One controller per main page, named after the page; no controllers for widgets; split the page, never the controller
- [ ] `Rx` only where the UI must react or animate; **`Rxn` for nullable reactive values**; explicitly typed normal values for everything else
- [ ] Full-page status uses `StateMixin` + `controller.obx()` — never `Obx()` + if/else for page status
- [ ] In `controller.obx()` success comes first; small trees stay inline and meaningful extraction follows §4.2 and §11.1
- [ ] `Obx` wraps the **deepest changing leaf** (the `Text`), not the parent `Column`/`Container`/`Row`
- [ ] Every comment is a **single short line** — never big or long — written for the AI, only where the reason isn't obvious
- [ ] All user-facing text via `LocalizationKeys.<key>.tr` — translations are Dart `const` maps in `lib/core/localization/` (Arabic default and English supported); **no JSON, no ARB, no gen-l10n, no string-literal keys, no hardcoded text**
- [ ] Every color, text style, size, radius, shadow comes from the theme (`context.colors` …) — nothing hardcoded in UI; extend an existing theme or create the central system first (§18)
- [ ] iOS: app bar, nav bar and app-bar buttons use Liquid Glass (`glass`); Android: never, our own UI
- [ ] Every screen works in light **and** dark mode
- [ ] Skeleton / loading / empty / error views are in `shared/pagesState/<pageName>/`
- [ ] Bottom sheets: opened by a controller function; whole sheet in one file `<pageName>BottomSheet.dart` (or `<pageName><Purpose>BottomSheet.dart`)
- [ ] Old StatefulWidgets converted: data status → `StateMixin` + `controller.obx()`, one changing widget → `Rx` + `Obx()`
- [ ] No UI code in `core/`; multi-page widgets → `shared/widgets/`, one-page widgets → `features/<featureName>/views/widgets/`
- [ ] `?` / `!` / `??` only on really nullable values; `try/catch` only where code can really throw
- [ ] Simple code — no abstraction for a single use; no invented files, classes or APIs
- [ ] Only the requested code changed
- [ ] Clear, short names (two to four words) — no `data`, `item`, `result`
- [ ] No `print` / `debugPrint`, no commented-out code, no `TODO` (tasks are in ClickUp)
- [ ] `const` everywhere it can be; no unused imports
- [ ] Every variable, field, parameter and return has an explicit type — no `dynamic`
- [ ] No `Get.put()`; `Get.find()` only where the route flow guarantees registration — no `Get.isRegistered()` checks
- [ ] Navigation only through `WidgetsRouteClick` — no direct `Get.toNamed` / `Get.to`
- [ ] Controllers call datasources, never the API or storage directly
- [ ] Everything opened in a controller is disposed / cancelled in `onClose()`
- [ ] Page API data loads in `onReady()`; independent requests run together with `Future.wait`
- [ ] Every value in a request path goes through `ApiPath.fill` / `ApiPath.segment` — no `replaceFirst`/`replaceAll`, no raw `$id`, no pre-encoding
- [ ] New pages have route constants, page/binding registration, validated arguments, and typed `WidgetsRouteClick` functions
- [ ] Project foundations verified or implemented before use (§26)
- [ ] Relevant analysis, behavioral tests, and requested target builds completed or limitations reported (§29)
- [ ] Authentication, permission, failure, and offline behavior match feature scope (§30–31)
- [ ] RTL/LTR, accessibility, text scaling, and responsive layout verified (§32)
- [ ] Environment configuration and release delivery requirements checked (§33)
- [ ] Collection lists/grids use lazy builders with bounded viewports; accurate extents are supplied when available (§34)
- [ ] Search suppresses stale results; compute, rendering, cache limits, repaint boundaries, and keep-alive follow actual workload needs (§35)
- [ ] Very long articles use lazy blocks without changing content order, line/paragraph spacing, or original layout; parsing and scroll updates avoid repeated work (§36)
- [ ] Android/iOS SDKs and build tools form a verified stable compatible set, meet current store requirements, and preserve intended minimum-OS coverage (§37)
- [ ] Navigation-bar destinations use PageView.builder; departed page content and destination-owned controllers/resources are released after transition settlement (§38)

---

## 28. analysis_options.yaml — Lint Enforcement

Configure the rules below in `analysis_options.yaml`; verify the actual SDK supports them and run `flutter analyze`. These built-in lints cover only part of this document; naming restrictions, file size, architecture, and other structural rules require explicit review or dedicated checks.

| Lint Rule | Enforces |
|---|---|
| `prefer_const_constructors` | Use `const` wherever possible |
| `avoid_unnecessary_containers` | No pointless Container wrappers |
| `use_key_in_widget_constructors` | Always pass `key` to widgets |
| `prefer_final_fields` | Mark fields `final` when never reassigned |
| `avoid_print` | Use a logging service, not `print()` |
| `unnecessary_this` | Don't write `this.` when not needed |
| `camel_case_types` | Class names must be `UpperCamelCase` |
| `non_constant_identifier_names` | Variables and functions must be `lowerCamelCase` |

> **`file_names` must stay DISABLED (`file_names: false`).**
> That lint enforces Dart's default `lower_case_with_underscores.dart`, which directly contradicts
> §2 of this document (`lowerCamelCase.dart`). Leaving it on makes the analyzer flag every correctly
> named file in the project and drowns every real warning in the noise.


---

## 29. Testing and Verification

- Add or update behavioral tests for changed business logic, validation, controller state transitions, and datasource response/failure mapping.
- Test relevant loading, empty, error, success, retry, and invalid-input cases, including stale responses or controller closure where asynchronous work is involved.
- Test `ApiPath` encoding, route argument handling, localization key parity, and session behavior when those foundations are introduced or changed.
- Use widget tests for meaningful user-visible behavior; check affected screens in Arabic/English, RTL/LTR, and light/dark modes.
- For a complete application, exercise its actual critical flows with integration tests or repeatable device checks: login/session restoration, primary task, logout, and payment/order creation only if included in scope.
- Mock external side effects; tests must not create real charges or production records.
- Run `flutter analyze`, relevant tests, and a build for each requested delivery target. Report commands, results, and anything that could not be verified.
- Do not claim unrun checks passed. Fix failures caused by the change; disclose unrelated existing failures.

## 30. Security, Authentication, and Permissions

- Do not commit secrets, signing credentials, private keys, or production access tokens. Client configuration is not a safe place for server secrets.
- Use encrypted transport for production requests and do not disable certificate validation.
- Validate external input and response data at boundaries. Treat client route guards as UX controls; protected operations must also be authorized by the backend.
- When authentication is in scope, model signed-out, signed-in, expired, and refreshing states explicitly. Coordinate refresh so concurrent failures do not start multiple refresh requests; bound retries and avoid loops.
- Store sensitive session material using an appropriate protected-storage implementation. Clear session material and user-specific caches on logout, and invalidate sessions server-side where supported.
- Hide credentials and personal data from errors, telemetry, and diagnostics. Any production observability must use a deliberate sanitized service, never leftover debug statements.
- Request platform permissions only when a relevant action needs them. Handle denied, permanently denied, and restricted states with localized explanations and recovery actions.
- Confirm SDK, plugin, and platform permission configuration for the requested targets; do not add permissions or trackers unrelated to scope.

## 31. Failures, Connectivity, and Data Consistency

- Translate network, timeout, validation, authentication, and server failures into typed failures with localized actionable messages. Do not show raw exceptions or backend internals to users.
- Distinguish an empty successful response from a failure. Provide retry for recoverable failures without discarding usable content unnecessarily.
- Treat the request result as the authority; a connectivity signal alone does not prove the server is reachable.
- Retry only transient failures with bounded attempts and delay. Do not automatically repeat a mutation such as payment or order creation without a defined idempotency mechanism.
- Define offline behavior for each data-dependent feature: cached read-only data, explicit unavailability, or a scoped offline queue. Do not silently promise offline writes or sync.
- If caching or queued writes are required, define ownership, expiry, user isolation, pending status, synchronization, and conflict handling. Clear private cached data on logout.
- Prevent duplicate submissions and ignore or cancel stale responses after new searches, navigation, or controller closure. Controllers own lifecycle cleanup.

## 32. RTL, Accessibility, and Responsive Layout

- Support Arabic RTL and English LTR. Prefer directional padding, alignment, and positioning; mirror directional icons appropriately while preserving inherently fixed-direction content.
- Localize dates, numbers, currencies, and plural forms; do not build translated sentences by concatenation.
- Respect text scaling and available space; avoid clipping, overflow, and fixed text heights. Verify narrow and wide layouts relevant to the requested targets.
- Give interactive controls meaningful semantic labels, visible focus, sufficient contrast, and usable touch targets through the design system.
- Support keyboard navigation and screen readers where the target platform exposes them. Do not communicate status through color alone.
- Respect reduced-motion preferences for nonessential animations. Verify affected screens with larger text and assistive-technology checks appropriate to the change.

## 33. Environments, Builds, and Release Delivery

- Separate development, staging, and production configuration where the project needs them; centralize endpoints and feature flags without embedding secrets.
- Document required environment keys, setup commands, supported SDK/dependency versions, and target-specific configuration in the project's established documentation.
- Keep dependency resolution reproducible and use the actual selected versions when checking APIs. Do not upgrade unrelated dependencies as part of a feature.
- Automate relevant analysis, tests, and target builds in the existing CI system; add a minimal pipeline for a new complete application when repository tooling supports it.
- Before release delivery, verify version/build numbers, required assets, permission descriptions, signing configuration, and the release build on the requested targets.
- Keep signing material outside the repository. Never claim a release is signed, uploaded, or published unless the corresponding action actually succeeded.
- Prepare a concise delivery report with scope, setup, checks, known limitations, and required external credentials or services. Publish or deploy only within the user's authorized scope.


---

