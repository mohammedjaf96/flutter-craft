# Flutter Craft — Project Foundations

Section numbers preserve cross-references across this skill.

- 26. Project Discovery and Foundation Setup

## 26. Project Discovery and Foundation Setup

### 26.1 Existing project

- Inspect project instructions, `pubspec.yaml`, SDK constraints, folders, routes, bindings, datasources, themes, localization, and tests before coding.
- Locate and verify every referenced class and method; reuse actual equivalents instead of assuming the names in examples exist.
- Make the smallest foundation change needed for the requested feature. Do not rebuild an existing project or add a parallel architecture.
- Keep one controller per main page; never split it. All page state, variables, functions, getters, setters, and logic remain in that controller (§7, §9).

### 26.2 New project or missing foundation

Create only the infrastructure needed by the requested scope, before the feature depends on it:

| Foundation | Required instruction |
|---|---|
| GetX and bindings | Add a compatible dependency, configure app routing, and register services/controllers through application or feature bindings. Never use `Get.put()`. |
| Routes | Create `AppRoutes`, route configuration, and typed `WidgetsRouteClick` functions; register the page and its binding together. |
| Request paths | Implement `core/api/apiPath.dart` with `fill` and `segment`; encode each raw value once, reject dot segments, and reject missing/unresolved placeholders. Add behavioral tests for special characters and invalid templates. |
| Network | Create the shared HTTP client, environment configuration, timeouts, authentication handling where needed, and typed failure mapping. Feature datasources own data access. |
| Storage | Create the needed shared storage service; keep sensitive session material in platform-appropriate protected storage. Controllers access persisted data through datasources/services. |
| Models | Add explicitly typed models and validate nullable, absent, and malformed response fields at the data boundary. Do not pass raw dynamic payloads through the app. |
| Theme | Follow the existing-system or new-system path in §18. |
| Localization | Create `Localization`, `LocalizationKeys`, Arabic and English maps, supported-locale persistence, and key-parity checks. |
| Status views | Create loading, empty, and error views for the actual page states in `shared/pagesState/`; add skeletons where useful. |

- Example class names are specifications for new helpers, not claims that code already exists.
- Verify package and SDK APIs against the actual dependency versions before implementation; do not guess APIs.
- Report created foundations, important assumptions, verification performed, and any remaining external configuration.

---

