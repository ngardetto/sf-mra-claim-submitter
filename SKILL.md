---
name: sf-mra-claim-submitter
description: Automates filing health-expense reimbursement claims through the HealthEquity/WageWorks online "Pay Me Back" portal (onlineclaims2.wageworks.com/Pmb and participant.wageworks.com) — most commonly for the San Francisco Medical Reimbursement Account (SF MRA / SF City Option), but the same portal is used by many other employer-sponsored FSA/HRA/MRA plans, so it should generalize. Use this skill whenever the user wants to submit receipts for reimbursement, mentions SF MRA, SF City Option, WageWorks, or HealthEquity Pay Me Back, or asks to "file a claim for my medical receipts," "submit these receipts for reimbursement," "upload these to my MRA/FSA/HRA," or hands over a folder of receipt photos/PDFs without naming the portal. Requires the Claude in Chrome extension; the user logs into the portal themselves — Claude never handles their credentials.
---

# SF MRA / WageWorks Pay Me Back Claim Submitter

## What this does

Walks through submitting a batch of health-expense receipts as a "Pay Me Back" reimbursement claim on the HealthEquity/WageWorks portal: review receipts, enter each as a claim line item, upload the receipt images, and submit. Built and proven against the San Francisco Medical Reimbursement Account (SF MRA), but the underlying portal is shared by many other WageWorks-administered Pay Me Back plans — expect the provider list and category-dropdown wording to differ by plan, but the overall flow to hold.

## Requirements

- Claude in Chrome browser extension, connected.
- The user has (or is willing to start) an active login session on the portal. Never enter their username, password, or any verification code on their behalf — wait for them to log in themselves.
- Read access to wherever the user's receipt files live.

## The shape of the workflow

This is a five-step pipeline. Steps 1–2 happen entirely in chat/filesystem, before the browser is touched. Steps 3–5 are browser automation, where steps 4 and 5 each end in a real, irreversible "submit" click.

1. **Find and review the receipts** (no browser)
2. **Flag anything uncertain, get one combined sign-off** (no browser)
3. **Enter claim line items** (browser, no pauses needed)
4. **Submit the claim batch** (browser, one quick confirmation right before clicking, then continue)
5. **Upload receipts and submit them** (browser, continues straight through from step 4's approval — no second wait)

This shape exists specifically to solve a real problem: WageWorks sessions can time out after a period of inactivity, and the riskiest moment for that is Claude sitting idle waiting on a chat reply *in the middle* of a portal session. So every question gets asked either before the browser opens, or right before the one continuous run of irreversible clicks — never in the middle of one.

## Step 1: Find and review the receipts

Locate the receipt files (ask the user where they are if unclear). For each file:

- If the filename starts with `SKIP`, skip it — that's the user's own signal that it isn't ready to submit yet.
- If it's a HEIC file (common for iPhone photos) or another format the portal won't accept, plan to convert it to JPG before upload — the WageWorks upload page typically only accepts JPG/PDF/TIFF/GIF/PNG, max 5MB/file.
- If a single photo shows multiple receipts/documents in frame, treat it as one image but check whether it actually corresponds to one transaction or several — don't split or merge without confirming with the user.
- Extract: vendor/provider name, date of service (or the closest reliable date if the receipt doesn't clearly state one — e.g. a pharmacy "promised" pickup date vs. the register transaction date), and amount.

Build a simple table of every receipt: filename → vendor → date → amount → proposed category. Note explicitly, for your own benefit and the user's, anywhere you had to guess at a date, pick a vendor on an ambiguous receipt, or fall back to a category that doesn't quite match. Genuinely stuck cases (illegible receipt, can't tell what was purchased, can't match to a vendor) should be skipped, not guessed — call them out instead of papering over them.

Once data is extracted and any ambiguities are resolved (after step 2), rename each source file before upload using this format:

```
{M}.{DD}.{YYYY}_{Vendor} {Brief Description}_{$Amount}{.ext}
```

For example: `8.22.2025_Amazon Nasal Spray_$6.97.jpg` or `11.11.2025_CVS Pharmacy Rx Copay_$14.98.jpg`

A few notes on the renaming:
- Month has no leading zero (8, not 08); day and year are as-is from the date of service.
- The description should be short but specific enough to identify the item at a glance — "Rx Copay" or "Eye Exam" or "OTC Allergy Meds" rather than just "prescription" or "purchase."
- Keep the original file extension.
- Strip or replace any characters that aren't filename-safe (slashes, colons, etc.).
- Rename in place (same folder) so the upload step can reference the updated paths.

This renaming happens after step 2's approval but before the browser session opens, so the descriptive names also appear in the WageWorks portal's file list (making it easier to match uploads to line items later).

## Step 2: One combined check-in, before opening the browser

Show the user the full table and ask once: "Here's what I found across N receipts, total $X. A few things to flag: [list]. Anything to correct before I start entering these?"

Get this fully resolved before touching the browser. The goal is that once the portal session starts, there should be nothing left to ask until step 4.

## Step 3: Enter claim line items

See `references/wageworks-portal-flow.md` for the exact page-by-page flow (URLs, button labels, the "Provider Name" / category dropdowns).

The category-mapping rule that matters most: WageWorks' claim-entry form typically has two expense-category dropdowns, worded differently from each other. Default to whichever one is labeled like "Other Services" and try to match the receipt's expense type to its wording. Only fall back to the "Common Services"-style dropdown when "Other Services" genuinely has nothing that fits (over-the-counter items are a common example). Log every fallback case — it gets re-surfaced in step 4.

Entering line items is just filling out a form — nothing is submitted to the plan administrator yet, so there's no need to pause for confirmation between items. Move through all of them, then proceed to step 4.

## Step 4: Submit the claim batch (first irreversible click)

Once every item is entered, the portal shows a review page with all items, the total, and a button (something like "Submit Receipt Online NOW") that files the claim with the plan administrator. This is real and not easily undone, so before clicking it: show the user the final item count and total, plus anything flagged in step 1/3, and get an explicit go-ahead. Keep this message short — a fast yes/no, not a re-litigation of step 2.

Once approved, click it and continue straight into step 5 without stopping again — the user's approval here covers the whole "submit claim → upload receipts → submit receipts" sequence. Pausing partway through this sequence is exactly what caused session timeouts in earlier runs of this workflow.

## Step 5: Upload receipts and submit them (second irreversible click)

The portal walks through an instructions page, then a receipt-upload page, then a final review page with a "SUBMIT RECEIPTS" button. Two important quirks, detailed further in the reference file:

- The upload page's file input typically registers **one file per upload action**, even though a file-upload tool may nominally accept multiple paths in one call. Loop through the receipts one at a time — click "Add Receipt"/"Add More Documentation," re-find the freshly-appeared file input (its reference changes after every click — never reuse a stale one), upload exactly one file, repeat.
- After the last file, click "No More Documentation for This Claim" to reach the final review page.

Because the user already approved this whole sequence in step 4, click "SUBMIT RECEIPTS" as soon as the upload is verified — don't pause again. The only reason to stop here is if something looks actually wrong (a missing file, a mismatched total) — in that case, stop and flag it rather than guessing.

After a successful submission, report back with a short confirmation (items submitted, total, claim status) and ask if the user wants the source receipt files moved to wherever they keep "already submitted" files.

## If a session times out mid-flow

WageWorks sessions can expire during idle periods. The good news: both entered claim items and uploaded-but-unfinalized receipt files persist server-side even after a timeout — nothing already entered or uploaded is lost, but the final "submit" click won't happen automatically.

Recovery path: go to the Claims & Activity page, open any one of the affected line items, find its "Submit Receipt" link, and follow it back into the same claim-review/upload flow. If files were already uploaded, a "Previously Uploaded Files Found" dialog will say so — no need to re-upload, just dismiss it and continue to "No More Documentation for This Claim" → final review → confirm with the user → submit.

## Hard rules (apply regardless of how confident things look)

- Never type the user's username, password, or any verification code. They log in; you wait until the portal shows as authenticated.
- Never guess on a receipt that's genuinely ambiguous — skip it and tell the user, rather than entering a category, date, or amount you're not confident in.
- Always skip files named with a `SKIP` prefix.
- Always get an explicit go-ahead before either of the two submit clicks in steps 4 and 5 — these are real, hard-to-reverse filings, and that doesn't change no matter how routine the batch looks. What does change, based on what's worked best, is *when* you ask: ask once, right before the time-sensitive sequence starts, not in the middle of it.
