# Student Onboarding Tool (Flat-based, Webcam + Gemini)

> **New: Annexure 2A flow.** In addition to the legacy 6-tab onboarding tool below, the
> project now generates the **EFOA Annexure 2A (Tripartite)** form end-to-end:
> 1. **Students self-fill** a mobile **web form** (the default `doGet` route, share `…/exec?flat=<flat>`),
>    uploading Aadhaar + college-ID. Owner + caretaker details are captured here too.
> 2. Data → `Students` + `Flats` sheets; images → per-flat Drive folder.
> 3. An **admin web page** (hosted on GitHub Pages, `docs/admin.html`) pulls a flat, shows
>    the agreement, Aadhaar and ID side-by-side for verification, and captures each student's
>    **photo**. It talks to the script over the token-guarded `doPost` JSON API.
> 4. `generateAnnexurePdf(flat)` merges a **Google Doc template** → one combined PDF.
>
> Setup: see [`google-apps-script/ANNEXURE_TEMPLATE.md`](google-apps-script/ANNEXURE_TEMPLATE.md)
> (template + script properties).
> The legacy 6-tab tool is still available at `…/exec?page=legacy`.

---

Onboard up to 6 students per flat, one student per tab, with document capture, AI extraction, and cross-document name verification. Built on:
- Google Sheets (database)
- Google Apps Script (web app + automation)
- Google Drive (per-flat folder holding photos, ID cards, agreement, and detail files)
- WhatsApp click-to-chat links
- Webcam capture for the student photo + college ID; file/drag-drop/paste upload for Aadhaar
- Gemini API for Aadhaar, college-ID, and agreement extraction

## How it works

1. **Step 1 — Flat & agreement.** Enter the **flat number** (free text). Existing students for that flat auto-load into their tabs. Optionally upload the **flat agreement** (one per flat, PDF or image). A single agreement is signed between the owner and all (up to 6) students. For multi-page PDFs you get **page thumbnails with checkboxes** — tick only the page(s) containing student details, and **only those pages are sent to AI** (the rest are skipped to save tokens). The full original file is still stored in Drive. On save the selected pages are read by AI to extract **a list of students** (name, address, Aadhaar, phone, father/guardian), which **pre-fill empty tabs** as a starting point.
2. **Step 2 — Upload Aadhaar.** For each student tab, upload the Aadhaar image by **clicking to choose a file, dragging & dropping, or pasting (Ctrl/Cmd+V)**. It is auto-scanned.
3. **Step 3 — Verify details.** Aadhaar Number and Name auto-fill from the scan; review and complete Phone/Notes.
4. **Step 4 — Student photo & college ID.** Capture the **student photo** (webcam) and the **college/student ID** (webcam or upload, auto-scanned).
5. **Step 5 — Save.** On save, the student's name from **Aadhaar + College ID + Agreement** is compared. On a mismatch you get a dialog with the differing values and can **save anyway (accept) or cancel**.

Each tab is labelled `Student 1`…`Student 6` and renames live to the student's name. Students arrive at different times — saving a tab adds a Sheet row; re-saving an already-loaded student updates that same row in place.

## Key behaviours

- **Flat number** is stored in the Students sheet's `HouseId` column.
- **Capacity:** max 6 active students per flat (enforced on new saves).
- **Drive layout:** a folder named after the flat number is created under `IMAGE_FOLDER_ID` (or Drive root). It holds, per student: `<StudentId>.jpg` (face), `<StudentId>_aadhar.jpg`, `<StudentId>_collegeid.jpg`, and `<StudentId>.txt` (details). Per flat: `agreement.<ext>` plus `agreement_extract.json` (cached extraction).
- **Agreement is optional.** When present, it yields a list of up to 6 students; each onboarded student's name must match one entry in that list (see Name verification).
- **WhatsApp:** after onboarding a new student, a `wa.me` welcome link opens.
- **ID card export:** `Export CSV For ID Card` regenerates a CSV of active students.

## Folder content

- `google-apps-script/Code.gs`: backend logic
- `google-apps-script/Index.html`: web UI (tabs, capture, upload, matching)
- `google-apps-script/appsscript.json`: Apps Script config

## Data model in Sheets

### Students
- Timestamp
- StudentId
- Name
- AadharNumber
- Phone
- HouseId  *(stores the flat number)*
- AgreementStatus  *(legacy; left blank — agreement is now per-flat)*
- AgreementRef  *(legacy; left blank)*
- PhotoUrl
- AadharPhotoUrl
- Notes
- Status

### IDCardExport
- StudentId, Name, Phone, AadharNumber, PhotoUrl, HouseId

### Houses *(legacy)*
- HouseId, HouseName, OwnerName, OwnerEmail, OwnerPhone, Active
- Created by `setupSheets()` but no longer used by the UI (the flat number is free text). Safe to ignore.

## Setup (10–15 minutes)

1. Create a new Google Sheet (e.g. `Students Onboarding`).
2. Open `Extensions -> Apps Script`.
3. Copy `google-apps-script/Code.gs` into `Code.gs`.
4. Create an HTML file named `Index` and paste `google-apps-script/Index.html`.
5. Set the timezone to `Asia/Kolkata` in Apps Script settings.
6. Run `setupSheets()` once from the editor.
7. Authorize permissions.
8. Deploy the web app:
   - `Deploy -> New deployment -> Web app`
   - Execute as: `Me`
   - Who has access: `Anyone with the link` (or your domain only)
9. Open the web app URL and test one onboarding.

## Gemini setup (required for AI extraction)

1. Create a Gemini API key in Google AI Studio / Google Cloud.
2. In Apps Script: `Project Settings -> Script properties`.
3. Add property `GEMINI_API_KEY` = `<your-gemini-api-key>`.
4. Redeploy the web app after saving.

The current model is `gemini-1.5-flash` (see `GEMINI_MODEL` in `Code.gs`), which accepts both images and PDFs and is ~10–20× cheaper than `gemini-1.5-pro` with comparable accuracy for ID/document reading. Image uploads extract most reliably; scanned/image-only PDFs may extract less. To favour maximum accuracy over cost, change `GEMINI_MODEL` to `gemini-1.5-pro`. Always verify extracted values before saving.

### Approximate cost (Flash)

All three extractions (Aadhaar, College ID, agreement) go through one helper, `geminiExtractJson_`, which sends the image/PDF inline to Gemini. With Flash this is roughly **a small fraction of a cent per scan** — on the order of **~$0.001–0.003 to onboard a full 6-student flat** (12 image scans + 1 agreement). The main cost drivers are re-scans/retries and large image uploads. A free Gemini quota (AI Studio key) may cover low volumes at $0, just rate-limited. Verify current rates on Google's pricing page.

## Save photos in a specific Drive folder (recommended)

1. Create a Google Drive folder and copy its ID from the URL.
2. In Apps Script: `Project Settings -> Script properties`.
3. Add property `IMAGE_FOLDER_ID` = `<your-folder-id>`.

Per-flat subfolders are created inside this folder. If unset, they are created at Drive root.

## Hardware (office desk)

- One 1080p USB webcam with autofocus for the student photo (and college ID).
- Aadhaar is uploaded (file / drag-drop / clipboard paste), so no Aadhaar camera is required.
- Use Chrome on desktop; camera access requires the HTTPS web-app URL.

## Name verification

On save, the student's name is verified two ways (case- and spacing-insensitive): (1) the **Aadhaar** and **College ID** names must agree with each other, and (2) that name must appear **somewhere in the flat agreement's student list**. Any mismatch is flagged in a dialog with the offending values and can be **manually accepted** (save anyway) or cancelled.

## WhatsApp integration note

The app uses free `wa.me` links (opens a WhatsApp chat draft). The official WhatsApp Business API is typically paid.

## Current limitations

- No role-based login.
- Agreement extraction quality depends on the document; verify the pre-filled details before saving.
- College-ID image is stored in Drive but not tracked by a Sheet column.
- No official WhatsApp API automation.

## Possible enhancements

1. Search by phone/Aadhaar to avoid duplicate records.
2. Flat occupancy dashboard in the Sheet.
3. Print-ready ID label export.
4. Track the college-ID photo URL in the Students sheet.
