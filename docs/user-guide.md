# Using OpenNutriTracker

[Documentation index](README.md) · [Features and screenshots](../README.md#key-features)

## Start with your own profile, or try the app

On first launch, follow onboarding to enter your body measurements, activity
level, and weight goal. Review the privacy information before completing setup.
These inputs produce estimates, not medical advice; see the
[disclaimer](../README.md#disclaimer) and in-app Sources & References.

To explore first, choose **Try it with sample data** on the introduction page.
This fills the active profile with three weeks of example tracking and opens
Home. A banner identifies sample mode on every tab. Choose **Set up your profile**
and confirm when ready: the sample profile's personal tracking data is cleared
and onboarding starts again. Do not treat sample data as your own history.

## Find your way around

| Tab | Use it for |
| --- | --- |
| **Home** | Today's energy/macros, meal and activity sections, quick weight/water entry, and fasting entry point |
| **Diary** | Selecting a date and reviewing or changing that day's logs |
| **Trends** | Reviewing intake, weight, and activity over time |
| **You** | Body measurements, goals, profile switching, and recipes |

Use the central **+** action to add food or activity. Settings are available from
the main screen's settings control. Labels below describe the English UI;
translations may differ.

## Log a meal

1. For a past day, select it in Diary first; check the destination day and meal
   section when adding food.
2. Open the food picker from **+** or the meal section's add control. Search
   packaged products or reference foods, use a saved/custom meal or recipe, or
   open the barcode scanner.
3. Open the result, check nutrition and serving units, enter the quantity, and
   add it to the selected meal section. Database values can be incomplete or
   wrong: compare them with the package label when accuracy matters.
4. Tap a logged meal to change its amount. Use its delete action to remove it;
   the deletion dialog asks for confirmation.

To correct a reusable food definition, use the food's edit/custom-meal flow,
rather than only changing a logged portion. A custom meal can have a brand,
barcode, nutrition values, and photo; scanning a saved custom barcode prefers
your own entry over remote products.

### Recipes versus custom meals

- **Custom meal:** a reusable food with nutrition values you enter yourself.
  Useful for a missing packaged food or a food you want to scan again.
- **Recipe:** a composition of ingredients with quantities and a serving count.
  Open **You → Recipes** to build one, then log the amount/servings you ate.
  Recipes can have photos and tags, but are not barcode-bearing products.

Both libraries are shared across local profiles. Logged entries retain meal
snapshots; do not assume changing a library item rewrites all past logs.

## Log or adjust an activity

Open **+ → Activity**, choose a catalog activity and enter its duration, or use
the custom-activity option for your own activity/calorie entry. Saved templates
let you reuse custom activities. Tap a logged activity to edit its duration;
use its delete action to remove it. Activity energy contributes to the day's
calorie allowance, so avoid double-counting the same exercise.

## Use profiles on one device

Tap the profile header on **You** to switch profiles, add one, or open
**Manage profiles**. Management offers editing and deletion; the final remaining
profile cannot be deleted there. A new profile needs its own setup.

Profiles separate body data, goals, diary, weight, water, and fasting history.
They share recipes, custom foods, activity templates, cached foods, and app-wide
preferences such as language/theme/units. They are local profiles, not online
accounts or a password-protected access boundary. Nothing here syncs a diary
between devices. See [storage boundaries](architecture.md#storage-boundaries).

## Tune goals and your tracking day

In **You**, update body measurements, activity level, weight goal, weekly rate,
and optional target weight. Settings provides calorie adjustment and macro
targets; the calorie-goal explanation shows how the estimate is assembled.
An optional taper reduces weight-goal adjustment near the target.

In **Settings → Day starts at**, choose the hour and minute when your tracking day
begins. For example, with a 04:00 boundary, a 02:00 timestamp belongs to the
previous tracking day. Changing this setting changes how timestamps are grouped;
it does not move the recorded clock times. If entries appear on an unexpected
date, check this preference before logging duplicates.

## Track water or a fasting session

- **Water:** use Home's water controls to add an amount or undo the latest
  entry; set your daily goal on You. Amounts use your chosen display units.
- **Fasting:** open the fasting timer from Home, read and acknowledge its safety
  warning, choose a preset or custom duration, and start a session. The screen
  offers cancellation while active and a completion screen when done. A timer is not medical advice;
  follow the warning, and do not use it as a substitute for professional care.

## Export, import, and protect your data

Open **Settings → Export / Import App Data**, choose JSON or CSV, and export a
zip file. Store it somewhere you control, outside the app's private storage.
To import, select the matching format and choose an existing export zip.
Import adds/merges supported records; it is not a reset-and-restore operation.

**Prefer JSON for portability**, but neither format is a complete app backup:
profile/settings, custom-meal definitions, water, and fasting history are not
included. CSV has fewer datasets and additional fidelity limits. Read
[Export bundle format](export-format.md) for exact contents and restrictions
before deleting data or moving devices. Export separately from each profile
whose personal logs you need; shared recipes/templates can be included again.

Export zips are **not encrypted**. Anyone with a copy can read personal records.
Keep private backups securely; do not attach one to a public bug report.

## Privacy and deletion

Tracking records are stored locally in encrypted Hive boxes. Food searches go
to Open Food Facts or the configured Supabase reference-food service; food
images can load from remote image hosts. Optional crash reporting is consent
controlled. The [README privacy section](../README.md#privacy) explains these
boundaries and links the privacy policy. Local attached image files and exported
zips are not protected by Hive's box encryption.

Settings' delete-all-data action clears the **active profile's personal data**,
not every profile or shared library. Profile management is a separate operation.
Export supported data first; an export cannot recover omitted datasets.

## Troubleshooting

| Problem | What to try |
| --- | --- |
| Search is empty or unavailable | Check connectivity and Settings' food-database selection; try another term/source or create a custom meal. Cached foods and local libraries can still help offline, but uncached remote results require a connection. |
| A barcode has no result | Enter it manually if the camera cannot read it, search by name, or save a custom meal with that barcode. |
| History seems missing | Check the active profile, selected Diary date, and day-start setting before importing or adding duplicates. |
| Calorie allowance seems surprising | Check body measurements, activity level, weekly goal, calorie adjustment, and logged activity; consult the goal explanation. |
| Import fails or is partial | Match JSON/CSV format to the zip, keep the original, and review the export-format limitations. Do not repeatedly import an unknown file hoping to repair it. |
| “Your local data could not be opened safely” | Use **Retry**. Record the error code and contact support if it persists. Do not clear app data or reinstall before asking for help: the encryption key may be needed to read existing files. |

For unresolved issues, use the [contribution/issue guidance](../CONTRIBUTING.md)
and include app version, platform, steps, and error text without private records.
