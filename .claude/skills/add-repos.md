---
name: add-repos
description: Add one or more GitHub repos (with optional loose notes) to the Label-Printing-Resources index. Dedupes against existing entries, classifies into the right section, and creates new sections/subsections if needed. Always emits the standard entry format (header + description + dynamic badges + link).
---

# add-repos

Use this skill whenever the user pastes a list of GitHub repo URLs (with or without notes) and asks to add them to the index.

## Inputs

- One or more `https://github.com/OWNER/REPO` URLs.
- Optional free-text notes per repo (manufacturer hint, "this one's bluetooth", "broken", etc.). Treat these as classification hints, not as the final description.

## Workflow

1. **Read** `README.md` to learn current sections and the full set of indexed repos.
2. **Dedupe.** Extract `github.com/OWNER/REPO` strings from `README.md`, lowercase, and drop any incoming URL that matches (case-insensitive). Report skipped duplicates in the final summary.
3. **Classify** each remaining repo into one of the existing top-level sections:
   - `Brother` — Brother QL, P-touch, PT-*, ptouch-*, brother_ql*, etc.
   - `Niimbot`
   - `Dymo`
   - `Zebra`
   - `Multi-Brand / Bridges` — CUPS plugins, WebUSB/BLE bridges, InvenTree/Homebox integrations that aren't tied to one brand
   - `Format Conversion` — convert between label formats / image-to-label
   - `Label Design / General` — designers, generic label tools, NFC/QR utilities not tied to a brand
   If a repo doesn't fit cleanly, **add a new top-level section** (alphabetical-ish, before `Label Design / General` which stays last before `Entry Format`). For a meaningful sub-grouping within a section (e.g. Brother → Bluetooth), use `### Subsection` only if there are 3+ entries; otherwise keep flat.
4. **Fetch a one-line description.** Prefer the repo's GitHub description (via `gh repo view OWNER/REPO --json description -q .description`). Fall back to the user's note, then to a generic line based on the repo name. Keep it to one sentence, no trailing period unless natural.
5. **Insert** each new entry in its section using the standard format (below). Append at the end of the section unless an obvious alphabetical/grouping slot exists.
6. **Update the `Snapshot:` date** near the top of the README to today's date (YYYY-MM-DD).
7. **Commit and push** per the user's git policy: stage all, single commit `docs: add N repos to index`, push immediately. Do not split commits.

## Standard entry format

Every entry MUST follow this exact shape — header, one-line description, badges line, link line, blank line between entries:

```markdown
### Project Name

One-line description.

![Stars](https://img.shields.io/github/stars/OWNER/REPO?style=flat) ![Last commit](https://img.shields.io/github/last-commit/OWNER/REPO?style=flat)

[OWNER/REPO](https://github.com/OWNER/REPO)
```

Rules:
- **Header (`###`)** is the project's short name, not `OWNER/REPO`. If the bare name collides with an existing entry, disambiguate with ` (owner)` suffix — see `ptouch (nbuchwitz)` for precedent.
- **Description** is one sentence. No marketing fluff.
- **Badges** are exactly two: `Stars` and `Last commit`, both `style=flat`, in that order, on one line separated by a space. No other badges (no license, no issues, no build).
- **Link line** is the bare `OWNER/REPO` text wrapped in a markdown link to the repo root. No trailing text.
- All four blocks separated by blank lines. No bullets, no tables.

The `Entry Format` section at the bottom of the README is the canonical reference — keep it in sync if the format ever evolves.

## Section ordering

Top of README → ToC → sections in this order:

1. Brother
2. Niimbot
3. Dymo
4. Zebra
5. (any new brand-specific sections, alphabetical)
6. Multi-Brand / Bridges
7. Format Conversion
8. Label Design / General
9. Entry Format
10. Contributing

When adding a new top-level section, update the ToC at the top too.

## Output

After editing, report:
- N added (list as `OWNER/REPO → Section`)
- N skipped as duplicates (list)
- Any new sections/subsections created
- Commit SHA + pushed status
