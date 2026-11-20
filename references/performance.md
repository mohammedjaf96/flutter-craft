# Flutter Craft — Performance

Section numbers preserve cross-references across this skill.

- 34. Lazy Lists, Grids, and Item Extents
- 35. Search, Heavy Computation, Rendering, and Memory
- 36. Very Long Articles and Text
- 38. Navigation Bar Pages — PageView and Memory Release

## 34. Lazy Lists, Grids, and Item Extents

- Build every collection-driven scrolling list or grid on demand with `ListView.builder`, `GridView.builder`, or a sliver builder delegate. `ListView.separated` is also a lazy builder when separators are needed.
- Do not eagerly construct collection children using `ListView(children: ...)`, `Column(children: collection.map(...).toList())`, or `List.generate` passed as scrolling children. Ordinary fixed page layout is not a data list.
- Construct each item inside `itemBuilder`; do not prebuild and cache a list of widget instances. Provide the actual `itemCount` when known.
- Preserve lazy construction with a bounded viewport. Do not expand a long builder list to full content height with `shrinkWrap: true` inside another scroller; use a coordinated sliver layout when combining scrolling sections.
- Lazy widget construction is separate from network pagination: a builder does not fetch API pages automatically. Define paginated loading when the dataset requires it, with next-page loading, failure/retry, end-of-data, and duplicate-request handling in the page controller.
- For a `ListView.builder` whose items share a known main-axis extent, set `itemExtent` using a design-system size. Extent means height in a vertical list and width in a horizontal list; it constrains the item rather than measuring it automatically.
- If uniform extent should follow a representative widget's layout, use `prototypeItem` instead. Verify localized content and text scaling fit the resulting extent.
- If extents vary but can be determined cheaply and accurately from item data and current layout constraints, use `itemExtentBuilder` where the selected Flutter SDK supports it. Its callback must not do network access or expensive measurement.
- Choose only one of `itemExtent`, `prototypeItem`, and `itemExtentBuilder`. Do not impose guessed fixed sizes on unknown content; allow normal layout when no accurate extent can be supplied.
- For grids, configure the grid delegate with appropriate design-system sizing; do not assume ListView extent properties exist on GridView or every other constructor.
- Verify fast scrolling, large data, localization, and increased text size for clipping, stutter, and memory growth on the requested targets.

Official references: [Long lists](https://docs.flutter.dev/cookbook/lists/long-lists), [ListView](https://api.flutter.dev/flutter/widgets/ListView-class.html), [itemExtentBuilder](https://api.flutter.dev/flutter/widgets/ListView/itemExtentBuilder.html).


---

## 35. Search, Heavy Computation, Rendering, and Memory

### 35.1 Search and repeated requests

- Debounce text-driven remote search with a configurable delay appropriate to the interaction; do not delay explicit submission unnecessarily.
- Normalize and deduplicate identical in-flight queries when their parameters and user context match.
- Cancel obsolete requests where supported and use a request generation or query identifier so old responses cannot replace newer results.
- Own search state, debounce workers/timers, and cancellation in the page controller; dispose or cancel them in `onClose()`.

### 35.2 CPU-intensive work

- When heavy parsing, transformations, or calculations demonstrably block the UI, move suitable pure computation to an isolate on supported targets.
- Keep widgets, rendering, and UI work on the main isolate. Do not move `TextPainter` layout or other `dart:ui` work into a background isolate.
- Verify execution support on each requested target; do not assume a helper creates a background thread everywhere.
- Account for startup and message-copy costs. For repeated expensive work, consider reusing a worker instead of spawning one per tiny operation.
- Keep orchestration, task ownership, results, and page state in the one page controller; a compute callback is not another controller or a split of that controller.

### 35.3 Layout and painting cost

- Avoid unnecessary intrinsic layout passes, especially `IntrinsicHeight` or `IntrinsicWidth` inside repeated scrolling items. Prefer constraints supplied by the parent and design-system sizing.
- Avoid unnecessary `saveLayer`, `Clip.antiAliasWithSaveLayer`, full-subtree opacity, and repeated costly blur effects. Preserve required visual behavior and verify whether a simpler equivalent actually helps.
- Do not apply rendering changes mechanically or remove approved design effects without evidence and an equivalent result.

### 35.4 Bounded retained state

- Bound cache capacity by an appropriate count or memory budget and define eviction/expiry. Avoid retaining an entire unbounded dataset, parsed documents, or widget instances indefinitely.
- Release page-owned large data and resources when the controller closes; ensure subscriptions, workers, timers, and global references do not keep it alive.
- Retain only the page or navigation state required by the actual flow. Do not change shared-object ownership or clear shared resources still in use.
- Keep user-specific caches isolated and clear private data on logout (§30–31).

### 35.5 Repaint boundaries and keep-alive

- Inspect existing delegate defaults before adding `RepaintBoundary`; builder lists commonly already insert boundaries. Do not add a second boundary around every item automatically.
- Add or adjust boundaries only for a meaningful independently repainting region when the gain justifies extra layers and memory.
- Use keep-alive only for state that must survive scrolling out of view. Do not force every long-list item to remain alive.
- Do not globally disable automatic keep-alive when items require it, such as active input; choose retention behavior for the actual item lifecycle.
- Verify scrolling and state preservation after these changes; do not assume that more boundaries or retained items always improves performance.

Official references: [Performance best practices](https://docs.flutter.dev/perf/best-practices), [Isolates](https://docs.flutter.dev/perf/isolates), [SliverList lifecycle](https://api.flutter.dev/flutter/widgets/SliverList-class.html).


---

## 36. Very Long Articles and Text

### 36.1 Lazy article blocks with faithful layout

- Represent very long articles as an ordered sequence of semantic content blocks, such as headings, paragraphs, images, quotations, and code blocks. Render them on demand through `SliverList.builder`, `ListView.builder`, or the equivalent lazy builder supported by the selected SDK.
- Preserve the original content order exactly. Do not omit, duplicate, reorder, or rewrite text, inline spans, links, images, captions, or other article content as a performance optimization.
- Splitting must not change line spacing, paragraph spacing, indentation, alignment, text direction, typography, or intended line wrapping. Preserve the original design's layout and whitespace semantics, including explicit line breaks and the distinction between soft wraps and paragraph breaks.
- Do not add `Padding`, `SizedBox`, separators, margins, or extra newlines merely because a new block or widget was introduced. Apply existing article spacing exactly once at the original boundaries.
- For an existing renderer, compare the original and optimized article at the same width, font, locale, text scale, and direction. Confirm boundary spacing, line breaks, and content order remain unchanged before accepting the optimization.

### 36.2 Exceptionally large paragraphs

- Split an exceptionally large paragraph only at logical boundaries and only when the rendering strategy preserves its original visual flow, text shaping, inline formatting, and line spacing.
- Do not arbitrarily split at character counts, individual words, or visual lines; this can change wrapping, Arabic shaping, bidirectional ordering, or paragraph layout.
- Do not turn an internal split into a new semantic paragraph or insert a paragraph gap. Preserve the original text and its original newline behavior.
- If ordinary separate text widgets cannot reproduce the original paragraph layout, keep that paragraph intact and lazily render its surrounding article blocks. Do not trade visual fidelity for additional fragmentation.

### 36.3 Variable text extents

- Do not force a uniform `itemExtent` or `prototypeItem` across article paragraphs of differing height.
- Use `itemExtentBuilder` only when each block's extent is accurate for its actual content and layout constraints. Otherwise let normal layout determine block height.
- Do not eagerly lay out every paragraph with `TextPainter` solely to supply extents; that can defeat the intended lazy-work benefit.
- If measured extents are cached, bound that cache and invalidate affected measurements when content, available width, typography, font availability, locale, direction, or text scaling changes.

### 36.4 Parse when content changes

- Parse HTML, Markdown, or other article markup into typed content blocks when the source changes, not inside widget `build` or on every scroll update.
- Reuse parsed content while its source and parsing configuration remain unchanged. Bound retained parsed documents; do not cache widgets as the article model.
- Avoid repeating code highlighting or equivalent expensive text transformations for unchanged blocks.
- Move suitable heavy pure parsing to an isolate when justified under §35.2. Widget creation, text layout, and painting remain on the main isolate.
- The page controller owns article state, parsing orchestration, and asynchronous result handling; keep the one-controller rule.

### 36.5 Reserve image space

- Reserve the article image's intended display space from known dimensions or aspect ratio before it loads, respecting the content width and design-system layout.
- Keep placeholder and loaded-image geometry consistent to avoid shifting surrounding text during reading.
- Use actual metadata where available; do not invent a fixed ratio that distorts the image or changes the article's intended layout. If geometry is unknown, resolve metadata when practical rather than promising zero shift.

### 36.6 Isolate scroll-driven updates

- Keep article content independent from per-scroll reactive state. Update a reading-progress indicator or back-to-top control locally rather than rebuilding the whole article on every scroll event.
- Emit updates only when the displayed state meaningfully changes; throttle high-frequency work when appropriate to the interaction.
- Keep scroll listeners and related workers owned by the page controller and release them in `onClose()`.
- Verify long-article scrolling, asynchronous image loading, and block boundaries with Arabic and the supported text scales. Preserve the original article layout while applying these optimizations.

Official references: [SliverList](https://api.flutter.dev/flutter/widgets/SliverList-class.html), [itemExtentBuilder](https://api.flutter.dev/flutter/widgets/ListView/itemExtentBuilder.html), [Isolates](https://docs.flutter.dev/perf/isolates).


---

## 38. Navigation Bar Pages — PageView and Memory Release

- When an in-app navigation bar switches among peer pages, use `PageView.builder` as the page host. Keep the selected destination and the `PageView` synchronized for navigation-bar taps and swipe navigation when enabled.
- The navigation host has its own controller for selection, `PageController`, and transition/lifecycle orchestration. Each main destination still has its one page controller (§9); do not create controllers for individual navigation widgets.
- Build destinations on demand. Do not precreate all destination widgets or use `IndexedStack`, `Offstage`, unconditional keep-alive, or a retained list of page instances to keep inactive destination contents alive.
- After a transition settles, release the departed destination's mounted content, page-owned controller, large datasets, subscriptions, timers, workers, media, focus, and other disposable resources. Keep outgoing/incoming content only while required for the transition; handle cancelled swipes and rapid destination changes correctly.
- `PageView.builder` alone does not guarantee disposal of every inactive page or GetX controller. Implement explicit active-page ownership and cleanup; replace any retained inactive destination content with lightweight placeholders when necessary without changing page indices or navigation geometry.
- Remove departed widget subtrees before disposing dependencies they may still read. Determine departure from the settled scroll position rather than assuming `onPageChanged` always means a transition has finished.
- Register each destination controller through its binding on activation, and explicitly release its destination-scoped registration on departure through the project's supported dependency lifecycle. Pages inside a persistent host are not automatically popped GetX routes; do not rely on route cleanup alone.
- Do not register destination-scoped controllers as permanent or allow background recreation after departure. Re-entering a destination must create fresh content and a fresh controller through its binding, then load required data through the normal page lifecycle.
- Invalidate or cancel outstanding page requests before releasing ownership so late responses cannot repopulate closed-page state or recreate disposed dependencies.
- Release only destination-owned resources. Shared services and durable application data, such as the authenticated session or saved cart, keep their defined application lifetime; leaving a tab is not deletion of persisted user data.
- `PageController.keepPage` stores/restores the selected page position; it is not a widget/controller keep-alive switch. Do not treat `keepPage: false` as proof that page contents were released.
- The host controller disposes its `PageController` and transition listeners in `onClose()`.
- Verify repeated switching, rapid taps, completed/cancelled swipes, and re-entry. Confirm departed content unmounts, destination controllers close, resource counts stay bounded, and memory does not grow without bound. Releasing references makes objects collectible; garbage collection need not immediately reduce process memory.

Official references: [PageView.builder](https://api.flutter.dev/flutter/widgets/PageView/PageView.builder.html), [PageController.keepPage](https://api.flutter.dev/flutter/widgets/PageController/keepPage.html).
