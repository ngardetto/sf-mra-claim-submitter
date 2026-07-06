# sf-mra-claim-submitter

A Claude skill that automates filing reimbursement claims through the HealthEquity/WageWorks "Pay Me Back" portal — built for the San Francisco Medical Reimbursement Account (SF MRA), but it works with any WageWorks-administered Pay Me Back plan.

It reads your receipt photos, flags anything uncertain upfront, enters each claim line, uploads the receipts, and submits — pausing only to confirm before any irreversible action.

## What you need

- A **paid Claude plan** (Pro at $20/mo or higher) with Cowork mode
- The **[Claude in Chrome](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn)** browser extension installed in Chrome
- An active account at [participant.wageworks.com](https://participant.wageworks.com) (you log in yourself — the skill never touches your credentials)

## Install

1. Click the green **Code** button on this page → **Download ZIP**.
2. In Claude, open **Customize → Skills** and upload the downloaded zip file.

Alternatively, click `sf-mra-claim-submitter.skill` in the file list above and use the download icon (top-right of the file preview) to download just the skill file and upload that instead.

## Use

1. Put your receipt photos or PDFs in one folder.
2. Log into your WageWorks/SF MRA account in Chrome.
3. Open a new Cowork chat and say something like: *"Submit my SF MRA receipts — photos are in [folder path]."*
4. Claude will summarize what it found and flag anything that needs your input before starting.
5. Confirm once before each irreversible submit step and let it go.

## Notes

- Receipt files named with a `SKIP` prefix are skipped automatically.
- HEIC photos (iPhone default) are converted to JPG before upload, since WageWorks doesn't accept HEIC.
- The category-mapping logic and portal quirks (including the one-file-per-upload limitation) are documented in [`references/wageworks-portal-flow.md`](references/wageworks-portal-flow.md).

---

*A standalone version (no Claude subscription required, no browser extension) is on the roadmap.*
