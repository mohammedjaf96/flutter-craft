# Flutter Craft — Architecture

Section numbers preserve cross-references across this skill.

- 1. Folder Structure
- 2. Naming Conventions — ZERO TOLERANCE
- 3. One Class Per File — ABSOLUTE RULE
- 4. File Size — Every UI File Is 1–200 Lines
- 5. Splitting a Large UI Tree — pages/ vs widgets/
- 6. Custom Widgets — Cards, Spec Widgets, Spec Elements
- 7. Widget Classes Are Pure — Constructor, Fields, `build()` — ABSOLUTE RULE
- 8. Widget Rules — StatelessWidget First
- 9. Controllers — One Per Main Page
- 10. Reactive Variables — Rx, Rxn, and Normal
- 11. Page / Endpoint Status — StateMixin + `controller.obx()`
- 12. Obx Usage Rules — Wrap the Leaf, Not the Tree
- 13. Comments
- 14. Localization — GetX Translations with Dart const maps
- 15. Request Paths — Every Value Goes Through `ApiPath` — ABSOLUTE RULE
- 16. New Pages — Routes and Bindings
- 17. Code Formatting — Never Break a Call Chain
- 18. Design System — Theme Only — ABSOLUTE RULE
- 19. Liquid Glass — iOS Only
- 20. Light and Dark Mode — Always Both
- 21. Code Hygiene
- 22. Dependency Injection — Bindings Only
- 23. Navigation — Through `WidgetsRouteClick` Only
- 24. Data Access — Always a Datasource
- 25. Performance — Always Dispose and Close

## 1. Folder Structure

```
lib/
├── core/                          # Shared services, utilities, algorithms
│   ├── network/                   # HTTP client, interceptors, base API
│   ├── storage/                   # Local storage service
│   ├── theme/                     # App theme, colors, text styles
│   ├── routing/                   # GetX routes & bindings
│   ├── utils/                     # Formatters, validators, helpers
│   ├── services/                  # Small shared services — one file each
│   └── <serviceName>/             # Big shared service (more than one file) — its own folder
│                                  # NO widgets / UI code in core/
│
├── shared/
│   ├── widgets/                   # Widgets used by more than one page/feature
│   └── pagesState/                # Status pages: skeleton / loading / empty / error
│       └── <pageName>/            # One folder per page — its state files
│
├── features/                      # One folder per feature — strictly isolated
│   └── <featureName>/
│       ├── bindings/              # GetX bindings — dependency registration
│       ├── controllers/           # GetxController only — all logic & state live here
│       ├── data/
│       │   ├── datasources/       # API calls, local DB, remote sources
│       │   └── model/             # Data models / entities
│       └── views/
│           ├── pages/             # Full screens + page-tree splits  (see §4, §5)
│           └── widgets/           # Widgets used only by this feature's page(s)  (see §6)
│
└── main.dart
```

### Folder rules

- Every feature **must** live inside `lib/features/<featureName>/`.
- Features must **never** import directly from another feature's `controllers/` or `views/`.
  Share data through `core/` services or GetX dependency injection only.
- `core/` is for code that is written **once** and reused many times — **never** UI code or widgets.
- A widget used by **more than one page or feature** → `lib/shared/widgets/`.
- A widget used by **one page / one class only** → `lib/features/<featureName>/views/widgets/`.

### 1.2 Status pages — `shared/pagesState/`

The **skeleton**, **loading**, **empty**, and **error** views of a page are its status pages.
They **always** go in `lib/shared/pagesState/<pageName>/` — one folder per page, one file per state.
Status views used by the whole app (one generic loader, one generic error) go in `lib/shared/pagesState/common/`.

```
lib/shared/pagesState/
├── productDetails/
│   ├── productDetailsSkeleton.dart
│   ├── productDetailsLoading.dart
│   ├── productDetailsEmpty.dart
│   └── productDetailsError.dart
└── common/
    ├── appLoader.dart
    └── appErrorState.dart
```

- **Never** put a skeleton / loading / empty / error view in `features/<featureName>/views/` or `shared/widgets/`.
- They are used in `controller.obx()` as `onLoading`, `onEmpty`, `onError` (§11).
- `views/pages/` = the screen itself and any file it is split into (§5).
  `views/widgets/` = cards, spec widgets, and reusable UI pieces (§6).

### 1.1 Shared services — where they go

A service used by **more than one page or more than one controller** is a shared service and
**must** live in `core/` — never inside a feature. Where in `core/` depends on its size:

| Service size | Where it goes | Example |
|---|---|---|
| **Small** — the whole service is **one file** | `core/services/<serviceName>.dart` | `core/services/clipboardService.dart` |
| **Big** — the service has **more than one file** | `core/<serviceName>/` — its own folder named after the service | `core/notification/notificationService.dart`, `core/notification/notificationModel.dart` |

- A small service that grows into more than one file **moves** out of `core/services/` into its own
  `core/<serviceName>/` folder.
- A service used by **only one** page/controller stays inside that feature — it is not shared.

```
// ✅ CORRECT
lib/core/services/clipboardService.dart          // small, one file
lib/core/notification/notificationService.dart   // big, own folder
lib/core/notification/notificationModel.dart

// ❌ WRONG
lib/features/cart/data/datasources/clipboardService.dart   // used by many pages → belongs in core
lib/core/services/notification/notificationService.dart    // big service → core/notification/, not under services/
```

---

## 2. Naming Conventions — ZERO TOLERANCE

| Element | Rule | ✅ Correct | ❌ Wrong |
|---|---|---|---|
| Classes | `UpperCamelCase` | `HomeController` | `home_controller`, `homeController` |
| Files | `lowerCamelCase.dart` | `homeController.dart` | `home_controller.dart` |
| Variables | `lowerCamelCase` | `userName` | `user_name`, `UserName` |
| Functions | `lowerCamelCase` | `fetchUser()` | `fetch_user()`, `FetchUser()` |
| Folders | `lowerCamelCase` | `userProfile/` | `user_profile/`, `UserProfile/` |
| Constants | `lowerCamelCase` | `apiBaseUrl` | `API_BASE_URL`, `api_base_url` |

> **Rule:** No underscores (`_`) and no hyphens (`-`) anywhere in class names, variable names,
> function names, folder names, or file names. Use `UpperCamelCase` for classes,
> `lowerCamelCase` for everything else.

### 2.1 Never use the underscore `_` — for anything

- The underscore prefix is **forbidden** for **every** identifier: variables, functions, classes, widgets, methods, and fields.
- This includes "private" members and local helper builders. **No** `_buildHeader()`, **no** `_MyCard`, **no** `_isLoading`.
- If something needs to be reused → give it a real name and its own file. If it does not → inline it (§6).

```dart
// ❌ WRONG — underscore-prefixed helper widget / method
Widget _buildProductCard() { ... }
class _ProductCard extends StatelessWidget { ... }

// ✅ CORRECT — real name, own file in views/widgets/  →  productCard.dart
class ProductCard extends StatelessWidget { ... }
```

### 2.2 No shortcut or abbreviated names

- Names must be **full words**, spelled out. No abbreviations, no shortcuts, anywhere.
- ❌ `usr`, `btn`, `img`, `ctrl`, `qty`, `catList`, `prodDtl`
  ✅ `user`, `button`, `image`, `controller`, `quantity`, `categoryList`, `productDetail`

---

## 3. One Class Per File — ABSOLUTE RULE

- **One class = one file. Always.** Never, never, never write two classes in one file.
- **Never** write two classes in one file — not a page + a widget, not a widget + a helper, not a controller + a model, nothing.
- No helper classes, no private classes, no enums bundled inside another class file.
- Each class gets its own dedicated `.dart` file.

```dart
// ✅ CORRECT — homeController.dart contains only HomeController
class HomeController extends GetxController { ... }

// ❌ WRONG — two classes in one file
class HomeController extends GetxController { ... }
class HomeState { ... }  // must be in its own file: homeState.dart
```

---

## 4. File Size — Every UI File Is 1–200 Lines

- **Any UI file (page or widget) must be between 1 and 200 lines.** Hard limit.
- If a file grows **beyond 200 lines, it must be split** into two (or more) files.
- Splits are **meaningful**, never random. Name each split after the feature + the part it holds — never a generic name like `part2.dart` or `widgets.dart`.

### 4.1 The standard split — header / body / footer

When a page crosses 200 lines, split it into **header**, **body**, and **footer** parts, each in its own
feature-named file. The page file then only composes them.

```
// ❌ WRONG — one 380-line file, or random split names
productDetailsPage.dart        // 380 lines
productDetailsPage2.dart

// ✅ CORRECT — header / body / footer, feature-named
productDetailsPage.dart        // < 200 lines, composes the parts
productDetailsHeader.dart
productDetailsBody.dart
productDetailsFooter.dart
```

> Where each split goes (page split vs widget extraction) is decided in §5.

### 4.2 Meaningful extraction and page composition

- Below 200 lines, keep the page together unless a reusable component warrants extraction under §6.
- Above 200 lines, extract meaningful layout sections; each UI file must remain within the limit.
- A page may compose its header, body, footer, and status views. Owning the screen layout is a real responsibility, even if the page mostly arranges other widgets.
- Do not create a wrapper that merely forwards unchanged arguments to one child and adds no layout, behavior, or reuse value.
- These rules also apply to the success builder in §11.1.

---

## 5. Splitting a Large UI Tree — pages/ vs widgets/

When a screen crosses 200 lines you split it. **Where the split file lives depends on what it is:**

- **Page-tree split → `views/pages/`.**
  If you are splitting out a **large piece of the page's own layout** (for example a big `Container` … `Column` subtree that is only part of this screen and is not a reusable component), the extracted file goes in **`views/pages/`**, next to the page. It is still part of the page — **not** a widget.

- **Reusable UI piece → `views/widgets/`.**
  If the piece is a self-contained, reusable component (a card, a spec widget, a spec element), it goes in **`views/widgets/`** (§6).

```
features/category/views/
├── pages/
│   ├── categoryPage.dart          # the screen — composes the parts below
│   ├── categoryPageHeader.dart    # page-tree split (part of THIS screen)
│   ├── categoryPageBody.dart      # page-tree split (part of THIS screen)
│   └── categoryPageFooter.dart    # page-tree split (part of THIS screen)
└── widgets/
    └── categoryCard.dart          # reusable component → widgets/
```

- Splitting a page **never** splits its controller. All split page files share the **one** controller for that page (§9).
- Every split file must own a meaningful layout section or reusable component; page composition is allowed (§4.2).

---

## 6. Custom Widgets — Cards, Spec Widgets, Spec Elements

- **Where:** used by more than one page/feature → `lib/shared/widgets/`; used by one page only →
  `lib/features/<featureName>/views/widgets/`. **Never** in `core/`.
- **Cards, spec widgets, and spec elements each get their own file in `views/widgets/`,** named for exactly what they are: `productCard.dart`, `categoryCard.dart`, `storeCard.dart`, `offerCardWidget.dart` …
- **Never** define a custom widget **inside** the page file.
- **Never** create a custom widget as a local `_`-prefixed method or class (§2.1). If you need a custom widget, **create a new file** with the widget's name in `views/widgets/` and put the widget there.
- **Exception — inline it, don't extract:** if the tree is **small / not big**, or the widget is **very simple / very small**, do **not** create a separate file. Just place it inline in the build tree.

```dart
// ❌ WRONG — custom widget defined inside the page as a local builder
class ProductPage extends GetView<ProductController> {
  Widget buildCard() { ... }          // ← forbidden: widget inside the page
}

// ✅ CORRECT — own file: views/widgets/productCard.dart
class ProductCard extends StatelessWidget {
  const ProductCard({super.key, required this.product});
  final Product product;

  @override
  Widget build(BuildContext context) { ... }
}

// ✅ ALSO CORRECT — simple/small piece stays inline (do NOT extract)
Padding(
  padding: EdgeInsets.all(context.sizes.spacingSmall),
  child: Text(controller.title),
)
```

### 6.1 Bottom sheets — open in the controller, content in its own file

- The **action that opens** a bottom sheet is a function in the page's **controller** — never inside the view.
- The sheet's **whole tree and UI** is written in **one file** named **page name + `BottomSheet`**:
  `productDetailsBottomSheet.dart` → class `ProductDetailsBottomSheet`, in `views/widgets/`.
- A page with **more than one** sheet adds the purpose between them: `productDetailsSizeBottomSheet.dart`,
  `productDetailsShareBottomSheet.dart`.
- A sheet used by more than one feature goes in `lib/shared/widgets/` (§1).
- The sheet follows every widget rule: pure class (§7), uses the page controller through `GetView`, max 200 lines (§4).

```dart
// ✅ CORRECT — controller opens it
void openSizeSheet() {
  Get.bottomSheet(const ProductDetailsSizeBottomSheet());
}

// view
IconButton(
  onPressed: controller.openSizeSheet,
  icon: const Icon(Icons.straighten),
)

// views/widgets/productDetailsSizeBottomSheet.dart — the whole sheet tree
class ProductDetailsSizeBottomSheet extends GetView<ProductDetailsController> {
  const ProductDetailsSizeBottomSheet({super.key});

  @override
  Widget build(BuildContext context) {
    return Container(...);
  }
}

// ❌ WRONG — opened inside the view, sheet tree written inline in the page
IconButton(
  onPressed: () => showModalBottomSheet(
    context: context,
    builder: (context) => Column(...),
  ),
  icon: const Icon(Icons.straighten),
)
```

---

## 7. Widget Classes Are Pure — Constructor, Fields, `build()` — ABSOLUTE RULE

This applies to **every** widget class — pages in `views/pages/` **and** widgets in `views/widgets/`.

**Never, never, never** write a variable, a function, a getter, or a setter inside a widget class.
A widget class holds **only** these three things, in this order:

1. The **constructor** — `super.key` plus the parameters it receives.
2. The **`final` fields** those constructor parameters fill.
3. **`@override`** and then directly **`Widget build(BuildContext context)`**.

Nothing else. No `var`, no non-constructor `final`/`const`/`static` field, no `get`, no `set`,
no method, no `void` function, no helper builder, no logic — not even a "small" one.

```dart
// ✅ CORRECT — constructor, its final fields, then @override build. Nothing else.
class ForceUpdatePage extends GetView<ForceUpdateController> {
  const ForceUpdatePage({
    super.key,
    this.installedVersion,
    this.storeVersion,
  });

  final String? installedVersion;
  final String? storeVersion;

  @override
  Widget build(BuildContext context) {
    return Scaffold(...);
  }
}

// ❌ WRONG — every member below is forbidden inside a widget class
class ForceUpdatePage extends StatelessWidget {
  var isLoading = false;                          // ← variable → controller
  static const double padding = 16;               // ← non-constructor field → controller
  String get title => 'Update';                   // ← getter → controller
  set title(String value) { ... }                 // ← setter → controller
  void openStore() { ... }                        // ← function → controller
  Widget buildHeader() { ... }                    // ← helper builder → views/widgets/ (§6)
}
```

Everything else has a home:

| Need | Where it goes |
|---|---|
| A variable / getter / setter / state | the page's **controller** (`controllers/`) |
| A function / any logic | the page's **controller** (`controllers/`) |
| A custom widget | **`views/widgets/`** (§6), or `views/pages/` for a page-tree split (§5) |

- **Every page has a controller** (§9) — that is where all of the above lives. A widget that needs
  a value or an action takes it through its constructor, or reads it from the page's controller via `GetView`.

```dart
// ✅ CORRECT — page file has ONLY the build tree
class ProfilePage extends GetView<ProfileController> {
  const ProfilePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          const ProfileHeader(),                 // widget → views/widgets/
          Obx(() => Text(controller.userName.value)),
          ProfileMenuList(onTap: controller.openItem),  // function → controller
        ],
      ),
    );
  }
}

// ❌ WRONG — helper widget / function declared outside build()
class ProfilePage extends GetView<ProfileController> {
  Widget buildHeader() { ... }           // ← forbidden
  void openItem() { ... }                // ← forbidden — belongs in controller
}
```

---

## 8. Widget Rules — StatelessWidget First

### 8.1 StatefulWidget is a last resort, not a ban

- The default is **always** `StatelessWidget`, or `GetView<Controller>` when the widget has a controller.
- All state lives in the `GetxController`.
- `StatefulWidget` is **not forbidden** — but it is allowed **only at extreme need**, when the widget
  genuinely cannot work without Flutter's own element lifecycle (for example a `TickerProvider` for an
  `AnimationController`, or a third-party widget that requires a `State` object).
- Before reaching for it, ask: *can a `GetxController` hold this state?* If yes — and it almost always
  is — use the controller. "It's easier" is not extreme need.

### 8.1.1 How a StatefulWidget is replaced

Every old `StatefulWidget` is converted — its state moves to the page's controller:

| Old StatefulWidget did | Replace with |
|---|---|
| Loads data and shows loading / empty / error / success | `StateMixin` in the controller + `controller.obx()` in the view (§11) |
| Changes one widget (a toggle, a counter, a selected tab, a visible flag) | an `Rx` value in the controller + `Obx()` on that widget only (§12) |
| `initState` / `dispose` | `onInit()` / `onClose()` in the controller (§9.3) |
| `AnimationController` with `TickerProvider` | the controller with `GetSingleTickerProviderStateMixin` / `GetTickerProviderStateMixin` |
| `TextEditingController`, `ScrollController`, `FocusNode` … | fields in the controller, disposed in `onClose()` (§25) |

```dart
// ❌ OLD — state inside the widget
class FavoriteButton extends StatefulWidget { ... }
class FavoriteButtonState extends State<FavoriteButton> {
  bool isFavorite = false;
  void toggle() => setState(() => isFavorite = !isFavorite);
}

// ✅ NEW — Rx in the controller, Obx on the one widget that changes
final RxBool isFavorite = false.obs;
void toggleFavorite() => isFavorite.value = !isFavorite.value;

Obx(() => Icon(controller.isFavorite.value ? Icons.favorite : Icons.favorite_border))
```

```dart
// ✅ DEFAULT — StatelessWidget / GetView, state in the controller
class HomeView extends GetView<HomeController> {
  const HomeView({super.key});

  @override
  Widget build(BuildContext context) {
    return Obx(() => Text(controller.title.value));
  }
}

// ✅ ALLOWED ONLY AT EXTREME NEED — e.g. requires a TickerProvider
class PulseAnimation extends StatefulWidget { ... }
```

### 8.2 A StatelessWidget with a controller becomes GetView

- If a `StatelessWidget` has a controller, **replace it with `GetView<ControllerName>`**.
- `GetView` gives you `controller` for free without calling `Get.find()` manually.

---

## 9. Controllers — One Per Main Page

### 9.1 One controller per main page, named after the page

- **Every page has a controller. Always.** There is no page without one.
- Every **main page** gets its own controller: `categoryPage`, `productDetails`, `subCategoryPage`, `profilePage` … each has exactly one controller.
- Two closely related pages **may share one controller** (for example a list page and its filter page) — but a page never has zero controllers.
- **All** variables, functions, getters, setters, and logic go in the controller — never in a widget class (§7).
- The controller is **named after the feature page**: page `categoryPage` → controller `CategoryController`; page `productDetails` → `ProductDetailsController`.
- **Do NOT create controllers for custom widgets** (cards / spec widgets / spec elements). Controllers exist for pages, not for widgets.

### 9.2 Split the page, never the controller

- When a long page is split into several page files (§4.1, §5), they **all share the single controller** for that page.
- Split only the **page**. **Never** split the controller.

### 9.3 Never use initState / dispose in a widget

| ❌ Forbidden | ✅ Use instead |
|---|---|
| `initState()` | `onInit()` in `GetxController` |
| `dispose()` | `onClose()` in `GetxController` |
| `setState()` | `.value =` on `Rx` variable or `update()` |

```dart
// ✅ CORRECT
class CategoryController extends GetxController {
  @override
  void onReady() {
    super.onReady();
    fetchData();
  }

  @override
  void onClose() {
    scrollController.dispose();
    super.onClose();
  }
}
```

> The one exception is a `StatefulWidget` justified under §8.1 — a widget that exists *because* it
> needs the element lifecycle naturally uses `initState` / `dispose` for that lifecycle only.
> Business logic still belongs in the controller.

### 9.4 No logic or functions in the View

- **Zero** functions in the view file.
- **Zero** variables, getters, or setters declared in the view file.
- Every function, every variable, every getter/setter, every piece of logic → goes in the controller (§7).

### 9.5 Fetch API data in `onReady()`, not `onInit()`

- API calls that fill a page start in **`onReady()`** — it runs after the page's first frame, so the
  page (and its skeleton) appears **immediately** and the transition is never blocked by the request.
- `onInit()` is only for cheap setup: reading arguments, creating controllers, registering workers.

```dart
// ✅ CORRECT
@override
void onReady() {
  super.onReady();
  fetchProductDetails();
}

// ❌ WRONG — the request starts before the page has painted its first frame
@override
void onInit() {
  super.onInit();
  fetchProductDetails();
}
```

### 9.6 Independent requests run together — `Future.wait`

- When a page needs **several** requests and **none depends on another**, start them **all at once**
  with `Future.wait` — the page waits for the slowest one, not for the sum of all.
- Run them one after another **only** when a request needs the result of an earlier one.

```dart
// ✅ CORRECT — three independent requests in parallel
Future<void> fetchHomeData() async {
  await Future.wait([
    fetchBanners(),
    fetchCategories(),
    fetchOffers(),
  ]);
}

// ❌ WRONG — independent requests waiting for each other
Future<void> fetchHomeData() async {
  await fetchBanners();
  await fetchCategories();
  await fetchOffers();
}
```

---

## 10. Reactive Variables — Rx, Rxn, and Normal

Decide which kind you need **before** adding `.obs`.

### Type 1 — Reactive (`Rx`)

- Use `Rx` **only** for a value whose change must **drive a UI change or an animation**.
- **Only** the widget that must react to that value is wrapped in `Obx()` — nothing more (§12).

### Type 2 — Reactive and nullable (`Rxn`)

- When a reactive value can be **null**, use **`Rxn<T>`** — not `Rx<T?>`, not `.obs` on a nullable.
- Same rule applies: only use it when the null/non-null change must drive the UI.

### Type 3 — Normal (no `Rx`)

- A value that changes but does **not** change the UI and does **not** drive an animation stays an explicitly typed normal value, optionally `final`.
- **No `.obs`.** Do not make it reactive just because it changes.

```dart
// Type 3 — normal var: changes, but nothing in the UI observes it
final String apiUrl = 'https://api.example.com';
int retryCount = 0;

// Type 1 — Rx var: its change must rebuild a widget / run an animation
final RxString userName = ''.obs;
final RxBool isMenuOpen = false.obs;
final RxList<Product> products = <Product>[].obs;

// Type 2 — Rxn: reactive AND nullable
final Rxn<User> selectedUser = Rxn<User>();
final Rxn<DateTime> pickedDate = Rxn<DateTime>();
```

> **Rule:** Do not make everything `.obs` — only what the UI actually observes.
> Over-using `.obs` causes unnecessary rebuilds.

---

## 11. Page / Endpoint Status — StateMixin + `controller.obx()`

When an endpoint call has **multiple statuses** (loading / empty / error / success), you **must** use GetX `StateMixin` and render it with `controller.obx()`.

- Use `controller.obx()` when the **full page** (or the whole body) depends on the controller's response.
- Controller: `extends GetxController with StateMixin<T>`; drive status with `change(data, status: ...)`.
- View: render with **`controller.obx(...)`**.
- **Do NOT use `Obx()` to handle page/endpoint status.** `Obx()` is only for one widget that changes (§12) — never for page status.
- **Do NOT** scatter `bool isLoading` + `bool hasError` variables — the status is one object, owned by `StateMixin`.

### 11.1 Success first, with meaningful extraction

- Write the success builder first, followed by `onLoading`, `onEmpty`, and `onError`.
- Keep a small success tree inline. Extract meaningful page sections when the 200-line limit requires it; reusable components may be extracted under §6.
- Page sections belong in `views/pages/`; cards and list items belong in `views/widgets/`.
- A success builder may return an extracted body widget. This follows the same composition rules as the rest of the page.
- Use `StateMixin` for endpoint status and `Obx` for individual reactive values; do not replace page status with scattered loading/error booleans.

```dart
// ProductsPage build excerpt; other statuses remain explicit.
return Scaffold(
  body: controller.obx(
    (products) => ProductsBody(products: products!),
    onLoading: const ProductsLoading(),
    onEmpty: const ProductsEmpty(),
    onError: (message) => ProductsError(message: message),
  ),
);
```

`ProductsBody` is a page section with its own real layout, not a pass-through wrapper. The page and all sections share `ProductsController`. Their status widgets live in `shared/pagesState/products/`.

---

## 12. Obx Usage Rules — Wrap the Leaf, Not the Tree

- `Obx()` is for **one widget** that needs a UI change or an animation (Type 1 / Type 2 variable, §10) — never for page status (§11).
- **Every `Obx` costs memory** — use it only on the widget that changes, never as a page-level wrapper.
  Full page depends on the response → `controller.obx()`; one widget changes → `Obx()` on that widget only.
- Wrap **only** the widget that changes — never wrap the entire page in one `Obx`.
- Avoid rebuilding the full widget tree for a single value change.

### 12.1 Push Obx as deep as it goes

In a tree like `Column → Container → Row → Text`, if only the `Text` reacts to the value, put the
`Obx` around **the `Text` alone**. The rebuild then starts at the `Text` — not at the `Column`.
Everything above it stays untouched and stays `const`.

```dart
// ❌ WRONG — Obx at the top: the whole Column/Container/Row rebuilds for one string
Obx(() => Column(
  children: [
    Container(
      child: Row(
        children: [
          const Icon(Icons.person),
          Text(controller.userName.value),
        ],
      ),
    ),
  ],
))

// ✅ CORRECT — Obx wraps the Text only: rebuild starts at the Text
Column(
  children: [
    Container(
      child: Row(
        children: [
          const Icon(Icons.person),
          Obx(() => Text(controller.userName.value)),
        ],
      ),
    ),
  ],
)
```

---

## 13. Comments

- **Comments in this project are written for the AI, not for the programmer.** The developer reading
  this codebase does not need them and does not want them.
- **Every comment is ONE line.** If what you want to say is long, compress it into a single line —
  never a multi-line comment block.
- **Never write big or long comments.** A comment is **short** — a few words, not a paragraph squeezed
  into one long line. No doc-comment blocks, no banner/section comments, no explanations of history.
- Do not narrate the obvious. Comment only where the reason is not visible in the code itself.

```dart
// ❌ WRONG — multi-line block, explains what the code already says
/// Fetches the products from the repository, then
/// checks whether the returned list is empty, and
/// finally updates the state accordingly.
Future<void> fetchProducts() async { ... }

// ❌ WRONG — one line, but far too long
// We call this here because the backend sometimes returns an empty list when the user has no products yet, and that is not an error so we must not show the error screen.
Future<void> fetchProducts() async { ... }

// ✅ CORRECT — one short line, states the non-obvious reason
// Empty list is valid, not an error.
Future<void> fetchProducts() async { ... }
```

---

## 14. Localization — GetX Translations with Dart const maps

- All localization goes through **GetX `Translations`**. Do **not** use `flutter_localizations` ARB files,
  `gen-l10n`, `AppLocalizations`, or `.arb` of any kind. There is no code generation step.
- Translations are **Dart `const` maps**, never JSON.
- **Arabic** is the default language; **English** is also supported.
- Localization lives in **`lib/core/localization/`**, like any other core service (§1).

### 14.1 Structure

```
lib/core/localization/
├── localization.dart           # the Translations class and supported language maps
├── localizationKeys.dart       # LocalizationKeys — one static const per key
├── arabicTranslations.dart     # const Map<String, String> arabicTranslations
└── englishTranslations.dart    # const Map<String, String> englishTranslations
```

- **Never** `.json` translation files, never `rootBundle`, never a translation file listed under `assets:` in `pubspec.yaml`.
- Register each supported language map in `Localization.keys`.

### 14.2 Keys and usage

- Every key is a `static const String` in **`LocalizationKeys`**, `lowerCamelCase` (§2), with the same name as its value.
- Always read text through the key class: **`LocalizationKeys.cartTitle.tr`** — **never** a string literal like `'cartTitle'.tr`.
  A wrong key then fails the build instead of silently showing the key to the user.
- Every language map has **exactly** the same keys as `LocalizationKeys`.
- Parameters use `@name` inside the value and are supplied with `.trParams`.
- Never hardcode user-facing text in a widget or a controller.

```dart
// localizationKeys.dart
class LocalizationKeys {
  static const String cartTitle = 'cartTitle';
  static const String cartItemCount = 'cartItemCount';
}

// englishTranslations.dart
const Map<String, String> englishTranslations = {
  LocalizationKeys.cartTitle: 'Shopping cart',
  LocalizationKeys.cartItemCount: 'Items: @count',
};

// ✅ CORRECT
Text(LocalizationKeys.cartTitle.tr)
Text(LocalizationKeys.cartItemCount.trParams({'count': '$itemCount'}))

// ❌ WRONG — string-literal key (a typo shows the key to the user), and hardcoded text
Text('cartTitle'.tr)
Text('Shopping cart')
```

### 14.3 Wiring

```dart
// localization.dart
import 'arabicTranslations.dart';
import 'englishTranslations.dart';

class Localization extends Translations {
  static const Locale fallback = Locale('ar');
  static const List<Locale> supportedLocales = <Locale>[Locale('ar'), Locale('en')];

  @override
  Map<String, Map<String, String>> get keys => <String, Map<String, String>>{
    'ar': arabicTranslations,
    'en': englishTranslations,
  };
}
```

- Restore the saved supported locale; use Arabic if none is saved or the value is unsupported.
- Adding a language requires its Dart map, registration in `keys`, and entry in `supportedLocales`.
- Test that every language map contains exactly the declared localization keys.

---

## 15. Request Paths — Every Value Goes Through `ApiPath` — ABSOLUTE RULE

- **Every** value placed inside a request path (ids, item ids, codes — anything) **must** go through `lib/core/api/apiPath.dart`. No exceptions.
- Templated endpoints use `ApiPath.fill(template, {...})`. A value appended to a path uses `ApiPath.segment(value)`.
- `ApiPath` percent-encodes each value as **one** path segment, so `/`, `?`, `#` and `{` can never change the path or add a query, and it rejects `.` and `..` (Dio normalizes dot segments away even after encoding).
- **Never** build a request path with `replaceFirst` / `replaceAll`, raw `$id` interpolation, or `+` concatenation.
- **Never** call `Uri.encodeComponent` on a value before passing it to `ApiPath` — that double-encodes it and breaks the request.
- A new templated endpoint must also be added to `test/core/api/apiPath_test.dart`.

```dart
// ✅ CORRECT
apiServices.get(ApiPath.fill(EndPoints.productDetail, {'id': id}), ...);
apiServices.get(
  ApiPath.fill(EndPoints.serviceSpecialistTimeSlots, {
    'id': serviceId,
    'specialistId': specialistId,
  }),
  ...
);
apiServices.delete('${EndPoints.cartItems}/${ApiPath.segment(itemId)}', ...);

// ❌ WRONG — the value is not encoded and can change the request path
apiServices.get(EndPoints.productDetail.replaceFirst('{id}', id), ...);
apiServices.delete('${EndPoints.cartItems}/$itemId', ...);

// ❌ WRONG — double encoding
ApiPath.fill(EndPoints.cancelCheckout, {'id': Uri.encodeComponent(orderId)});
```

---

## 16. New Pages — Routes and Bindings

- Declare the route constant in `lib/core/routing/appRoutes.dart`.
- Register its page and binding in the project's route configuration (`appPages.dart` in a new project).
- Add its typed navigation function to `WidgetsRouteClick` in the same change (§23).
- Define and validate required arguments and identifiers in the controller. Missing or invalid arguments must produce a controlled state, not a crash.
- Apply authentication and authorization guards where the feature requires them.
- Inspect an existing project's routing before editing it; do not create a competing registry or assume unavailable wrapper classes.
- Page loading strategy is not prescribed by these rules.

---

## 17. Code Formatting — Never Break a Call Chain

- **Never** put a line break before a `.` in a call chain. The whole chain stays on **one line**.
- When the call takes a body (a map, a list, a closure, arguments that need several lines),
  break **after `{` / `(` / `[`** and **before `}` / `)` / `]`** — never before the `.`.

```dart
// ❌ WRONG — new line before the dot
FirebaseMessaging.instance
    .setForegroundNotificationPresentationOptions(alert: true);

// ✅ CORRECT — chain on one line
FirebaseMessaging.instance.setForegroundNotificationPresentationOptions(alert: true);

// ✅ CORRECT — the body opens after { and closes before }
FirebaseMessaging.instance.fallback({
  'key': value,
  'otherKey': otherValue,
});
```

### 17.1 Widget trees — where a line may break

In a widget tree (`Container`, `Padding`, `Column` …) a new line is allowed **only** in these places:

- after **`(`**
- after **`),`** or **`)`**
- after **`;`**
- after **`,`** that ends an argument

**Never** break a line before or after a **`.`** — `EdgeInsets.all(context.sizes.spacingSmall)`, `context.colors.backgroundPrimary`,
`Theme.of(context).textTheme.bodyLarge`, `controller.title.value` all stay on one line.

```dart
// ✅ CORRECT
return Container(
  padding: EdgeInsets.symmetric(horizontal: context.sizes.spacingMedium, vertical: context.sizes.spacingSmall),
  decoration: BoxDecoration(
    color: Theme.of(context).colorScheme.surface,
    borderRadius: BorderRadius.circular(context.radii.card),
  ),
  child: Text(controller.title.value),
);

// ❌ WRONG — line broken at the dot
return Container(
  padding: EdgeInsets
      .symmetric(horizontal: context.sizes.spacingMedium, vertical: context.sizes.spacingSmall),
  decoration: BoxDecoration(
    color: Theme.of(context)
        .colorScheme
        .surface,
  ),
);
```

---

## 18. Design System — Theme Only — ABSOLUTE RULE

The app uses **only** the design system from our UI/UX designer (`designSystem/tokens.json`),
exposed through `lib/core/theme/`. Nothing in the UI comes from outside it.

- Colors → `context.colors`, text styles → `context.textStyles`, sizes/spacing → `context.sizes`,
  radii → `context.radii`, shadows → `context.shadows`.
- Component look (buttons, app bar, inputs, cards …) comes from the theme builders in
  `lib/core/theme/components/` — do not restyle a component inline.
- **Never** hardcode a value: no `Color(0xFF…)`, no `Colors.red`, no raw `TextStyle(fontSize: 14)`,
  no magic `16` padding, no `BorderRadius.circular(12)` with a literal.

```dart
// ✅ CORRECT
Container(
  height: context.sizes.buttonHeightMedium,
  decoration: BoxDecoration(
    color: context.colors.backgroundPrimary,
    borderRadius: BorderRadius.circular(context.radii.card),
  ),
  child: Text(controller.title, style: context.textStyles.largeRegular),
)

// ❌ WRONG — values from outside the design system
Container(
  padding: const EdgeInsets.all(16),
  color: const Color(0xFFF5F5F5),
  child: Text(controller.title, style: const TextStyle(fontSize: 14)),
)
```

### 18.1 Existing design system

- Read existing tokens, theme extensions, component themes, assets, and supplied designs before building UI.
- Reuse their semantic tokens; do not add an almost identical token to bypass an existing one.
- If a required token is missing and the supplied design specifies it, add it centrally with values for both modes, then use it in the UI.
- If a missing design decision cannot be inferred, identify the component, missing token, and needed decision; ask only about that unresolved choice. Continue independent work.
- Record the token additions in the delivery report.

### 18.2 New app without a design system

- Create `designSystem/tokens.json` and `lib/core/theme/` before implementing feature screens.
- Use supplied branding or design references. If none exists, establish a coherent initial system and explicitly report its assumed visual direction.
- Define semantic colors for both modes, typography, spacing, sizes, radii, shadows, and component themes for controls the app actually uses.
- Expose `context.colors`, `context.textStyles`, `context.sizes`, `context.radii`, and `context.shadows` through typed theme extensions.
- Store literal design values only in the central tokens/theme implementation. Screens and widgets consume tokens, not literals.
- Central theme infrastructure may define theme extensions and component builders; rendered reusable widgets still belong in `shared/widgets/`.
- A formatting example's token names are illustrative: verify or create them before use, never assume they already exist.

---

## 19. Liquid Glass — iOS Only

We use the `glass` package (pub.dev) for Liquid Glass.

- **iOS:** the **app bar**, the **navigation bar**, and the **buttons on the app bar** always use Liquid Glass.
- **Android:** **never** use Liquid Glass — use our own design-system UI (§18).
- The platform choice is made **once**, in a shared widget in `lib/shared/widgets/` (§1) — pages use that widget
  and never write their own `Platform.isIOS` check for the app bar / nav bar.

---

## 20. Light and Dark Mode — Always Both

- **Every** screen and widget must work in **both** light and dark mode.
- Every color comes from the theme (§18), so it switches with the mode — never a color that only fits one mode.
- `lib/core/theme/appTheme.dart` provides **both** a light and a dark `ThemeData`; a token added to the
  theme gets a value for **both** modes.
- Check every new screen in light **and** dark before calling it done.

---

## 21. Code Hygiene

### 21.1 Null handling and try/catch — only where it can really happen

- Use `?`, `!`, `??` **only** on a value that is really nullable (declared `Type?`, or nullable from the API).
- Use `try/catch` **only** around code that can really throw (API call, parsing, platform call) — not everywhere "just in case".

```dart
// ❌ WRONG — userName is a non-null String
Text(controller.userName ?? '')

// ✅ CORRECT — only the nullable field gets ??
Text(product.discountLabel ?? '')
```

### 21.2 Keep it simple — no over-engineering

- No abstract class / interface / factory / helper / generic for something used **once**.
- Write the simplest code that does the job. Add a layer only when a second real use exists.

### 21.3 Never invent code

- Before importing a file or calling a class/function, **check it exists** in the project.
- Never guess a file path, class name, function name, or package API.

### 21.4 Touch only what was asked

- Change **only** the code the task needs.
- No reformatting whole files, no renaming, no "cleanup" of unrelated code in the same change.

### 21.5 Clear names — not generic, not huge

- A name says what the thing **is**: `selectedProduct`, `cartItems`, `openProductDetails` — not `data`, `item`, `result`, `value`, `handleTap`.
- Keep names short: **two to four words**. ❌ `selectedProductFromHomeSliderListForCart`.

### 21.6 No print / debugPrint

- **Never** leave `print`, `debugPrint`, or `log` for debugging in the code.

### 21.7 No dead code, no TODO

- **Never** leave commented-out code — delete it (git keeps history).
- **Never** write `// TODO` / `// FIXME` — tasks live in **ClickUp**, not in code.

### 21.8 Always `const`

- Every widget / constructor / value that can be `const` **must** be `const`.

### 21.9 Imports — only what is used

- Import **only** what the file uses. No unused imports.

### 21.10 Always write the type

- Every variable, field, parameter, and return type has an **explicit type**.
- No untyped `var` / `final` without a type, no `dynamic`, no raw `Map<String, dynamic>` passed around — use a typed model.

```dart
// ❌ WRONG
var count = 0;
final items = [];
dynamic response;

// ✅ CORRECT
int count = 0;
final List<Product> items = <Product>[];
final ProductResponse response;
```

---

## 22. Dependency Injection — Bindings Only

- **Never, never** use `Get.put()`. Every controller and service is registered in the feature's **binding**.
- `Get.find()` is allowed **only** where the controller is **already registered** by the route flow
  (its page's binding, or a page earlier in the same flow).
- Do **not** write runtime checks like `Get.isRegistered()` — check the **routes and page flow** instead,
  and use `Get.find()` only when the flow guarantees it.

---

## 23. Navigation — Through `WidgetsRouteClick` Only

- All navigation goes through **one** file: `lib/core/routing/widgetsRouteClick.dart` (class `WidgetsRouteClick`).
- Every route gets **one function** there that opens it with its arguments.
- Pages, widgets, and controllers call that function — **never** `Get.toNamed(...)` / `Get.to(...)` directly.
- A new route → add its function to `WidgetsRouteClick` in the same change.

```dart
// ✅ CORRECT — in the controller
void openProductDetails(Product product) {
  WidgetsRouteClick.productDetails(productId: product.id);
}

// ❌ WRONG — navigation written directly
Get.toNamed(AppRoutes.productDetails, arguments: {'id': product.id});
```

---

## 24. Data Access — Always a Datasource

- A controller **never** calls the API, the HTTP client, or local storage directly.
- Every API / storage call lives in the feature's `data/datasources/`, and the controller calls the datasource.

---

## 25. Performance — Always Dispose and Close

Performance is a top priority. Everything a controller opens, it **must** close in `onClose()`:

- `ScrollController`, `TextEditingController`, `PageController`, `TabController`, `AnimationController`, `FocusNode` → `.dispose()`
- `StreamSubscription` → `.cancel()`
- `Timer` → `.cancel()`
- `Worker` (`ever`, `debounce`, `interval`) → `.dispose()`

```dart
@override
void onClose() {
  scrollController.dispose();
  searchTextController.dispose();
  searchDebounceTimer?.cancel();
  notificationSubscription.cancel();
  super.onClose();
}
```

---

