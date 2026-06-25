# Hosting the admin page on GitHub Pages

The admin page (`docs/admin.html`) can run **outside** the Apps Script sandbox. The benefit:
a real **live webcam** for the student photo works, which the embedded Apps Script page can't do
(its iframe blocks `getUserMedia`).

It talks to the same Google Apps Script backend over the token-guarded JSON API (`doPost`).

## 1. Deploy the Apps Script web app for cross-origin access

In the Apps Script editor → **Deploy → New deployment → Web app**:

- **Execute as:** Me
- **Who has access:** **Anyone**  ← required; "Anyone with Google account" will **not** work from a browser page

Copy the **Web app URL** (ends in `/exec`).

Set the API token (Apps Script editor → **Project Settings → Script properties**):

| Property | Value |
|----------|-------|
| `API_TOKEN` | a long random secret string of your choosing (do not commit it anywhere) |

> Re-deploy a **new version** whenever you change `Code.gs`.

## 2. Enable GitHub Pages

Repo → **Settings → Pages**:

- **Source:** Deploy from a branch
- **Branch:** `main`  · **Folder:** `/docs`
- Save. After a minute the site is at:
  `https://saravanans-0753.github.io/estanciastudentonboarding/admin.html`

## 3. Connect the page

Open the Pages URL. Click **⚙ API connection** (top-right) and paste:

- the **Web app URL** (`…/exec`)
- the **API token**

These are saved in your browser only (localStorage). Load a flat and verify as usual; the
photo section now uses the live camera.

## Notes

- The same `Admin.html` still works embedded in Apps Script at `…/exec?page=admin`
  (it auto-detects which mode it's in). `docs/admin.html` is just a copy for Pages —
  keep them in sync when editing.
- Deep links work: `…/admin.html?flat=1137`.
- The API token lives in client-side storage and travels in requests; treat it as a
  shared secret for trusted admins, and rotate it via the `API_TOKEN` script property if needed.
