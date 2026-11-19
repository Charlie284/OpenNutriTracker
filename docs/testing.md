# Testing

[Documentation index](README.md) · [Local setup](../GettingStarted.md) · [Contribution rules](../CONTRIBUTING.md)

## Prepare a checkout

Use the SDK pinned by `.fvmrc` and the environment/code-generation steps in
[GettingStarted](../GettingStarted.md). From the repository root:

```sh
fvm flutter pub get
fvm flutter gen-l10n
fvm dart run build_runner build
```

Generation needs `.env` keys as described in setup. Do not put real credentials
in tests or logs. Unit/widget tests normally replace data dependencies rather
than calling live services. `lib/generated/` and `env.g.dart` are generated, not
hand-maintained test fixtures.

## Choose the smallest useful test

| Layer | Location / example | Device needed? |
| --- | --- | --- |
| Unit | `test/unit_test/tdee_calc_test.dart`, `test/features/add_meal/data/repository/` | No |
| Widget | `test/widget_test/`, widget tests under `test/features/` | No; Flutter's test runner renders a test surface |
| Integration | `integration_test/app_boot_test.dart` | Yes: compatible Android emulator/device or iOS simulator/device |

```sh
# All unit and widget tests under test/
fvm flutter test

# Single calculation test file
fvm flutter test test/unit_test/tdee_calc_test.dart

# Single widget test file, or selected test name
fvm flutter test test/widget_test/storage_recovery_app_test.dart
fvm flutter test test/unit_test/tdee_calc_test.dart --plain-name 'IOM TDEE calculation for a male user'

# Analyze the app separately
fvm flutter analyze
```

`--plain-name` is a name filter: replace the example string with text from the
test you want. Tests under `test/features/` include unit and widget tests; folder
placement alone does not tell you which harness a test needs.

## Helpers and mocks

- [`hive_test_setup.dart`](../test/helpers/hive_test_setup.dart) guards Hive
  adapter registration across tests. Follow neighboring tests' temporary-box
  setup and teardown to keep disk state isolated.
- [`FakeHiveDBProvider`](../test/helpers/fake_hive_db_provider.dart) injects fake
  boxes instead of opening application storage.
- [`test_l10n.dart`](../test/helpers/test_l10n.dart) exposes English strings for
  assertions; localized widgets still need localization delegates in their app
  wrapper. Reuse the wrappers in nearby widget tests.
- [`off_data_source_fallback_test.dart`](../test/unit_test/off_data_source_fallback_test.dart)
  demonstrates `http/testing.dart`'s `MockClient` and
  `PackageInfo.setMockInitialValues()` for deterministic HTTP/User-Agent tests.
- [`ios_localizations_drift_test.dart`](../test/unit_test/ios_localizations_drift_test.dart)
  reads ARBs and the plist directly and needs no iOS device; see [Translating](translating.md).

For new behavior cover normal, empty, and error cases at the closest layer.
Avoid real API requests and dependence on another test's Hive contents. For UI
tests, provide enough viewport space and pump async changes intentionally;
copy the tested feature's existing Bloc/provider setup rather than booting the
whole app unnecessarily.

## Integration tests and devices

The boot test starts the real `main()` and checks clean boot, `MaterialApp`,
onboarding on a fresh install, and localization rendering. It uses
`IntegrationTestWidgetsFlutterBinding`, not a mocked repository harness.

```sh
fvm flutter devices
fvm flutter test integration_test/app_boot_test.dart --flavor full -d DEVICE_ID
```

Replace `DEVICE_ID` with an identifier from `flutter devices`. Android needs
the Android SDK and a running emulator or attached device. iOS requires macOS,
Xcode, CocoaPods, and a simulator or a provisioned physical device. A fresh app
state matters for first-run assertions; use a disposable device/profile rather
than clearing real tracking history. Supabase is initialized during boot even
when a test does not search foods, so setup still needs a syntactically valid URL.

For scripted Android checks, use the [AGENTS ADB tooling guide](../AGENTS.md#adb-test-tooling)
and [accessibility identifier rules](../AGENTS.md#accessibility-identifiers-for-interactive-widgets).
`tools/adb/adb-driver.sh` supplies stable-ID and form helpers;
`walk-onboarding.sh` clears app data before walking setup.
`run-branch-tests.sh` is a broader branch smoke runner, not a replacement for
Flutter tests. Read its destructive/setup behavior before pointing it at a device.

## Local checks versus GitHub CI

`just test` runs `flutter test`. `just ci` installs dependencies, checks formatting,
generates localizations and build-runner output, runs tests, and analyzes.
Its Flutter/Dart commands require the intended SDK on PATH; use the explicit
`fvm` forms above if that is not configured.

The source of truth for automated gates is
[`default_workflow.yml`](../.github/workflows/default_workflow.yml):

| Job | Additional coverage |
| --- | --- |
| `linux-checks` | Dependency install, localization/code generation, analysis, unit/widget tests, iOS pod-install fallback parser test; currently deliberately skips formatting because of existing format drift |
| `ios-podfile-lock-guard` | PR-only guard for relevant iOS dependency changes |
| `ios-build`, `android-build` | Native compile checks, beyond Dart analysis |
| `ios-integration-tests`, `android-integration-tests` | Full-flavor boot tests on simulator/emulator |

The workflow pins SDK/toolchain setup and supplies stub environment values in
check jobs; do not mistake these for a working food backend. Native builds and
device tests are not covered by a successful `just ci`. Release packaging and
deployment are separate gates described in the [release runbook](../RELEASE.md).
