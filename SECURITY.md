# Security and Patient Data Policy

This repository contains no executable code. Its risks are of a different kind:

1. **Patient data (PHI) exposure.** Templates, prompts or test files that contain real patient information.
2. **Clinical-safety defects.** A wrong threshold, interval or recommendation in a FUSE rule, or misleading default-normal text in a template or the field library. RadiusOS could insert these into a report.
3. **Secrets.** Credentials, API keys or internal URLs committed by mistake.

## Patient data

- **Never commit PHI.** This includes names, dates of birth, MRNs, accession numbers, real exam or comparison dates, facility-specific patient identifiers, and free-text history copied from a real report.
- Build examples and test templates (such as those in `templates/Other/`) from synthetic content only.
- Use placeholders (`[]`, `[None]`, `[Unspecified.]`) for every patient-specific slot.

## Reporting an issue

**Do not open a public issue or PR that reveals PHI, a secret, or details of an unreleased clinical-safety defect.**

Instead, contact the repository maintainers (the RadiusMD team) privately through your usual internal channel, or use GitHub's private vulnerability reporting if it is enabled for this repository. Please include:

- the file path(s), plus the commit SHA or release tag affected;
- what is wrong, and for clinical issues the guideline or reference that shows it;
- whether the affected content is in a released tag listed in `manifests/`, because that determines whether RadiusOS is currently serving it.

## If PHI or a secret has been committed

Deleting the file in a new commit is **not enough**. The data stays in git history, in release tags and in GitHub's caches.

1. Notify the maintainers immediately. Don't try to fix it quietly.
2. Rotate any exposed secret right away.
3. The maintainers will coordinate a history rewrite across `main`, `dev` and any affected tags, a GitHub cache purge, and re-cloning by everyone with a checkout.

## Clinical-safety fixes

Fixes to released clinical content follow the normal [release process](CONTRIBUTING.md#releasing-a-component), with radiologist sign-off, and are expedited. The fix ships as a new tag and the manifest moves to it. Existing tags are never rewritten for a clinical fix, so the audit trail stays intact.
