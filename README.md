# Flex Launcher

A complete, themeable Android **HOME launcher** written in Kotlin with Jetpack Compose and
Material 3. Flex Launcher discovers your real installed apps through `PackageManager`, launches
them, shows recent apps at the top of the app drawer, and lets you restyle almost everything:
20 bundled themes plus a full custom palette editor, 28 font families, 12 real icon masks,
8 animation styles and an edge-to-edge transparent status bar.

The repository is GitHub-ready: push it, and the included **GitHub Actions** workflow runs the
unit tests, builds a signed debug APK, verifies the package name / HOME intent filter / signature,
and uploads the APK as a downloadable artifact.

> **Status:** this project was built, compiled and unit-tested in a headless Linux container
> (JDK 17, Android SDK 35, Gradle 8.9). A debug APK was produced and inspected (`aapt2`, `apksigner`).
> It has **not** been run on a physical phone or emulator here — see
> [Testing status](#testing-status) for exactly what was and was not verified.

---

## Table of contents

1. [What it is](#what-it-is)
2. [Feature list](#feature-list)
3. [Screens & navigation](#screens--navigation)
4. [Requirements](#requirements)
5. [Build locally](#build-locally)
6. [Open in Android Studio](#open-in-android-studio)
7. [Install on a phone & set as default launcher](#install-on-a-phone--set-as-default-launcher)
8. [Download the APK from GitHub Actions](#download-the-apk-from-github-actions)
9. [Permissions & privacy](#permissions--privacy)
10. [Project structure](#project-structure)
11. [Testing status](#testing-status)
12. [Troubleshooting](#troubleshooting)
13. [Known Android platform limitations](#known-android-platform-limitations)
14. [Licences](#licences)

---

## What it is

Flex Launcher is a genuine launcher, not a home-screen demo:

* `MainActivity` declares `ACTION_MAIN` + `CATEGORY_HOME` + `CATEGORY_DEFAULT` and is exported,
  `singleTask`, `stateNotNeeded`, so Android offers it in the "Use as Home app" chooser.
* The window is translucent and shows the system wallpaper; the Compose UI draws its own
  themeable background on top with a user-controlled opacity.
* Apps come from `PackageManager.queryIntentActivities(MAIN/LAUNCHER)`; icons are the real
  `Drawable`s provided by each package, converted to bitmaps and masked by the chosen shape.
* Package installs, removals, replacements and disabled/enabled state changes are observed with a
  runtime `BroadcastReceiver`, so the drawer updates without a restart.
* Nothing is hardcoded: there is no sample application list anywhere in the app.

---

## Feature list

### Launcher core
- Real HOME intent filter, default-launcher flow (button opens Android's *Default apps → Home*
  screen; the "Set as default" pill only disappears when Android reports Flex Launcher as default).
- Installed-app discovery with `<queries>` scoped package visibility (no `QUERY_ALL_PACKAGES`).
- Real app labels and icons; monogram fallback when an icon cannot be loaded.
- One-tap launching with `ActivityNotFoundException` / `SecurityException` handling.
- Responsive grids (3–7 columns), icon size/spacing, tablet-friendly padding, gesture- and
  three-button-navigation insets, display-cutout support.
- Live package-change updates; graceful handling of uninstalled pins/hidden entries.
- First-run onboarding (set as default → optional usage access → done), all steps reachable later.
- Clock/date widget, pinned ("favourite") home shortcuts with reordering, haptic toggle.

### Recent apps (top of the app drawer)
- `UsageStatsManager` based ordering when Usage Access is granted, with Android's
  `queryEvents` foreground events and a `queryUsageStats` fallback for OEMs that emit no events.
- **Always-on fallback:** Flex Launcher records its own launches locally (package → timestamp), so
  the section keeps working when Usage Access is denied.
- Configurable item count (3–12), enable/disable, optional home-screen recent row,
  clear-history action, permission status + shortcut to the system Usage Access screen,
  automatic refresh when you return to the launcher.
- Clearly labelled as approximate: no access to the system Recents screen or running tasks.

### Theme Studio
- 20 named presets: Pure AMOLED Black, Midnight Black, Graphite Gray, Dark Titanium, Ocean Blue,
  Arctic Blue, Neon Cyberpunk, Purple Aurora, Deep Violet, Emerald Green, Forest Green,
  Sunset Orange, Rose Pink, Ruby Red, Golden Luxury, Pastel Dreams, Minimal White, Warm Sand,
  Frosted Glass, Gradient Galaxy.
- Live preview card, preset swatches, light/dark/system mode, pure-AMOLED override,
  Material You dynamic colours (Android 12+), wallpaper-derived palette (androidx Palette).
- Custom editor: primary, secondary, accent, background, surface, card, primary/secondary text,
  gradient start/end + direction (none, top→bottom, bottom→top, left→right, diagonal, radial).
- HEX colour fields with strict validation (`#RGB`, `#RRGGBB`, `#AARRGGBB`), swatches, reset,
  instant application; text colours are auto-corrected when contrast drops below WCAG AA.
- Background opacity, wallpaper dim, card corner radius, layout density — all applied to the real UI.

### Typography
- 28 font choices: 23 bundled TTF families (46 files) + 5 platform families, all real and distinct.
- Sliders for home label size, drawer label size, section headings, search text, clock, date,
  font weight, line spacing, letter spacing and text scale — each with the numeric value and reset.
- Label visibility toggle, live sample, large-text clipping protection (single-line ellipsis).
- Fonts apply to the launcher only (documented limitation).

### Icons
- 12 genuine masks implemented as real Compose clip paths: circle, rounded square, square,
  squircle (superellipse), teardrop (single sharp corner), hexagon, octagon, diamond, flower,
  pebble, shield, rounded diamond.
- Icon size, spacing, home/drawer column counts, label visibility, optional background plate
  with colour, shape previews, reset.

### Animations
- 8 styles: fade, slide, scale/zoom, spring, crossfade, vertical reveal, parallax, depth.
- Per-surface selection (app drawer, settings screens, search results, theme changes), global
  enable/disable, reduced-motion mode, speed multiplier (0.25×–3×) and a replayable preview.
- Implemented with `AnimatedVisibility`/`AnimatedContent` transitions — no continuous animation,
  no blur, no per-frame work.

### Status bar & edge-to-edge
- `enableEdgeToEdge()` + translucent wallpaper window → transparent status bar on the home screen.
- Status/navigation bar icon contrast chosen from the active theme (or forced light/dark).
- Optional scrim for busy wallpapers, correct insets for cutouts and gesture/3-button navigation.

### App drawer
- Complete installed-app list, search (label *and* package name, accent-insensitive),
  alphabetical or recently-used sorting, adjustable columns/icon size, favourites row,
  recent row, long-press menu (pin/unpin, hide, app info), empty-search state, keyboard "Search"
  action, accessible 48 dp+ touch targets.

### Hide apps
- Dedicated screen with search, multi-select, package-level persistence, reset with confirmation.
- Hidden apps disappear from the drawer, home and search. Flex Launcher **cannot hide itself**,
  so the management screen is always reachable.
- Honest labelling: this is a launcher-level filter, not a vault and not a system-level disable.

### Dial pad
- Numeric dial pad, `+`, backspace, inline validation, and an `ACTION_DIAL` hand-off to the
  system dialer (pre-filled, never auto-dialled). No `CALL_PHONE` permission is requested.

### Settings centre
Categories: Home screen, Recent apps, App drawer, Hide apps, Theme Studio, Fonts & typography,
Icons & shapes, Animations, Status bar, Dial pad, Permissions, Backup & reset, About.
Includes permission status, links to Android settings, JSON backup/restore, reset appearance,
full reset with confirmation, and version/build information.

### Backup & restore
- Export/import the entire configuration (themes, fonts, icons, layout, hidden apps, pins,
  recent history) as JSON through Android's document picker.
- Import validates the file marker and schema version before touching your settings, and rejects
  random JSON files with a visible error.

---

## Screens & navigation

```
Home (launcher surface)
 ├─ Clock/date widget, pinned apps, optional recent row
 ├─ Search/drawer button ──▶ App drawer overlay (recents → favourites → all apps)
 ├─ Dial-pad overlay
 └─ ⚙ Settings
      ├─ Home screen          (default launcher, wallpaper, widgets, pins, haptics)
      ├─ Recent apps          (count, sources, usage access, clear history)
      ├─ App drawer           (sorting, sections)
      ├─ Hide apps            (search, multi-select, reset)
      ├─ Theme Studio ──▶ Custom theme editor
      ├─ Fonts & typography
      ├─ Icons & shapes
      ├─ Animations
      ├─ Status bar
      ├─ Dial pad
      ├─ Permissions
      ├─ Backup & reset
      └─ About
```

Back navigation is handled by `NavHost`; pressing HOME anywhere (including inside settings)
returns to the launcher surface via `MainActivity.onNewIntent`.

---

## Requirements

| Tool | Version used / tested |
|------|----------------------|
| JDK | 17 (Temurin 17.0.20) |
| Gradle | 8.9 (wrapper committed) |
| Android Gradle Plugin | 8.7.3 |
| Kotlin | 2.0.21 |
| Compose BOM | 2024.12.01 (Compose 1.7.x, Material 3 1.3.x) |
| compileSdk / targetSdk | 35 |
| minSdk | 26 (Android 8.0) |
| Build tools | 35.0.0 |

`android-35` platform + build-tools `35.0.0` are required. Everything else is resolved from
`google()` / `mavenCentral()`.

---

## Build locally

```bash
# 1. Point Gradle at your SDK (Android Studio normally does this for you)
echo "sdk.dir=$HOME/Android/Sdk" > local.properties      # macOS: $HOME/Library/Android/sdk

# 2. Unit tests
./gradlew testDebugUnitTest

# 3. Debug APK  → app/build/outputs/apk/debug/app-debug.apk
./gradlew assembleDebug

# 4. Optional: compile the instrumented (on-device) tests
./gradlew assembleDebugAndroidTest
```

`local.properties` is git-ignored and must never be committed.

Low-memory machines (≤ 2 GB RAM) can shrink the Gradle JVM:

```bash
./gradlew assembleDebug -Dorg.gradle.jvmargs="-Xmx850m -XX:MaxMetaspaceSize=560m -XX:+UseSerialGC" --no-daemon --max-workers=1
```

---

## Open in Android Studio

1. **File → Open…** and select the repository root (the folder containing `settings.gradle.kts`).
2. Let Gradle sync; Android Studio will use the committed wrapper (8.9) and its own JDK 17.
3. Select the `app` run configuration and a device (or create an emulator with **API 33–35**).
4. Run. Because Flex Launcher declares `CATEGORY_HOME`, Android will ask whether to use it as your
   home app; choosing "Just once" is handy for testing.

The project uses no annotation processors, no Hilt/KSP and no custom Gradle plugins, so sync is
usually fast and dependency-light.

---

## Install on a phone & set as default launcher

1. Build the APK (locally or from CI) and copy `app-debug.apk` to the phone.
2. Install it (tap the file, or `adb install -r app-debug.apk`). Allow "install unknown apps" for
   your file manager/browser if prompted.
3. Press the Home button — Android shows a chooser: pick **Flex Launcher** and "Always".
   Alternatively: *Settings → Apps → Default apps → Home app → Flex Launcher*.
4. Optional: grant **Usage access** (*Settings → Apps → Special app access → Usage access →
   Flex Launcher*) so Recent apps can order apps opened from any launcher. It is optional; the
   launcher keeps working with its own local history.
5. To go back to your stock launcher: same screen, pick the OEM launcher.

---

## Download the APK from GitHub Actions

1. Push the repository to GitHub (default branch `main` or `master`).
2. Open the **Actions** tab → **Android CI** → the run for your commit.
3. When the job finishes, scroll to **Artifacts** at the bottom of the run summary and download
   **`flex-launcher-debug-apk`** (a zip containing `app-debug.apk`).
4. Unzip and install the APK on your phone.

The workflow also uploads `unit-test-reports` (HTML + XML) every run, and
`gradle-failure-logs` when something fails.

To run it manually: **Actions → Android CI → Run workflow**.

### What the workflow does

`checkout` → JDK 17 (Temurin, with Gradle cache) → Android SDK → validate the committed wrapper →
`testDebugUnitTest` → `assembleDebugAndroidTest` (compiles instrumented tests) → `assembleDebug` →
verify APK exists / non-empty / signed / correct package / HOME category present / permission list →
upload APK artifact → upload logs on failure.

### Release / R8 builds

The `release` build type has R8 enabled (`isMinifyEnabled = true`, `isShrinkResources = true`) with
ProGuard rules for kotlinx.serialization in `app/proguard-rules.pro`. **It was not executed in this
environment**: R8 needs more than the 2 GB of RAM available here and the build was OOM-killed twice.
The supported, verified output is the debug APK (locally and in CI). Build a release on a machine or
runner with ≥ 4 GB RAM (`./gradlew assembleRelease`) before publishing anything.

### Release signing (optional)

The workflow builds a **debug** APK, signed with the throwaway Android debug key — fine for
personal use, not for distribution. For a release build, create your own keystore **outside the
repository** and provide it through GitHub secrets:

```
KEYSTORE_BASE64   # base64 of your .jks
KEYSTORE_PASSWORD
KEY_ALIAS
KEY_PASSWORD
```

Then add a signing config in `app/build.gradle.kts` that reads those values from
`System.getenv(...)`, and a `release` step in the workflow. Never commit a keystore, password or
`keystore.properties` file — `.gitignore` already blocks `*.jks`, `*.keystore` and
`keystore.properties`.

---

## Permissions & privacy

Flex Launcher declares exactly two permissions and no `<queries>`-bypassing flags:

| Permission | Why | When |
|---|---|---|
| `PACKAGE_USAGE_STATS` | Order the Recent apps list from Android's app-open events. Read locally, never uploaded. | Optional; you grant it in system settings. |
| `SET_WALLPAPER` | Only lets the "Change wallpaper" row open the system wallpaper picker. Flex Launcher never writes wallpaper pixels. | No prompt. |

Deliberately **not** requested: `INTERNET`, `QUERY_ALL_PACKAGES`, `CALL_PHONE`, contacts, SMS,
location, camera, microphone, storage, accessibility services, notification access,
`SYSTEM_ALERT_WINDOW`. A unit test (`ManifestTest`) fails if any of these are added.

* `allowBackup="false"` + `data_extraction_rules.xml` exclude preferences from cloud backup;
  the supported backup path is the explicit JSON export you perform yourself.
* No analytics, ads, telemetry or third-party SDKs. No network code at all.
* Recent-app history lives in app-private DataStore files (max 60 entries) and can be cleared
  from *Recent apps → Clear Flex Launcher history*.
* Hidden-app lists store package names only.

---

## Project structure

```
app/src/main/java/com/flex/launcher/
├── FlexLauncherApp.kt         Application + dependency container
├── MainActivity.kt            single HOME activity (edge-to-edge, wallpaper window)
├── core/
│   ├── AppGraph.kt            manual DI (DataStore, repositories, actions)
│   └── util/                  HexColor + ColorMath (WCAG), haptics
├── domain/                    pure Kotlin, unit-testable
│   ├── theme/                 20 presets, palette model, light/dark/AMOLED variants
│   ├── typography/            28 font options, bounded typography settings
│   ├── icons/                 12 shape geometries, icon settings
│   ├── animations/            8 styles + scopes, speed, reduced motion
│   ├── dialer/                ACTION_DIAL intent builder + validation
│   └── model/                 LauncherPrefs, AppEntry, AppQuery, RecentResolver
├── data/
│   ├── prefs/                 DataStore JSON repository, recent history, backup codec
│   ├── apps/                  PackageManager discovery, icon LRU cache
│   ├── recents/               UsageStats wrapper (permission-aware)
│   └── wallpaper/             Palette-based wallpaper colour extraction
├── launcher/LauncherActions   launch intents, dialer, wallpaper, settings deep links
└── ui/
    ├── theme/                 LauncherColors, FlexType, Material 3 bridge
    ├── icons/                 ShapeGeometry → Compose Shape
    ├── animations/            style → Enter/ExitTransition
    ├── components/            icons, search field, settings rows, states, HEX field
    ├── home/                  launcher surface, clock, onboarding
    ├── drawer/                app drawer with recents/favourites/all apps
    ├── dialer/                dial pad
    ├── settings/              13 settings screens + scaffold + routes
    ├── FlexApp.kt             theme resolution, nav graph, snackbar, system bars
    └── MainViewModel.kt       immutable UI state from repositories
```

Design notes: one activity, no fragments, immutable `StateFlow` UI state collected with
`collectAsStateWithLifecycle`, preferences persisted as a single typed JSON document (atomic
writes, trivial backup), and all Android side effects funnelled through `LauncherActions` /
repositories so failures are surfaced instead of swallowed.

---

## Testing status

Everything below was executed in this build environment (headless Linux container, JDK 17,
Android SDK 35). Results are from the actual run; nothing is estimated.

| Test class | Type | Tests | Result |
|---|---|---:|---|
| `HexColorTest` | JVM | 10 | pass |
| `ThemePresetsTest` | JVM | 15 | pass |
| `TypographyTest` | JVM | 12 | pass |
| `IconShapeTest` | JVM | 9 | pass |
| `AnimationTest` | JVM | 9 | pass |
| `AppQueryTest` | JVM | 10 | pass |
| `RecentResolverTest` | JVM | 8 | pass |
| `PreferencesTest` | JVM | 11 | pass |
| `ManifestTest` | JVM | 8 | pass |
| `DialerTest` | Robolectric | 9 | pass |
| `AppRepositoryTest` | Robolectric | 3 | pass |
| `SettingsWidgetsTest` | Robolectric + Compose UI | 5 | pass |
| `LauncherUiTest` | Robolectric + Compose UI | 9 | pass |
| **Total** | | **118** | **0 failures** |

Verified in this environment:

* `./gradlew assembleDebug` → `app/build/outputs/apk/debug/app-debug.apk` (19.3 MB), package
  `com.flex.launcher`, versionCode 1 / versionName 1.0.0, minSdk 26, targetSdk 35, signed with the
  Android debug key (`apksigner verify` passes), `CATEGORY_HOME` present in the packaged manifest,
  and only the two intended permissions listed.
* `./gradlew assembleDebugAndroidTest` → instrumented test APK compiles.
* All 118 unit/UI tests above executed and passed.

Not verified (stated plainly, not implied): **no emulator or physical device was available**, so
the app was never launched at runtime here; `src/androidTest` tests were compiled but not executed;
OEM-specific behaviour (HOME chooser, UsageStats contents, wallpaper transparency, gesture insets)
must be smoke-tested on a real phone. See `docs/TESTING.md` for the full breakdown and the five
real bugs these tests caught.

`docs/TESTING.md` contains the exact commands, per-class results and the list of things that could
not be verified here.

### Prebuilt APK included in this workspace

`dist/flex-launcher-1.0.0-debug.apk` was produced by `./gradlew assembleDebug` in this environment:

* package `com.flex.launcher`, versionName 1.0.0, minSdk 26, targetSdk 35
* signed with the standard Android debug key (`apksigner verify` → OK)
* size 19,504,589 bytes, SHA-256
  `8895d3ac333933f6787d09c4383f128d6bc7a603a6432065b060e52d2ff8f15a`

It is not committed to git (`.gitignore` excludes `*.apk`); rebuild it yourself or download the CI
artifact for a reproducible copy.

---

## Troubleshooting

### The APK builds but Android never offers Flex Launcher as home app
* Check you installed the APK you just built (`adb shell dumpsys package com.flex.launcher | grep -A3 HOME`).
* Some OEMs (MIUI, ColorOS, One UI) hide third-party launchers: open *Settings → Apps → Default
  apps → Home app* and select Flex Launcher there; on MIUI also check *Settings → Home screen →
  System launcher*.
* Make sure you did not disable the app or restrict its background activity after install.

### "Set as default" pill stays visible
The pill reflects `resolveActivity(HOME)`; it disappears once Android actually resolves Flex
Launcher as the default HOME. If you chose "Just once", it stays — that is expected.

### Recent apps is empty or shows fewer apps than expected
* Grant Usage access (the drawer shows an inline link). Without it, only apps you opened from
  Flex Launcher appear — that is the documented fallback, not a bug.
* UsageStats never contains "apps used before the stats window" and some OEM builds return
  coarser data; the resolver degrades to `queryUsageStats` in that case.

### Icons look square / not masked
The mask is applied by Flex Launcher; if an app ships a legacy icon with baked-in padding you
will still see that padding inside the mask. Turn on *Icon background plate* for consistent
shapes with transparent icons.

### Build fails with `SDK location not found`
Create `local.properties` with `sdk.dir=/path/to/Android/Sdk`, or set `ANDROID_HOME`.

### Build fails with `Could not resolve androidx...`
You are offline or a repository is blocked; the project only needs `google()` and `mavenCentral()`.

### Gradle daemon killed / `Out of memory: Killed process`
The build needs roughly 2 GB. Use the low-memory command shown in
[Build locally](#build-locally) (`-Xmx850m --no-daemon --max-workers=1`).

### CI fails at "Run unit tests"
Download the `unit-test-reports` artifact: the HTML report shows the failing test with its stack
trace and the Robolectric `android-all` download logs.

---

## Known Android platform limitations

These are platform constraints, not missing features — the app states them in its own UI:

1. **System Recents is inaccessible.** A launcher cannot read the system Recents screen or the
   list of running tasks. Recent apps uses `UsageStatsManager`, which is approximate and requires
   optional Usage Access.
2. **No system font changes.** A launcher can only style its own UI; `android:fontFamily` for the
   whole OS is not available to third-party apps.
3. **No control over app-open animations.** The system owns the activity transition when another
   app launches. Flex Launcher's animation styles apply to its own surfaces.
4. **Notification shade is off-limits.** Only the launcher's status bar area is styled; the shade
   belongs to SystemUI.
5. **Hiding apps is launcher-local.** It does not uninstall, disable or hide an app from Android
   Settings, other launchers or system search. No vault semantics are claimed.
6. **Third-party icon artwork cannot be redrawn.** Masks can only clip what each app ships
   (adaptive or legacy bitmaps); monochrome/themed icons depend on the app providing that layer.
7. **"Frosted glass" is a visual style.** Real system-level blur behind an app window is not
   available to launchers; the preset imitates it with layered translucency.
8. **Wallpaper transparency depends on the OEM.** The window uses
   `windowShowWallpaper` + translucent theme; some OEM skins render it differently, and Android
   may show its own dim layer behind the status bar.
9. **Usage Access can be revoked at any time.** The launcher re-checks on every resume and falls
   back to local history instead of crashing.
10. **Release/R8 builds are unverified in this environment** (see above); only debug builds were
    built and inspected here.
11. **Hidden/pinned entries can outlive their apps.** Uninstalled packages are ignored when the
    app list is rebuilt and can be cleared with "Reset hidden apps".

---

## Licences

* Application code: provided as-is for personal use (add your preferred licence file when you
  publish the repository).
* Bundled fonts: 23 open-source families under SIL OFL 1.1 / Apache 2.0 — see
  [`docs/FONT-LICENSES.md`](docs/FONT-LICENSES.md).
* AndroidX, Jetpack Compose, Material 3, Kotlin, kotlinx.serialization and Robolectric are used
  under their respective Apache 2.0 licences.
