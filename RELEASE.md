# Release notes for maintainers

This runbook is for maintainers, not app installation. The executable references
are [the build/release workflow](.github/workflows/default_workflow.yml),
[iOS Fastfile](ios/fastlane/Fastfile), and
[Android Fastfile](android/fastlane/Fastfile). See the [documentation index](docs/README.md)
for contributor and user guides.

## 1. Prepare the release

1. Land reviewed work on `develop`; contributor PRs target that branch. Prepare
   the release merge to `main`, which triggers packaging/deployment after checks.
2. Set `version: MAJOR.MINOR.PATCH+BUILD` in `pubspec.yaml`. Both platforms take
   the build number from `+BUILD`; choose a new build number accepted by both
   stores. The GitHub job derives tag `vMAJOR.MINOR.PATCH` and release name
   `Release vMAJOR.MINOR.PATCH (build BUILD)`; the build number is not in the tag.
   Choose the version with existing tags/store uploads in mind before merging.
3. Run the relevant [tests and native checks](docs/testing.md). Review generated
   output when source changes require it, but do not hand-edit generated files.
4. Confirm access to signing material and store accounts, and review release
   notes/changes before the `main` merge. Do not use workflow dispatch casually:
   it also enables real packaging and store deployment, not just a dry run.

The workflow runs on pushes to `main`, PRs to `develop`/`main`, and manual
dispatch; direct pushes to `develop` do not trigger it. Package/deploy/release gates allow a push to `main` **or** manual
dispatch. Manual dispatch uses the selected ref: verify it before starting.

## 2. Signing and service prerequisites

Use the Flutter version from `.fvmrc` and CI's pinned setup. iOS packaging needs
macOS, Xcode, Ruby/Bundler, CocoaPods, FVM, and the iOS engine artifacts; the
workflow explicitly precaches iOS before installing pods. Android needs its
SDK/Java toolchain and release keystore. Follow the workflow's setup rather
than substituting a develop-flavor build for a store artifact.

Configure these GitHub secret **names** through the repository's secure settings;
never commit or print values:

| Purpose | Names |
| --- | --- |
| Compiled app environment | `FDC_API_KEY`, `SENTRY_DNS`, `SUPABASE_PROJECT_URL`, `SUPABASE_PROJECT_ANON_KEY` |
| iOS match repository/signing | `MATCH_GIT_URL`, `MATCH_GIT_SSH_PRIVATE_KEY`, `MATCH_PASSWORD` |
| App Store Connect API | `APP_STORE_CONNECT_API_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`, `APP_STORE_CONNECT_API_KEY_P8` |
| Android signing | `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD` |
| Google Play upload | `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON` |

iOS uses match `appstore` profiles. `MATCH_READONLY` defaults to `true`; ensure
certificates/profiles already exist rather than silently creating new signing
assets in a release job. SSH-backed match repositories use the private-key
secret. The Fastfile expects base64 key content by default through
`APP_STORE_CONNECT_API_KEY_IS_KEY_CONTENT_BASE64=true` (an environment switch,
not another secret). The `full` scheme uses production bundle identity and
manual Apple Distribution signing. Android CI decodes the keystore and writes
`key.properties` only in the runner workspace.

## 3. Follow the job gates

| Stage / jobs | Gate and result |
| --- | --- |
| `linux-checks` | Generate, analyze, unit/widget tests, pod-fallback parser test; currently skips format check |
| `ios-podfile-lock-guard` | PR-only iOS dependency lock guard |
| `ios-build`, `android-build` | Native compilation checks |
| `ios-integration-tests`, `android-integration-tests` | Full-flavor boot tests on simulator/emulator |
| `ios-package`, `android-package` | Need Linux checks plus both platforms' build/integration jobs; produce signed artifacts |
| `ios-deploy`, `android-deploy` | Each needs both package jobs, avoiding deployment when one platform failed packaging |
| `github-release` | Needs both deploy jobs; downloads artifacts and publishes a version-derived GitHub release |

The cross-platform packaging gate does not make store uploads transactional:
one deploy can still succeed while the other fails.

## 4. Identify artifacts and destinations

| Artifact | Runner output | Destination |
| --- | --- | --- |
| `ios-ipa` | `build/ios/ipa/*.ipa` | TestFlight; GitHub asset renamed to `opennutritracker.ipa` |
| `android-aab` | `build/app/outputs/bundle/fullRelease/app-release.aab` | Google Play **internal** track; GitHub `app-release.aab` |
| `android-apk` | `build/app/outputs/apk/full/release/app-release.apk` | GitHub `app-release.apk` for sideloading |

All release builds use flavor `full`. The APK upload uses Gradle's renamed output,
not Flutter's separate `app-full-release.apk` copy.

iOS CI runs `bundle exec fastlane ios build` in `ios/`, then uploads the downloaded
IPA via `bundle exec fastlane ios beta_from_ipa`. Upload skips waiting for
TestFlight processing: a green job is not proof the build finished processing
or is available to every tester. See the [lane reference](ios/fastlane/README.md).
`ios beta` builds then uploads; `ios release` builds and uploads to App Store
Connect with `submit_for_review: false` and `automatic_release: false`. The
workflow does not invoke that production lane.

Android CI runs `bundle exec fastlane android internal` in `android/`. It uploads
the AAB with completed release status on the **internal testing** track, skipping
metadata/images/screenshots. `ANDROID_AAB_PATH` can override its default local
artifact path. No production-track promotion is performed by this lane.

After uploads, check store processing/status separately. GitHub release notes
are generated by the action; inspect the published assets and version label.

## 5. Recover a failed release

- **Checks fail:** repair the failing source/tooling on the normal branch flow;
  do not bypass the gate or publish unsigned/develop artifacts as release files.
- **Signing fails:** inspect missing/expired match profiles, API access, keystore
  configuration, and secret names. Never echo keys or commit runner signing files.
  `bundle exec fastlane ios certs` syncs signing when run from `ios/`; it retains
  the same read-only default.
- **Packaging succeeds but deployment fails:** retain the run's signed artifacts
  and inspect store state before retrying. Re-running failed jobs may be enough
  for a transient service error; a whole workflow dispatch can rebuild/re-upload
  both platforms and is not an idempotence guarantee.
- **Only one store accepted a build:** do not assume rollback occurred. Resolve
  the other upload explicitly; if a new binary is required, increment the build
  number before building again. Avoid duplicating a store version/build pair.
- **Reusing an IPA locally:** `ios beta_from_ipa` finds the newest `.ipa` under
  `build/ios/ipa/`. Put only the intended signed artifact there and verify its
  identity before upload; the lane does not rebuild it.
- **GitHub release or metadata fails:** check the tag and existing assets before
  retrying the publish job. A manual dispatch is a full release-capable run, not
  a metadata-only repair. The fingerprint workflow can be dispatched separately
  for an existing tag as described below.

## 6. Update public verification metadata

Review the fingerprint workflow's PR against `develop` and include its website
metadata change in `main`. [Deploy site](.github/workflows/deploy-site.yml) runs
for `docs/site/**` (or its own workflow) changes on `main`, or manual dispatch.
Verify the published download links and metadata after deployment; updating
`develop` alone does not publish the site.

## Generating the SHA256 fingerprint

When publishing a new APK to GitHub Releases (and in future to F-Droid), a security-conscious person installing the build may want to verify that it was signed by the same release key as every other official build. The README has a `Verifying APK signatures` section that points them at the canonical SHA256 fingerprint of the release certificate — this section is the reference for keeping that value accurate.

The fingerprint comes straight from the release keystore. With the keystore path and alias from `android/key.properties`, run:

```sh
keytool -list -v -keystore /path/to/release.keystore -alias <alias>
```

Look for the `SHA256:` line under `Certificate fingerprints`. The value is stable for the lifetime of the keystore — it only changes if the signing key itself is rotated, which would also force every existing installation to uninstall before upgrading — so once the README has the right value, it should not need to be touched again on routine releases.

If the key is ever rotated, update the fingerprint in `README.md` alongside the signing configuration change, so that anyone verifying a freshly downloaded APK against the README sees the new value as soon as the new build is published. Never commit the private keystore.

The website also reads the signing fingerprint and per-asset hashes from `docs/site/release-info.json`. `.github/workflows/update-release-fingerprint.yml` extracts these from published APK/IPA assets and opens a PR against `develop`; it can also be dispatched manually for an existing tag. Review and merge that update, then include it in `main` for the site deployment. Key rotation must keep both the README fingerprint and this website metadata in sync with the released APK.
