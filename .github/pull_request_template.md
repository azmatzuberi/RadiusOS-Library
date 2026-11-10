## Summary

<!-- What changed and why. -->

## Components touched

- [ ] Templates (`templates/…`)
- [ ] Field library (`field-library/field_library.json`)
- [ ] FUSE rules (`fuse-rules/…`)
- [ ] Manifests / release (`manifests/…`, tags)
- [ ] Docs only

## Checklist

- [ ] Targets `dev` (or is a `dev` → `main` promotion)
- [ ] All JSON parses (see README → Validating changes)
- [ ] `version` (field library) / `_readme.version` (FUSE rules) bumped where required
- [ ] FUSE changes pass `validateFuseRules` in RadiusOS
- [ ] Clinical text and threshold changes reviewed by a radiologist. Reviewer: <!-- name -->
- [ ] No patient data (PHI) anywhere in the diff
- [ ] `CHANGELOG.md` updated under **Unreleased**
