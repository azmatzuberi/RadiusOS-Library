# Changelog

Notable changes to the RadiusOS content library. Each component is versioned and released on its own by git tag (`field-library-vN`, `fuse-rules-vN`). See [README → Manifests, versions and releases](README.md#manifests-versions-and-releases).

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Field library: working copy at `version` 13 (not yet tagged; manifest still serves v10)

Uploaded in several revisions, 2026-10-16 to 2026-10-30.

- **Added fields:** `neck`, `radiation_dose_and_fluoroscopy_time`.
- **Removed fields:** `complications`, `consent_and_sedation`, `estimated_blood_loss`, `lines_and_tubes`, `peripheral_arteries`, `postsurgical_changes`, `procedure`, `soft_tissues_of_the_neck`.
- 101 → 95 fields; 11,389 → 10,738 terms.
- Removed a duplicate `field_library.json` that had been uploaded to the repo root by mistake.

### Repository

- Added `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `SECURITY.md` and a pull request template.

## [fuse-rules-v1] - 2026-11-11

### Added

- First release of FUSE clinical-insight rules (`fuse-rules/fuse_rules.json`, `version` 1, content stamp `2026.11.06b`). Status: **DRAFT**, pending radiologist review.
- 11 rules: `endometrium`, `ovarian_cyst` (O-RADS US), `lung_rads` (Lung-RADS v2022), `fleischner` (Fleischner 2017), `pancreatic_cyst`, `adrenal_nodule`, `thyroid_incidental_ct_mr` (ACR incidental findings), `ti_rads` (TI-RADS 2017), `renal_cyst_bosniak` (Bosniak v2019), `aaa` (SVS 2018), `gallbladder_polyp` (SRU 2022).
- `shared_context` for `menopause`, `age`, `exam` and `risk`.
- `fuse-rules/fuse_rules.schema.json`: JSON Schema (draft 2020-12) editor aid.
- `manifests/fuse_rules_manifest.json`: `fuse-rules-v1` for RadiusOS 1.8.

## [field-library-v10] - 2026-10-16

### Changed

- Split `thyroid_and_parathyroid` into `thyroid` and `parathyroid_glands` (100 → 101 fields).
- Expanded vocabulary from 2,944 to 11,389 terms.
- Manifest now serves `field-library-v10` for RadiusOS 0.0 – 1.8.

## [field-library-v9] - 2026-10-08

### Added

- First tagged field library (`version` 9): 100 fields, 2,944 terms.
- `manifests/field_library_manifest.json`: `field-library-v9` for RadiusOS 0.0 – 1.8.

### Changed

- Moved the modality folders under `templates/`. Added the merged CT set (`templates/CT/_Merged/`) and the import log (`templates/_organize_report.csv`). Promoted from `dev` to `main` via PR #1.

[Unreleased]: https://github.com/azmatzuberi/RadiusOS-Library/compare/fuse-rules-v1...HEAD
[fuse-rules-v1]: https://github.com/azmatzuberi/RadiusOS-Library/releases/tag/fuse-rules-v1
[field-library-v10]: https://github.com/azmatzuberi/RadiusOS-Library/releases/tag/field-library-v10
[field-library-v9]: https://github.com/azmatzuberi/RadiusOS-Library/releases/tag/field-library-v9
