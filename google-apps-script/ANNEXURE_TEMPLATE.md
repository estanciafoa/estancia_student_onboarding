# Annexure 2A — Google Doc template setup (one-time)

`generateAnnexurePdf(flat)` works by copying a **Google Doc template**, replacing
`{{tokens}}` with the flat's data, inserting signature/photo images, and exporting a PDF.
You build this template **once** by hand, then never touch it again.

## 1. Create the template Doc

1. Open the original `TRIPARTITE LETTERS TO OWNERS Annexure 2A .pdf` for reference.
2. Create a new Google Doc named e.g. **"Annexure 2A — Template"**.
3. Recreate the 5 pages using **tables** (Insert ▸ Table) so the layout matches the
   official form. Keep the clause text (points 1–28) verbatim.
4. Where a value goes, type the matching `{{token}}` from the list below.
5. **Image cells** (`{{..._photo}}`, `{{..._sig}}`, `{{owner_sig}}`, `{{efoa_sig}}`): put the token
   **alone in its own table cell** (nothing else in that cell). The script clears the
   token and drops the image into that cell. Give the cell roughly the final size so
   the image sits where you want.
6. Leave the **manual-fill cells empty** (no token): per-student parent-email, course,
   ID valid-upto; owner address; nature/date of rent agreement; tenancy period; the
   whole POA block. These print blank for handwriting.
7. Add the two intentional deviations from the official layout:
   - a **photo cell** in each of the 6 student blocks (token `{{sN_photo}}`),
   - a **caretaker line** in the owner block (`{{caretaker_name}}`, `{{caretaker_number}}`).

## 2. Wire it up

1. In the Doc URL `https://docs.google.com/document/d/<ID>/edit`, copy `<ID>`.
2. Apps Script ▸ **Project Settings ▸ Script properties** ▸ add
   `ANNEXURE_TEMPLATE_ID` = `<ID>`.
3. (For the native admin app) also add `API_TOKEN` = a shared secret the app sends.
   For the EFOA (association) signature, add `EFOA_SIGNATURE_ID` = the Drive file id/URL
   of the signatory's signature PNG (or run `saveEfoaSignature(base64Png)` once to store it).
4. Run `setupSheets()` once to add the new `Students` columns and the `Flats` sheet.
5. Redeploy the web app (Deploy ▸ Manage deployments ▸ New version).

## 3. Token reference

### Flat / owner / caretaker (header — page 1, undertaking — page 5)
| Token | Source |
|---|---|
| `{{apartment}}` | flat number |
| `{{owner_name}}` `{{owner_mobile}}` `{{owner_email}}` | `Flats` sheet |
| `{{caretaker_name}}` `{{caretaker_number}}` | `Flats` sheet (added line) |
| `{{owner_sig}}` (image) | owner signature (admin tablet) — or leave blank and rely on the uploaded rental agreement |
| `{{efoa_sig}}` (image) | EFOA authorized-signatory signature — one stored image, stamped on every agreement (`EFOA_SIGNATURE_ID`) |
| `{{signing_date}}` `{{signing_place}}` | admin tablet |

### Per student — repeat for `N` = 1..6 (pages 3–5)
| Token | Source |
|---|---|
| `{{sN_name}}` | name |
| `{{sN_mobile}}` | mobile |
| `{{sN_email}}` | email |
| `{{sN_address}}` | residential address |
| `{{sN_parent_name}}` | parent name |
| `{{sN_parent_mobile}}` | parent mobile |
| `{{sN_college_id}}` | college student-ID |
| `{{sN_college}}` | college name |
| `{{sN_acad_year}}` | academic year / year |
| `{{sN_sig}}` (image) | student signature (drawn by the resident on the in-person digital pad in the admin app, after reading the agreement) — also the page-3 signature grid |
| `{{sN_photo}}` (image) | student photo (admin tablet, added cell) |

Notes:
- Unfilled student slots (a flat with < 6 students) get blank tokens automatically.
- Image tokens left without a captured image are simply cleared (cell stays empty).
- The page-3 "signatures" grid can reuse the same `{{sN_sig}}` token in each numbered box.
