# Contributing to RadiusOS Library

This repository is clinical content that RadiusOS reads at runtime. A typo in a template or a wrong threshold in a FUSE rule goes straight into a radiology report. Please read this guide before changing anything.

- [Ground rules](#ground-rules)
- [Branch workflow](#branch-workflow)
- [Editing templates](#editing-templates)
- [Editing the field library](#editing-the-field-library)
- [Editing FUSE rules](#editing-fuse-rules)
- [Releasing a component](#releasing-a-component)
- [Commit messages](#commit-messages)
- [Pull request checklist](#pull-request-checklist)

---

## Ground rules

1. **No patient data, ever.** Templates, prompts and test files must not contain names, dates of birth, MRNs, accession numbers, real exam dates or any other PHI. See [SECURITY.md](SECURITY.md).
2. **Clinical changes need a radiologist.** Any change to recommendation text, follow-up intervals, size thresholds or default-normal wording needs sign-off from a radiologist before release.
3. **Releases are tags, not branch tips.** RadiusOS uses what a release tag points at, as listed in the manifests. Merging to `main` alone does not change what RadiusOS serves.

---

## Branch workflow

| Branch | Use |
| --- | --- |
| `dev` | Integration branch. Target it with your changes, either by direct commit for small edits or a feature branch plus PR. |
| `main` | Default and release branch. `dev` is promoted to `main` once changes are reviewed. |

Promoting `dev` to `main`: when `main` has no commits that `dev` lacks, fast-forward it so both branches point at the same commit:

```bash
git fetch origin
git checkout main
git merge --ff-only origin/dev
git push origin main
```

If `main` has diverged (for example a direct upload through the GitHub web UI landed on `main`), open a PR from `dev` to `main` instead, then fast-forward `dev` to the merged `main`.

> Uploading through the GitHub web UI (**Add file → Upload files**) is fine for templates and the field library. Pick `dev` as the target branch and replace the existing file at the **same path**. A file uploaded at the repo root (e.g. `field_library.json` instead of `field-library/field_library.json`) creates a second copy rather than updating the real one.

---

## Editing templates

Templates live in `templates/<Modality>/`. The format is described in [README → Report templates](README.md#report-templates).

- Keep the two-section structure: a `Content` header line, the report skeleton, a `Prompt` header line, then the instructions.
- Use `[ ... ]` for text that dictation replaces and `[]` for an empty slot. Keep default-normal text inside brackets.
- Put new templates in the matching modality folder and start the file name with the modality prefix (`CT`, `CTA`, `MR`, `US`, `XR`, `NM`, `DEXA`, `FLUORO`), followed by the study name. Use `.txt` (lower case).
- Save as UTF-8. Existing files are UTF-8 with BOM and CRLF line endings, and most editors keep that when you edit in place.
- Changing a template in `templates/CT/_Merged/`? Resolve `ALT:` lines rather than adding new ones. If you re-merge, update the file's entry in `_merge_notes.txt`.
- Section headers should map to a field in the field library, either by `label` or one of its `headerAliases`. If you introduce a new header wording, add it as an alias (see below).
- `templates/_organize_report.csv` is a historical import log. Don't edit it to reflect new files.

---

## Editing the field library

File: `field-library/field_library.json`. The schema is described in [README → Field library](README.md#field-library).

- **Field ids** are `snake_case` and stable. RadiusOS and other fields' `absorbs` lists refer to them. Renaming or removing a field is a breaking change, so record it in the changelog.
- **`label`** is upper case. **`terms`** are lower case and should include plurals and common spelling variants. Before adding a term, check whether another field already lists it, so the same phrase doesn't map to two sections by accident.
- **`headerAliases`** hold alternative header wordings found in templates (upper case).
- **`defaultNormalPlaceholder`** must be a single bracketed sentence: `[ ... ]`.
- **`absorbs`** may only list ids that exist as fields.
- **Bump `version`** (integer) once per published revision. A release `field-library-vN` is cut when the file is at `version: N`.
- Keep the file as valid, 2-space-indented JSON.

---

## Editing FUSE rules

File: `fuse-rules/fuse_rules.json`, described in [README → FUSE rules](README.md#fuse-rules). Open it in an editor that honours `$schema` (VS Code does) and fix any schema warnings before committing.

Rules enforced by the schema or the evaluator:

- `id` matches `^[a-z0-9_]+$` and is unique. **Ids are permanent.** To remove a rule, delete it and append its id to `retired_rule_ids`. Never reuse a retired id.
- Each rule needs `id`, `finding`, `guideline`, `triggers` (at least 1) and `recommendations` (at least 1).
- `recommendations` are evaluated top to bottom and the first match wins. Put specific conditions above general ones. An empty `when: {}` is the catch-all and must be the **last** entry.
- If `measure.use` is `most_suspicious`, every recommendation needs an integer `severity` (higher = worse).
- `output` strings must not contain `{` or `}`.
- `modality` is not a valid key in `when`, `context` or `shared_context`. Gate on modality with `requires` / `exclude_if`.
- Numeric conditions use `size_`, `solid_`, `age_` or `hu_` with `gt`/`gte`/`lt`/`lte`. Sizes are in mm.
- Add long trigger phrases alongside short ones. Matching is longest-first.
- Use `ask_if_missing` for context that changes the recommendation and shouldn't be guessed.

Process:

1. Cite the guideline (publication, year, and journal reference where available) in `guideline`. Put non-obvious interpretation choices in `notes`.
2. Bump `_readme.version` (date stamp, e.g. `2026.11.06b`) on every content edit. Bump the integer `version` only when cutting a new `fuse-rules-vN` release.
3. Run the change through RadiusOS's `validateFuseRules` (`doxvox/modules/fuse/fuseRulesConfig.js`). It is the authoritative validator. The JSON Schema in this repo is only an editor aid.
4. Get radiologist sign-off on every changed threshold and output string, and name the reviewer in the PR.
5. Moving an item from `next_rules_to_add` into `rules`? Remove it from that list.
6. Keep the file's existing formatting (1-space indent) so diffs stay small and reviewable.

---

## Releasing a component

Releases are annotated tags on the content commit, followed by a commit that updates the manifest. Using the field library as the example:

```bash
# 1. Make sure the content commit is on main and field_library.json has "version": 11
git checkout main && git pull --ff-only

# 2. Tag the content commit
git tag -a field-library-v11 -m "Field library v11"
git push origin field-library-v11

# 3. Point the manifest at the new tag (manifests/field_library_manifest.json),
#    setting minRadiusOS / maxRadiusOS to the RadiusOS versions that support it.
git add manifests/field_library_manifest.json
git commit -m "content library - field library v11 release"
git push origin main

# 4. Bring dev level with main
git checkout dev && git merge --ff-only main && git push origin dev
```

For FUSE rules use tag `fuse-rules-vN`, tag message `FUSE rules vN`, and `manifests/fuse_rules_manifest.json`.

So far each release has *replaced* the previous manifest entry. If an older release must keep serving older RadiusOS versions, *add* an entry and make the RadiusOS ranges non-overlapping instead.

After releasing, move the component's entries in [CHANGELOG.md](CHANGELOG.md) from **Unreleased** to a new section for the tag.

---

## Commit messages

Use the existing prefix:

```text
content library - <short summary>
```

Examples from history:

- `content library - restructured repo into templates/, added field library v9 & manifest`
- `content library - fuse rules v1`
- `content library - fuse rules v1 release`

Prefer a descriptive summary over GitHub's default `Add files via upload` (you can edit the message in the web UI before committing).

---

## Pull request checklist

The [PR template](.github/pull_request_template.md) repeats this list:

- [ ] Targets `dev` (or is a `dev` → `main` promotion)
- [ ] All JSON parses, e.g. by running the one-liner in [README → Validating changes](README.md#validating-changes)
- [ ] `version` / `_readme.version` bumped where required
- [ ] FUSE changes pass `validateFuseRules` and have radiologist sign-off
- [ ] No PHI anywhere in the diff
- [ ] [CHANGELOG.md](CHANGELOG.md) updated under **Unreleased**
