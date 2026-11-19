# Export bundle format

This document describes the structure of the data export that **Settings →
Export / Import App Data → Export** produces. It exists for the same reason
issue #40 asked for it: people who want to keep their nutrition history in a
plaintext format they can sync via Syncthing, open in a spreadsheet, or feed
into other tooling deserve a stable schema they can read against. Issue #132
asked specifically for CSV export so the import / export round trip would be
symmetric. The format selector offers separate JSON and CSV bundles.

## Zip layout

The export is a single `.zip` file (default filename
`opennutritracker-export.zip`). Select JSON or CSV before exporting or
importing. The JSON bundle contains:

| File                  | Format | Notes                                                              |
| --------------------- | ------ | ------------------------------------------------------------------ |
| `user_intake.json`    | JSON   | Canonical format the app re-imports from.                          |
| `user_activity.json`  | JSON   | Canonical format the app re-imports from.                          |
| `user_tracked_day.json` | JSON | Canonical format the app re-imports from.                          |
| `user_recipes.json`   | JSON   | Recipes only. Nested-ingredient shape, no CSV counterpart.         |
| `weight_log.json`     | JSON   | Weight history.                                                   |
| `custom_activity_templates.json` | JSON | Reusable custom activity templates.                     |
| `recipe_images/`, `meal_images/` | Images | Attached recipe/custom-meal photos, when present.       |

The CSV bundle contains only `user_intake.csv`, `user_activity.csv`, and
`user_tracked_day.csv`: flat tables for the corresponding datasets, without
recipes, photos, weight history, or custom activity templates.

User profile (height, weight, birthday, PAL, goal), settings, custom-meal
definitions, water, and fasting history are **not** included. See
`lib/core/data/data_source/user_data_source.dart` for profile storage.
Logged meals contain embedded food snapshots, but the custom-meal library
itself is not serialized, even though its attached photos are included in JSON exports.

The user can re-import the same zip via **Settings → Import**, selecting the
matching format. CSV is also suitable for spreadsheets or external tooling
without going through Hive.

## JSON schema

The JSON files are direct serializations of the Hive DBO classes via
`json_serializable`, so the source of truth lives in
`lib/core/data/dbo/`. The shapes below are summaries — fields may be added in
future versions, and unknown fields are ignored on import.

### `user_intake.json`

Array of intake records:

```jsonc
[
  {
    "id": "uuid-string",
    "unit": "g",                     // or "ml", "serving", ...
    "amount": 120.0,                 // numeric amount in `unit`
    "type": "breakfast",             // breakfast | lunch | dinner | snack
    "dateTime": "2026-05-13T08:15:00.000",
    "meal": {
      "code": "1234567890123",       // barcode or null
      "name": "Whole Milk",
      "brands": "Acme Dairy",
      "thumbnailImageUrl": null,
      "mainImageUrl": null,
      "url": null,
      "mealQuantity": "100",
      "mealUnit": "g",
      "servingQuantity": 250.0,
      "servingUnit": "ml",
      "servingSize": "250 ml",
      "source": "off",               // unknown | custom | off | fdc | recipe
      "nutriments": {
        "energyKcal100": 61.0,
        "carbohydrates100": 4.8,
        "fat100": 3.3,
        "proteins100": 3.2,
        "sugars100": 5.1,
        "saturatedFat100": 1.9,
        "fiber100": 0.0,
        // Optional extended fields (#237):
        "monounsaturatedFat100": null,
        "polyunsaturatedFat100": null,
        "transFat100": null,
        "cholesterol100": null,
        "sodium100": null,
        "potassium100": null,
        "magnesium100": null,
        "calcium100": null,
        "iron100": null,
        "zinc100": null,
        "phosphorus100": null,
        "vitaminA100": null,
        "vitaminC100": null,
        "vitaminD100": null,
        "vitaminB6100": null,
        "vitaminB12100": null,
        "niacin100": null
      }
    }
  }
]
```

### `user_activity.json`

Array of logged physical-activity records:

```jsonc
[
  {
    "id": "uuid-string",
    "duration": 45.0,                // minutes
    "burnedKcal": 320.0,
    "date": "2026-05-13T18:00:00.000",
    "physicalActivityDBO": {
      "code": "01010",               // 2024 Adult Compendium code
      "specificActivity": "Running, 8 km/h",
      "description": "Running, 8 km/h (7.5 min/km)",
      "mets": 8.3,
      "tags": ["cardio", "outdoor"],
      "type": "running"              // bicycling | conditioningExercise | dancing | running | sport | waterActivities | winterActivities
    }
  }
]
```

### `user_tracked_day.json`

Array of per-day calorie/macro totals:

```jsonc
[
  {
    "day": "2026-05-13T00:00:00.000",
    "calorieGoal": 2200.0,
    "caloriesTracked": 1850.0,
    "carbsGoal": 330.0,
    "carbsTracked": 270.0,
    "fatGoal": 61.0,
    "fatTracked": 55.0,
    "proteinGoal": 110.0,
    "proteinTracked": 95.0
  }
]
```

### `user_recipes.json`

Array of recipe records, each holding an `ingredients` list with embedded
`snapshotMeal` records, `amount`, `unit`, and `convertedAmountG`. The nested-ingredient shape makes CSV a poor fit
here, so recipes ship as JSON only. See `lib/core/data/dbo/recipe_dbo.dart`
for the precise field list.

## CSV schema

All CSV files use UTF-8, comma as the field separator, and `\n` as the line
terminator. The exporter uses these quoting rules (the importer uses
`lib/core/utils/csv_row_parser.dart`, with limitations noted below):

- A cell is wrapped in double quotes when it contains `,`, `"`, `\n`, or `\r`.
- Embedded double quotes inside a quoted cell are escaped by doubling: `""`.
- Empty cells represent `null` for nullable columns.
- Numeric cells accept `.` or `,` as the decimal mark; the exporter always
  writes `.`.
- Date/time cells are ISO-8601 strings (`DateTime.toIso8601String()`).

Column **order** in the exported CSV matches the table below, but the parser
is **header-driven**, so external tooling can re-order columns freely.
Header lookup is case-insensitive.

### `user_intake.csv`

| Column                          | Type   | Required | Notes                                          |
| ------------------------------- | ------ | -------- | ---------------------------------------------- |
| `id`                            | string | yes      | Intake UUID.                                   |
| `date_time`                     | string | yes      | ISO-8601 timestamp.                            |
| `type`                          | enum   | yes      | `breakfast` / `lunch` / `dinner` / `snack`.    |
| `amount`                        | number | yes      | Quantity logged.                               |
| `unit`                          | string | yes      | e.g. `g`, `ml`, `serving`.                     |
| `meal_code`                     | string | no       | Barcode / FDC ID; empty for custom foods.      |
| `meal_name`                     | string | no       |                                                |
| `meal_brands`                   | string | no       |                                                |
| `meal_source`                   | enum   | no       | `unknown` / `custom` / `off` / `fdc` / `recipe`. |
| `meal_quantity`                 | string | no       | e.g. `100`.                                    |
| `meal_unit`                     | string | no       | e.g. `g`.                                      |
| `meal_serving_quantity`         | number | no       |                                                |
| `meal_serving_unit`             | string | no       |                                                |
| `meal_serving_size`             | string | no       | Human-readable size.                           |
| `meal_thumbnail_url`            | string | no       |                                                |
| `meal_main_image_url`           | string | no       |                                                |
| `meal_url`                      | string | no       |                                                |
| `kcal_per_100g`                 | number | no       | All `*_per_100g` columns are nutriment values per 100 g of the food, not per logged portion. |
| `carbs_per_100g`                | number | no       |                                                |
| `fat_per_100g`                  | number | no       |                                                |
| `protein_per_100g`              | number | no       |                                                |
| `sugars_per_100g`               | number | no       |                                                |
| `saturated_fat_per_100g`        | number | no       |                                                |
| `fiber_per_100g`                | number | no       |                                                |
| `monounsaturated_fat_per_100g`  | number | no       | Extended lipid profile (#237).                 |
| `polyunsaturated_fat_per_100g`  | number | no       |                                                |
| `trans_fat_per_100g`            | number | no       |                                                |
| `cholesterol_per_100g`          | number | no       | mg.                                            |
| `sodium_per_100g`               | number | no       | mg.                                            |
| `potassium_per_100g`            | number | no       | mg.                                            |
| `magnesium_per_100g`            | number | no       | mg.                                            |
| `calcium_per_100g`              | number | no       | mg.                                            |
| `iron_per_100g`                 | number | no       | mg.                                            |
| `zinc_per_100g`                 | number | no       | mg.                                            |
| `phosphorus_per_100g`           | number | no       | mg.                                            |
| `vitamin_a_per_100g`            | number | no       | µg RAE.                                        |
| `vitamin_c_per_100g`            | number | no       | mg.                                            |
| `vitamin_d_per_100g`            | number | no       | µg.                                            |
| `vitamin_b6_per_100g`           | number | no       | mg.                                            |
| `vitamin_b12_per_100g`          | number | no       | µg.                                            |
| `niacin_per_100g`               | number | no       | mg (B3).                                       |

### `user_activity.csv`

| Column              | Type   | Required | Notes                                                                  |
| ------------------- | ------ | -------- | ---------------------------------------------------------------------- |
| `id`                | string | yes      | UUID.                                                                  |
| `date`              | string | yes      | ISO-8601 timestamp.                                                    |
| `duration`          | number | yes      | Minutes.                                                               |
| `burned_kcal`       | number | yes      |                                                                        |
| `activity_code`     | string | yes      | 2024 Adult Compendium code.                                            |
| `specific_activity` | string | yes      | Human-readable activity name.                                          |
| `description`       | string | yes      |                                                                        |
| `mets`              | number | yes      | MET value.                                                             |
| `tags`              | string | no       | Pipe-separated (`outdoor|cardio`) so the cell stays single-field.      |
| `type`              | enum   | yes      | `bicycling` / `conditioningExercise` / `dancing` / `running` / `sport` / `waterActivities` / `winterActivities`. |

### `user_tracked_day.csv`

| Column              | Type   | Required | Notes                                |
| ------------------- | ------ | -------- | ------------------------------------ |
| `day`               | string | yes      | ISO-8601 date (start of day).        |
| `calorie_goal`      | number | yes      |                                      |
| `calories_tracked`  | number | yes      |                                      |
| `carbs_goal`        | number | no       | g.                                   |
| `carbs_tracked`     | number | no       | g.                                   |
| `fat_goal`          | number | no       | g.                                   |
| `fat_tracked`       | number | no       | g.                                   |
| `protein_goal`      | number | no       | g.                                   |
| `protein_tracked`   | number | no       | g.                                   |

## Round-trip limitations

`lib/core/utils/csv_data_exporter.dart` exports and parses the columns listed
above. CSV does not preserve every DBO field: for example, activity `userKcal`
and newer meal metadata are not represented. The parser trims cell whitespace
and normalizes empty nullable cells to null. It splits physical lines before
parsing cells, so embedded newlines do not round-trip, even in quoted cells.
`test/unit_test/csv_data_exporter_test.dart` covers supported round-trip cases.

The in-app **Import** action accepts JSON or CSV zips according to the selected
format. JSON preserves more fields and includes the additional datasets and
photos listed above, but neither format is a complete backup of all app data.

## Syncthing-friendly notes

The data tables are plain UTF-8 inside a normal zip; JSON bundles can also
contain binary image files. You can keep a copy in a Syncthing folder and
move it between devices, but this is a manual export/import workflow, not
live database synchronization. Exports are not encrypted, so protect the zip
as personal data. There is no schema-version field enforcing compatibility
across future versions; keep the round-trip limitations above in mind.
