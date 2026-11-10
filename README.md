# RadiusOS Library

The clinical content library behind **RadiusOS**, RadiusMD's radiology dictation and reporting app. This repository holds data, not code. It has four parts:

| Component | Path | What RadiusOS uses it for |
| --- | --- | --- |
| **Report templates** | [`templates/`](templates/) | Starting structure and default-normal text for each study type, plus per-template instructions for report generation |
| **Field library** | [`field-library/field_library.json`](field-library/field_library.json) | Canonical report sections ("fields"). Maps template headers and dictated terms onto them |
| **FUSE rules** | [`fuse-rules/`](fuse-rules/) | Guideline-based follow-up recommendations, triggered by findings in the dictation |
| **Manifests** | [`manifests/`](manifests/) | Which released version of each component supports which RadiusOS versions |

> [!WARNING]
> **Clinical safety.** The FUSE rules are marked `DRAFT` in [`fuse_rules.json`](fuse-rules/fuse_rules.json) (`_readme.status`). A radiologist must review every threshold and output string before clinical use. The same applies to new or changed recommendation text in templates. Never commit patient data. See [SECURITY.md](SECURITY.md).

---

## Contents

- [Repository layout](#repository-layout)
- [Report templates](#report-templates)
- [Field library](#field-library)
- [FUSE rules](#fuse-rules)
- [Manifests, versions and releases](#manifests-versions-and-releases)
- [Branches](#branches)
- [Validating changes](#validating-changes)
- [Known content issues](#known-content-issues)
- [Contributing, changelog, license](#contributing-changelog-license)

---

## Repository layout

```text
.
├── field-library/
│   └── field_library.json          # 95 fields, ~10.7k terms (working copy, internal version 13)
├── fuse-rules/
│   ├── fuse_rules.json             # 11 guideline rules (DRAFT)
│   └── fuse_rules.schema.json      # JSON Schema (draft 2020-12), editor aid
├── manifests/
│   ├── field_library_manifest.json # release tag -> supported RadiusOS range
│   └── fuse_rules_manifest.json
├── templates/
│   ├── CT/            (120)        # one .txt per template variant
│   │   └── _Merged/   (34)         # consolidated CT templates + _merge_notes.txt
│   ├── DEXA/          (2)
│   ├── Fluoro/        (4)
│   ├── MR/            (35)
│   ├── NM/            (4)
│   ├── Other/         (10)         # blanks, consults, test templates
│   ├── US/            (89)
│   ├── XR/            (95)
│   └── _organize_report.csv        # rename/de-duplication log from the original import
├── LICENSE
└── README.md
```

Counts are template files per folder. The `_Merged` files are counted separately.

---

## Report templates

### File format

Each template is a UTF-8 text file (with BOM, CRLF line endings) with two sections, each starting with a bare header line:

```text
Content

EXAM: CT HEAD [with/without] contrast

HISTORY: []

COMPARISON: No priors.

FINDINGS:

BRAIN: [The ventricles, sulci and gyri are within normal limits. There is no mass effect. There is no midline shift.]

IMPRESSION:

No acute intracranial process by unenhanced technique.  No intracranial bleed.

Prompt

1. Use input findings to change the template data and form an impression.

2. if comparison exam given, enter date in COMPARISON: field.
```

(Shortened from [`templates/CT/_Merged/CT HEAD.txt`](templates/CT/_Merged/CT%20HEAD.txt).)

| Element | Meaning |
| --- | --- |
| `Content` | The report skeleton: exam line, headers (`HISTORY:`, `TECHNIQUE:`, `FINDINGS:`, organ headers, `IMPRESSION:` …) and their text. |
| `[ ... ]` | Placeholder or default-normal text that dictation replaces. `[]` is an empty slot. Options such as `[with/without]` mean "choose one". |
| `Prompt` | Free-text instructions for this template: unit conversions, field placement, ordering of impression vs findings, standard text to include, and so on. |
| `ALT:` lines | Only in `_Merged` files. Alternative wording for the line directly above it. Keep one wording and delete the other. |

Header names in templates don't need to match the field library exactly. The field library's `headerAliases` map common variants (for example `LUNG PARENCHYMA`) onto canonical fields (`lungs`).

### Folders and naming

Templates are grouped by modality folder: `CT`, `DEXA`, `Fluoro`, `MR`, `NM`, `US`, `XR`, and `Other` for blanks, consult letters and test templates. File names start with the modality prefix and study, for example `US THYROID.txt`.

- **Numeric and `v2` suffixes** (`CT ABDOMEN_18.txt`, `CT NECK_2 v2.txt`) come from the original export and mark different versions of the same study. They are not copies: exact duplicates were removed at import, as recorded in `_organize_report.csv`.
- **Parenthesised suffixes** (`(S)`, `(P)`, `(M)`, …) mark site- or reader-specific variants. Per [`_merge_notes.txt`](templates/CT/_Merged/_merge_notes.txt), S and P are site styles.

### Merged CT templates

[`templates/CT/_Merged/`](templates/CT/_Merged/) has one consolidated template per CT study (34 files). Each one was built by folding all variants of that study into the most common plain version.
[`_merge_notes.txt`](templates/CT/_Merged/_merge_notes.txt) explains the conventions:

- Lines that appear only in other plain versions were inserted where they appeared. Conflicting lines become `ALT:` lines.
- Site-specific lines (S/P styles) are grouped in a marked block at the end of `Content`, for example `----- S-SITE STYLE: … -----`. Delete the block if you don't use that style.
- The `EXAM` line uses `[with/without] contrast`. Hard-coded DLP values were replaced with `[]`.
- `Prompt` is the de-duplicated union of all versions' instructions. Conflicts are flagged per file in the notes as `CHECK prompt conflicts`.

The notes also list, per merged file, the base template, the source files, and counts of ALT, added and site-block lines.

### Organize report

[`templates/_organize_report.csv`](templates/_organize_report.csv) records the import pass that produced the current folder layout. It has one row per kept file:

| Column | Meaning |
| --- | --- |
| `Folder` | Modality folder the file was placed in |
| `New name` | File name in this repo |
| `Original name` | File name in the source export |
| `Duplicates removed` | Number of identical files dropped in favour of this one |
| `Duplicate file names` | Their original names |

It is a historical record. Files added or removed since the import are not reflected in it.

---

## Field library

[`field-library/field_library.json`](field-library/field_library.json) defines the canonical report sections RadiusOS understands. It maps the header names and clinical vocabulary found in templates and dictation onto those sections.

```jsonc
{
  "version": 13,
  "fields": {
    "thyroid": {
      "label": "THYROID",
      "headerAliases": ["THYROID", "THYROID GLAND", "THYROID BED", "THYROID UPTAKE"],
      "terms": ["thyroid", "thyroid gland", "..."],
      "defaultNormalPlaceholder": "[Normal in size without discrete nodule.]"
    }
  }
}
```

| Key | Required | Description |
| --- | --- | --- |
| *field id* (object key) | yes | `snake_case` identifier, for example `thyroid` or `spinal_canal_and_cord`. Other fields reference it in `absorbs`. |
| `label` | yes | Upper-case section header written into reports. |
| `headerAliases` | most fields | Alternative template headers that map to this field. |
| `terms` | yes | Lower-case words and phrases (anatomy, findings, exam names) that place dictated content in this field. Include plurals and spelling variants. |
| `defaultNormalPlaceholder` | yes | Bracketed default-normal sentence used when nothing is dictated for the field. |
| `absorbs` | optional | Ids of narrower fields this field subsumes. For example, `brain` absorbs `ventricles_and_csf_spaces`, `pituitary_and_sella` and others. Every id listed must exist as a field. |

The 95 fields cover neuro, head & neck, chest, abdomen, pelvis and OB, spine, MSK, vascular, nuclear medicine, and report-structure sections (`history`, `technique`, `contrast`, `comparison`, `impression`, `recommendation`, `critical_result_communication`, …).

**`version`** is an integer that goes up with each published revision of the file. A release tag `field-library-vN` is cut when the file is at `version: N` (see [releases](#manifests-versions-and-releases)). The working copy on `main` can be ahead of the latest release.

---

## FUSE rules

[`fuse-rules/fuse_rules.json`](fuse-rules/fuse_rules.json) holds RadiusOS's clinical-insight rules. They turn dictated findings into guideline-based follow-up recommendations:

```text
dictation ──▶ trigger phrase heard ──▶ measurement (→ mm) + context ──▶ first matching recommendation ──▶ output text
```

### How a rule is evaluated

These steps come from `_readme.how_it_works` in the file:

1. If any phrase in `triggers` is heard, the rule activates. The longest phrase is matched first (`bosniak 2f` before `bosniak 2`).
2. The size nearest the trigger is extracted and converted to mm (cm × 10). When several lesions are dictated, `measure.use` picks one: `single`, `largest`, `most_suspicious` (ranked by `severity`), or `each_nodule`.
3. Context keys are filled from the dictation, clinical history or order indication. A key can come from `shared_context`, from a named extractor (`"extract": "age"`), or from a map of value → phrases. `scope` is `patient` or `lesion`.
4. `defaults` fills a key when nothing is dictated. `derive` can compute a key from dictated features: `points` scoring (TI-RADS) or `first_match` classification (Bosniak).
5. `recommendations` are walked top to bottom. The first entry whose `when` conditions all match is output. An empty `when: {}` always matches, so it must be the last entry.
6. If a key in `ask_if_missing` can't be found, its prompt is shown instead of guessing.

`requires` and `exclude_if` gate a whole rule on `modality` (taken from the template's scan type) or on a context value. For example, `lung_rads` requires `exam: screening` and `fleischner` excludes it.

### Condition keys in `when`

| Key pattern | Compares against |
| --- | --- |
| `size_gt` / `size_gte` / `size_lt` / `size_lte` | Lesion size in mm |
| `solid_gt` / `solid_gte` / `solid_lt` / `solid_lte` | Solid component in mm (part-solid nodules) |
| `age_gt` / `age_gte` / `age_lt` / `age_lte` | Patient age in years |
| `hu_gt` / `hu_gte` / `hu_lt` / `hu_lte` | Attenuation in Hounsfield units |
| any other key | Must equal the context value, e.g. `"menopause": "post"` |

`modality` is not allowed in `when` or `context`. Use `requires` or `exclude_if` instead.

### Current rules (`fuse-rules-v1`)

| id | Finding | Guideline | Gate | `measure.use` |
| --- | --- | --- | --- | --- |
| `endometrium` | Endometrial stripe thickness | ACOG Committee Opinion 734 (2018); SRU consensus | — | single |
| `ovarian_cyst` | Adnexal / ovarian cyst | ACR O-RADS US v2022 | — | largest |
| `lung_rads` | Lung cancer screening nodule | ACR Lung-RADS v2022 | requires `exam: screening` | most_suspicious |
| `fleischner` | Incidental pulmonary nodule (non-screening) | Fleischner Society 2017 | excludes `exam: screening` | most_suspicious |
| `pancreatic_cyst` | Incidental pancreatic cyst | ACR Incidental Findings Committee 2017 | — | largest |
| `adrenal_nodule` | Incidental adrenal nodule | ACR Incidental Findings Committee 2017 | — | largest |
| `thyroid_incidental_ct_mr` | Incidental thyroid nodule on CT/MRI/PET | ACR Incidental Findings Committee 2015 | excludes `modality: US` | largest |
| `ti_rads` | Thyroid nodule on ultrasound | ACR TI-RADS 2017 | requires `modality: US` | each_nodule |
| `renal_cyst_bosniak` | Renal cystic mass | Bosniak v2019; ACR Incidental Findings 2017 | — | largest |
| `aaa` | Abdominal aortic aneurysm / ectasia | Society for Vascular Surgery 2018 | — | largest |
| `gallbladder_polyp` | Gallbladder polyp | SRU consensus 2022 | — | largest |

Full citations are in each rule's `guideline` field. Rules planned next are listed in `next_rules_to_add`: incidental liver and splenic lesions, thoracic aortic dilatation, LI-RADS, lymph nodes, bone islands, solid renal masses, gallbladder wall and CBD, and possibly BI-RADS.

### Example rule (abbreviated)

```json
{
  "id": "aaa",
  "finding": "Abdominal aortic aneurysm / ectasia",
  "guideline": "Society for Vascular Surgery 2018 practice guidelines",
  "destination": "recommendation",
  "triggers": ["abdominal aortic aneurysm", "aortic aneurysm", "aaa", "aortic ectasia", "ectatic aorta"],
  "measure": { "unit_output": "mm", "use": "largest", "note": "Maximum outer-to-outer diameter." },
  "context": { "sex": { "scope": "patient", "female": ["female", "woman"], "male": ["male", "man"] } },
  "recommendations": [
    { "when": { "size_lt": 25 },                 "output": "Normal caliber abdominal aorta. No follow-up needed." },
    { "when": { "size_gte": 25, "size_lt": 30 }, "output": "Ectatic abdominal aorta. Recommend repeat ultrasound in 10 years." },
    { "when": { "sex": "female", "size_gte": 50 }, "output": "Abdominal aortic aneurysm 5.0 cm or larger in a female patient. Recommend vascular surgery referral for repair consideration." },
    { "when": { "size_gte": 55 },                "output": "Abdominal aortic aneurysm 5.5 cm or larger. Recommend vascular surgery referral for repair." }
  ]
}
```

Rule order matters. The female-specific ≥ 50 mm entry sits above the general ≥ 50 mm entries, so it wins for female patients.

### Schema and validation

- [`fuse_rules.schema.json`](fuse-rules/fuse_rules.schema.json) (JSON Schema draft 2020-12) is an **editor aid**. `fuse_rules.json` points to it via `"$schema"`, so VS Code and other schema-aware editors validate as you type.
- The **authoritative** validator is `validateFuseRules` in the RadiusOS codebase (`doxvox/modules/fuse/fuseRulesConfig.js`). A file can pass the schema and still be rejected there.
- The file carries two version markers. `version` (integer) is the format/release number. `_readme.version` (for example `2026.11.06b`) is the content revision stamp. Bump `_readme.version` on every content edit.
- Rule `id`s are permanent. To remove a rule, delete it and add its id to `retired_rule_ids`. Never reuse an id.

---

## Manifests, versions and releases

Each component is released by a **git tag** named `<component>-vN`. The manifest maps tags to the RadiusOS versions that support them:

```json
{
  "versions": [
    { "fieldLibraryVersion": "field-library-v10", "minRadiusOS": "0.0", "maxRadiusOS": "1.8" }
  ]
}
```

| Component | Manifest | Current release | RadiusOS range |
| --- | --- | --- | --- |
| Field library | [`field_library_manifest.json`](manifests/field_library_manifest.json) | `field-library-v10` | 0.0 – 1.8 |
| FUSE rules | [`fuse_rules_manifest.json`](manifests/fuse_rules_manifest.json) | `fuse-rules-v1` | 1.8 – 1.8 |

The manifests are the source of truth. If this table disagrees with them, the manifests win. The content on a branch tip is not what's released. A release is whatever the tagged commit contains.

**Release flow:**

1. Content changes are committed.
2. The content commit is tagged with an annotated tag `<component>-vN`.
3. A separate commit, `content library - <component> vN release`, updates the manifest.

Step-by-step commands are in [CONTRIBUTING.md](CONTRIBUTING.md#releasing-a-component).

Existing tags: `field-library-v9`, `field-library-v10`, `fuse-rules-v1`. See [CHANGELOG.md](CHANGELOG.md).

---

## Branches

| Branch | Purpose |
| --- | --- |
| `dev` | Integration branch. Content changes land here first. |
| `main` | Default branch. Promoted from `dev` (e.g. PR #1) and carries the release commits and tags. |

The two branches are normally kept at the same commit after each promotion.

---

## Validating changes

Run this from the repo root. It checks that every JSON file parses (Python 3, standard library only):

```bash
python -c "import json,glob; [json.load(open(p, encoding='utf-8')) for p in glob.glob('**/*.json', recursive=True)]; print('all JSON files parse')"
```

For `fuse_rules.json`, also:

- Rely on your editor's schema validation (via `$schema`).
- Run the change through RadiusOS's `validateFuseRules` before release.
- Get radiologist sign-off. See [CONTRIBUTING.md](CONTRIBUTING.md) for the full checklist.

---

## Known content issues

- `field_library.json` on `main` is at internal `version` 13, but the manifest still serves `field-library-v10`. The changes since v10 are unreleased (see [CHANGELOG.md](CHANGELOG.md#unreleased)).
- [`templates/CT/_Merged/CT WHOLE BODY.TXT`](templates/CT/_Merged/CT%20WHOLE%20BODY.TXT) doesn't follow the template format. It has no `Content`/`Prompt` headers and no BOM, uses an upper-case `.TXT` extension, and isn't listed in `_merge_notes.txt`.
- Three other merged files omit the `Content` header line: `CT ABDOMEN.txt`, `CT ABDOMEN PELVIS.txt`, `CT CHEST LOW DOSE LUNG SCREENING.txt`.
- `templates/Other/` contains test templates (`test.txt`, `test-15.txt`, `test-template.txt`).

---

## Contributing, changelog, license

- **Contributing:** [CONTRIBUTING.md](CONTRIBUTING.md) covers the branch workflow, editing rules per component, clinical review, and the release process.
- **Changelog:** [CHANGELOG.md](CHANGELOG.md)
- **Security and patient data:** [SECURITY.md](SECURITY.md)
- **License:** [MIT](LICENSE), © 2026 Azmat. The guideline thresholds in the FUSE rules are summaries of the cited publications. Those publications remain the clinical reference.
