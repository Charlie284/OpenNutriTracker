# Contributing to OpenNutriTracker

Thanks for taking an interest in contributing! This guide covers the conventions you need to follow so your pull request can be merged smoothly.

For environment setup (Flutter / Android SDK / IDE), see [GettingStarted.md](GettingStarted.md).

Looking for a small first contribution? See the seed list in
[docs/first-timer-backlog.md](docs/first-timer-backlog.md) — concrete
“good first issue” ideas with file paths and acceptance criteria. Agent and
architecture conventions live in [AGENTS.md](AGENTS.md).

## Pull request target branch

**All pull requests must target the `develop` branch, not `main`.**

`main` is reserved for release merges only — it receives a single batched `develop → main` PR per release. If you open a PR against `main`, a maintainer will repoint it to `develop` before review.

## Before you start

- Check the [public board](https://github.com/users/simonoppowa/projects/2) before picking something up — it shows what is already in progress, so you don't duplicate work that is underway.
- Create your branch from the latest `develop`:
  ```sh
  git fetch origin
  git checkout -b feature/<short-name> origin/develop
  ```
- Keep changes scoped. Smaller, focused PRs are easier to review and faster to merge than sweeping ones.
- If you are fixing or implementing an open issue, mention it in the PR description (e.g. `Closes #123`).

## Adding or changing localized strings

Source strings live in `lib/l10n/intl_en.arb`. Translations live in a separate ARB file per supported locale. Flutter generates the Dart files under `lib/generated/` using `l10n.yaml`.

> [!IMPORTANT]
> The files under `lib/generated/` are gitignored generated output. Do **not** edit or format them by hand; run `just gen_l10n` after changing the ARBs.

When adding a new string key in the same PR you must:

1. **Add the key to every ARB file** under `lib/l10n/`. The currently supported locales are:

   | File | Language |
   | --- | --- |
   | `intl_en.arb` | English (source) |
   | `intl_de.arb` | German |
   | `intl_cs.arb` | Czech |
   | `intl_it.arb` | Italian |
   | `intl_pl.arb` | Polish |
   | `intl_sk.arb` | Slovak |
   | `intl_tr.arb` | Turkish |
   | `intl_uk.arb` | Ukrainian |
   | `intl_zh.arb` | Chinese (Simplified) |

   Provide a real translation for each locale — do not leave the English string in as a placeholder. If you only speak one of the languages, machine translation is acceptable as a starting point; native-speaker review is welcome post-merge.

2. **Declare placeholder metadata in the English template** when the string takes parameters.

3. **Run `just gen_l10n`** to regenerate `lib/generated/l10n.dart` and `l10n_<locale>.dart`.

4. **Run `flutter analyze` and the relevant tests** — CI generates localizations before analysis and tests too.

## Code generation

Some files are produced by `build_runner` (Hive type adapters and JSON serialization). Run `just build` after touching any `@HiveType`, `@HiveField`, or `@JsonSerializable` source file. See `AGENTS.md` for the full list of triggers.

## Code style and tests

- 120-character line width (configured in `analysis_options.yaml`).
- Format with `just format` before committing — this targets only `lib/core`, `lib/features`, `lib/l10n`, and `test` and deliberately skips `lib/generated/`.
- Run `flutter analyze` and `just test` locally before opening the PR.
- `just ci` runs the local pre-flight checks (install, format check, localization generation, build, analyze, test). GitHub CI additionally runs native builds, integration tests, and CocoaPods checks; its Linux checks currently skip formatting. Release jobs run separately on `main` pushes or manual dispatch.

## Commit messages

Use a short imperative subject line with a mandatory conventional-commit prefix (`type:` or `type(scope):`). Examples:

```
feat(activity): add high-intensity interval exercise
fix(home): correct kcal budget after onboarding
i18n(activity): wire HIIT codes 02210/02214 to translated strings
```

A body explaining the *why* is welcome but not required for small changes.

## Platform support

OpenNutriTracker ships on both iOS and Android. Any new dependency must support both platforms — check pub.dev before adding. Any platform-specific code must have a corresponding implementation for the other platform (or an explicit fallback). New runtime permissions on Android need a matching `Info.plist` entry on iOS, and vice versa.

## Questions

If you're unsure about anything, open a draft PR or an issue and ask — early feedback is much cheaper than reworking a finished change.
