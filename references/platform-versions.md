# Flutter Craft — Platform Versions

Section numbers preserve cross-references across this skill.

- 37. Stable, Compatible Android and iOS SDK Versions

## 37. Stable, Compatible Android and iOS SDK Versions

- Prefer the newest stable, supported toolchain combination that is compatible with the selected Flutter stable release, native dependencies, plugins, development host, and CI environment. Do not select beta, preview, RC, or obsolete versions for production unless explicitly requested.
- Check current official Flutter, Android, and Apple documentation at project setup, planned toolchain upgrades, and release preparation. Do not hardcode a permanent “latest” version number in this skill or infer compatibility from release recency alone.
- Pin the selected toolchain versions in project configuration and CI; record versions, selection reasons, verification date, and official reference links in established project documentation. Re-check requirements without automatically upgrading a working project during unrelated feature work.

### 37.1 Android

- Choose a stable `compileSdk` supported by the Flutter/Android build toolchain and required dependencies; it must be at least the selected `targetSdk`.
- Prefer the latest stable compatible `targetSdk` and meet the current Google Play target-API requirement applicable to the app type and submission date. Test the platform behavior changes activated by the selected target.
- Set `minSdk` from the intended device coverage and the maximum minimum required by Flutter, plugins, native libraries, and actual features. Do not equate it with `compileSdk` or `targetSdk` or raise it merely because a newer SDK exists.
- Verify Android Gradle Plugin, Gradle wrapper, JDK, Kotlin, Build Tools, and any required NDK as a compatible set. Keep defaults managed by the supported toolchain unless a documented requirement calls for an override.

### 37.2 iOS

- Choose a supported stable Xcode and its bundled iOS SDK that meet current App Store submission requirements and are compatible with Flutter, plugins, the macOS host, and CI.
- Select the iOS deployment target independently from the build SDK: satisfy Flutter/plugin minimums and the intended user device coverage without dropping older supported devices unnecessarily.
- Keep the deployment target consistent across app targets, extensions where appropriate, dependency-manager configuration, and build configurations; verify actual dependency requirements instead of forcing incompatible overrides.
- Use runtime availability checks and suitable fallbacks for newer native APIs when supporting older OS versions.

### 37.3 Verification and compatibility limits

- Build release artifacts for both requested platforms and exercise critical flows on the minimum supported OS and a recent supported OS, using physical devices for behavior or performance that simulators cannot establish.
- If the newest stable version is incompatible, use the newest verified compatible stable combination, report the exact blocker and tradeoff, and define an upgrade path. Do not silently lower store requirements or claim an incompatible combination is supported.
- If a mandatory store requirement cannot be met, report release delivery as blocked by that concrete requirement; continue independent implementation work where possible.

Official references: [Flutter supported platforms](https://docs.flutter.dev/reference/supported-platforms), [Android configuration](https://developer.android.com/build/configure-app-module), [Google Play target API requirements](https://developer.android.com/google/play/requirements/target-sdk), [Xcode compatibility](https://developer.apple.com/xcode/system-requirements/), [App Store submission](https://developer.apple.com/app-store/submitting/).


---

