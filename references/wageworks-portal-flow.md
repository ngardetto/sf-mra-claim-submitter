# WageWorks / HealthEquity Pay Me Back Portal — Page-by-Page Flow

This describes the exact screens encountered while submitting an SF MRA Pay Me Back claim. Page titles and button labels may vary slightly by plan, but the overall structure should hold for any WageWorks-administered Pay Me Back claim.

## Starting a claim

- `participant.wageworks.com` is the main dashboard ("Claims & Activity", etc.) — the user logs in here themselves.
- A "Submit Receipt or Claim" action (wording may vary) starts a new claim, landing on `onlineclaims2.wageworks.com/Pmb/...`.

## Entering claim items

- The claim-entry wizard asks for: Provider Name (a dropdown, plan-specific — check what's actually in the dropdown rather than assuming an option exists), Date of Service, an expense category (two dropdowns worded differently — see SKILL.md's category-mapping rule), and Amount, one item at a time, with an "Add Another" / "NEXT" pattern.
- This step is just form-filling — no irreversible action yet, so there's no need to pause between items.

## `SingleStepSubmit.aspx` — "Review and Submit Claim (Step 2 of 2)"

- Lists every entered item and the total.
- Three options: "Submit Receipt Online NOW" (the one this skill uses), "Submit Receipt Online LATER", "Download Claim Form (PDF)".
- Clicking "Submit Receipt Online NOW" files the claim — this is the **first irreversible action**. Confirm with the user before clicking, per SKILL.md step 4.

## `UploadInstructions.aspx` — "Instructions"

- Describes accepted file formats (typically JPG/PDF/TIFF/GIF/PNG) and a per-file size limit (typically 5MB). Click NEXT.

## `UploadReceipt.aspx` — "Select Receipt File(s) (Step 1 of 2)"

- Shows one tile per claim batch (vendor name, item-count badge, total $).
- First upload: click "Add Receipt for This Claim". Every upload after that: click "Add More Documentation for This Claim".
- **Each click reveals a new file input element. Re-find it fresh every time — do not reuse a previous element reference, it goes stale after the page's AJAX postback** (confirmed: reusing a stale reference throws "No element found with reference").
- **Upload exactly one file per upload call, even if the tool nominally accepts multiple paths.** Tested directly: passing 3 file paths to a single file-upload call reported success for all 3, but the page's file list only actually showed the first one. The reliable pattern is: click add button → find the new file input → upload one file → repeat for every receipt.
- `get_page_text` often fails on this page ("No text content found") because it's canvas/image-heavy — rely on screenshots to verify the file list instead.
- The browser's JS-execution tool may be blocked on this page (cookie/query-string restriction) — don't depend on it for verification here.
- When all files are uploaded, click "No More Documentation for This Claim".
- If returning after a session timeout, a modal titled "Previously Uploaded Files Found" may appear, confirming already-uploaded files persisted server-side — dismiss it with OK and continue; no re-upload needed.

## `SubmitReceipt.aspx` — "Review and Submit Receipt(s) (Step 2 of 2)"

- Final review: lists every uploaded file name and size, with a "SUBMIT RECEIPTS" button.
- This is the **second and final irreversible action**. Per SKILL.md, this should be covered by the same go-ahead obtained before starting step 5 — don't pause again here unless something looks actually wrong (file count, total) versus what was approved.
- On success, a "Success!" modal confirms submission and that the claim will process in a few business days. Dismiss with OK.

## Recovery / Claims & Activity

- `participant.wageworks.com/ClaimsAndPayments/Index.aspx` lists every claim line item individually, tagged "Need Receipt" until receipts are finalized.
- Clicking a line item opens `TransactionDetails/PmbClaimDetails.aspx`, which has a "Submit Receipt" link that routes back into the `SingleStepSubmit.aspx` flow for the whole claim batch (all line items in one claim share one Claim Form #).
- This recovery path is exactly how to resume after a session timeout discovered mid-upload: it picks back up at the batch level, not per line item.
