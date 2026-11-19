# Documentation

Choose a starting point by what you want to do. The [project README](../README.md)
contains installation links, screenshots, feature highlights, privacy information,
and contact details.

## Use the app

- [User guide](user-guide.md): first launch, logging, recipes, local profiles,
  goals, water/fasting, backups, and troubleshooting.
- [Export bundle format](export-format.md): exact JSON/CSV contents, import
  behavior, and portability limits. Exports are not full installation backups.
- [Privacy and policy links](../README.md#privacy) and [disclaimer](../README.md#disclaimer).
- [Project website](https://simonoppowa.github.io/OpenNutriTracker/): public landing page; its source
  is [`docs/site/`](site/), not a second app documentation tree.

## Build or contribute

- [Getting started](../GettingStarted.md): pinned SDK, environment, generation,
  and running a local build.
- [Contributing](../CONTRIBUTING.md): issues, PRs against `develop`, required
  commit conventions, and contribution workflow.
- [Architecture](architecture.md): modules, startup/recovery, state, profiles,
  food lookup, calculations, and per-feature code map.
- [Testing](testing.md): test layers, helpers, devices, ADB links, and CI parity.
- [Translating](translating.md): Weblate, ARBs, generation, and adding locales.
- [First-timer backlog](first-timer-backlog.md): scoped contribution ideas;
  check current issues before taking one on.
- [Code of Conduct](../CODE_OF_CONDUCT.md) and
  [PR template](../.github/PULL_REQUEST_TEMPLATE.md).

## Self-host food reference data

- [Supabase self-hosting and app wiring](supabase-self-hosting.md): app/backend
  contract and local build configuration.
- [Legacy FDC-only guide](supabase-fdc-self-hosting.md): historical setup context;
  prefer the current multi-source guide above for new work.
- [OpenNutriTracker-Backend](https://github.com/simonoppowa/OpenNutriTracker-Backend):
  canonical backend installation, schema, ingestion, and translation tooling.
  Hosting reference foods does **not** add personal diary synchronization.

## Maintain or automate

- [Release runbook](../RELEASE.md): signing, job gates, artifacts, store uploads,
  failure recovery, and website release metadata.
- [iOS Fastlane lane reference](../ios/fastlane/README.md).
- [AGENTS.md](../AGENTS.md): canonical AI-agent policy, code-generation rules,
  accessibility identifiers, ADB tooling, and coding conventions.
  [CLAUDE.md](../CLAUDE.md) and [Copilot instructions](../.github/copilot-instructions.md)
  delegate to it.
- [CI workflows](../.github/workflows/), [justfile](../justfile), and
  [launch-image asset note](../ios/Runner/Assets.xcassets/LaunchImage.imageset/README.md)
  are implementation/maintenance references.

## Keep these guides accurate

When behavior changes, update the relevant guide in the same PR. Verify commands,
paths, settings, export contents, and platform declarations against code; link
canonical material instead of copying it. Keep backend operations upstream.
For architecture changes, update Mermaid diagrams as well as prose. Never
document secret values or edit generated files to change behavior.
