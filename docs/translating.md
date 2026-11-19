# Translating and adding locales

[Documentation index](README.md) · [Contribution rules](../CONTRIBUTING.md)

## Choose Weblate or a repository change

For translation-only work, use [Hosted Weblate](https://hosted.weblate.org/engage/opennutritracker/):
no Flutter installation or Dart knowledge is needed. Request a new language on
the [project page](https://hosted.weblate.org/projects/opennutritracker/).
Coordinate overlapping Weblate and repository edits to avoid conflicting work;
the live Weblate synchronization configuration is outside this checkout.

For a feature adding strings, or a direct translation PR, edit
[`lib/l10n/intl_*.arb`](../lib/l10n/) and follow [CONTRIBUTING](../CONTRIBUTING.md).
The current locales are `en`, `de`, `cs`, `it`, `pl`, `sk`, `tr`, `uk`, and `zh`.
UI translations and backend food-name translations are separate datasets.

## ARB rules

- `intl_en.arb` is the template. Add the same message keys to every locale ARB;
  keep the files valid JSON, with unique keys and no trailing commas.
- Preserve message identifiers and placeholder names. Translate the surrounding
  text, not `{count}` or `{name}`. Keep placeholders compatible with their
  template types; metadata such as `"@key": {"placeholders": ...}` only needs
  to be declared in the English template.
- Preserve ICU plural/select structure and selector values if used; translate
  the displayed clauses and keep the required `other` case. Check interpolation
  in context, not just whether the JSON parses.
- Keep descriptions/context useful to translators and use existing message keys
  where the meaning is truly shared. Avoid composing grammatical sentences by
  concatenating separately translated fragments.

For example, the existing `clearOffCacheSubtitle` metadata declares `count` as
`int` and `size` as `String`. Both placeholders must survive translation, even
if their order changes. Use the template as the contract for any message you edit.

## Generate and check

After the [local setup](../GettingStarted.md), run from the repository root:

```sh
fvm flutter gen-l10n
fvm flutter test test/unit_test/ios_localizations_drift_test.dart
fvm flutter analyze
```

With the pinned SDK on PATH, `just gen_l10n` is equivalent to the first command.
[`l10n.yaml`](../l10n.yaml) configures `S` in `lib/generated/l10n.dart`, with
per-locale generated Dart files and English-first fallback. Review the
`l10n_untranslated.json` report and generation diagnostics. `lib/generated/` is
gitignored: **never edit generated localization code** or run a formatter over
it to maintain translations.

Run the affected widget tests, then view your changed text in the app with its
language selected under Settings. Check narrow screens, long labels, placeholder
values, plural counts, and dialogs. Compilation alone cannot catch clipping or
an incorrect translation.

## Add a locale end to end

1. Create `lib/l10n/intl_CODE.arb` with the template's message keys and translated
   text, and keep locale metadata consistent with the new locale if present.
2. Add the language to [`SupportedLanguage`](../lib/core/utils/supported_language.dart)
   and its locale mapping. This enum is used for food queries, not just labels.
3. Review [`SPConst.translationLocaleOf`](../lib/features/add_meal/data/dto/sp/sp_const.dart).
   Only map a backend translation locale that actually exists. English uses
   `food_summary.name`; unmapped languages can use English fallback rather than
   pretending backend translations exist. Food-name coverage/import tooling is
   maintained in [OpenNutriTracker-Backend](https://github.com/simonoppowa/OpenNutriTracker-Backend).
4. Add a native-language display name/code in `SettingsScreen._supportedLocales`
   in [`settings_screen.dart`](../lib/features/settings/settings_screen.dart), so
   users can select it explicitly. Generated `S.supportedLocales` supplies
   Flutter's supported locale list after generation.
5. **iOS:** add the locale code to `CFBundleLocalizations` in
   [`ios/Runner/Info.plist`](../ios/Runner/Info.plist). The
   [drift test](../test/unit_test/ios_localizations_drift_test.dart) derives locales
   from ARB filenames and fails if one is missing in that list. The Xcode
   project's `knownRegions` currently lists `en` and `Base`; it is not the app's
   complete declared locale list.
6. **Android:** inspect [`AndroidManifest.xml`](../android/app/src/main/AndroidManifest.xml)
   and resources when adding native translated text. The current app declares
   locale/layout-direction configuration handling but has no `android:localeConfig`
   declaration or locale-config XML; do not document an existing native list to
   edit. Flutter's generated locale list and Settings picker handle current UI
   language selection. Adding Android system per-app language declarations
   would be a separate native configuration change, with its own verification.
7. Generate, run the drift test and affected tests, analyze, and check the UI.
   Submit source ARBs and any required hand-written locale wiring, not generated
   output. The iOS drift test checks missing declarations, not translation
   quality, backend coverage, or Android native configuration.
