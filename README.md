# OMAC International — Research Forum

Two frontends, one backend (Google Sheets via Apps Script):

- **`apps-script/`** — `Code.gs`, `Index.html` (respondent-facing form),
  `Admin.html` (your admin panel) — all hosted directly by Apps Script.
- **`github-pages/`** — static copies of the same two pages
  (`index.html`, `admin.html`) for hosting on GitHub Pages, which call
  your Apps Script deployment as a JSON API instead of using
  `google.script.run`.

Both point at the same Google Sheet, so responses and questionnaire
changes made from either frontend land in the same place.

## One-time setup

1. Create a new Google Sheet.
2. Extensions → Apps Script.
3. Paste in `apps-script/Code.gs` as `Code.gs`.
4. File → New → HTML file, name it exactly `Index`, paste in
   `apps-script/Index.html`.
5. File → New → HTML file, name it exactly `Admin`, paste in
   `apps-script/Admin.html`.
6. Run `setup()` once from the function dropdown (approve permissions
   when asked). This creates three sheet tabs:
   - **Questionnaires** — empty at first; you add studies yourself (see below)
   - **Responses** — where every submission lands
   - **Accounts** — hidden by default; this is where your admin login lives
7. Unhide the **Accounts** sheet (right-click any tab → Unhide sheet)
   and add one row yourself: your chosen username in column A, your
   chosen password in column B. Nothing is pre-filled — set your own.
8. Deploy → New deployment → Web app. Execute as **Me**, access
   **Anyone with the link**. Copy the `/exec` URL.

## The three pages

- `<your-url>/exec` — the respondent-facing questionnaire (this is what
  you send to hospital contacts)
- `<your-url>/exec?admin=1` — your admin panel: sign in, see every
  questionnaire's status, open/close one, or publish a new one by
  pasting its JSON
- The **Questionnaires** sheet tab itself — you can also add or edit a
  study by pasting JSON directly into a row's `schema_json` cell and
  setting `status` to `open`, no admin panel needed, if you'd rather
  work in the Sheet directly

## Adding a new questionnaire (either way works)

**Via the admin panel:** open `?admin=1`, sign in, paste an id, a
title, and the full JSON schema (see
`questionnaire_overmanaged_underled.json` for the format) into the
"Add a new questionnaire" form, click Publish. This automatically
closes whatever was open before and opens the new one.

**Via the Sheet directly:** add a new row to Questionnaires —
`id | title | status | schema_json | created_at` — paste the JSON into
the `schema_json` cell, set `status` to `open`, and set every other
row's status to `closed` if you only want one live at a time.

Neither `index.html` needs to change or be redeployed when you do
this — both frontends always render whatever is currently marked
`"open"`.

## GitHub Pages (optional second frontend)

1. Make sure `apps-script/Code.gs` above is deployed — `github-pages/`
   depends on it for both reading and writing data.
2. Push `github-pages/index.html`, `github-pages/admin.html`, and the
   `github-pages/assets/` folder to your repo, keeping that folder
   structure (the pages reference `assets/omac-logo-dark.png` as a
   relative path).
3. Settings → Pages → set the source → Save.
4. If you ever redeploy Apps Script and get a new `/exec` URL, update
   the `APPS_SCRIPT_URL` constant near the top of both
   `github-pages/index.html` and `github-pages/admin.html`.

Submissions from the static page are sent as `Content-Type: text/plain`
rather than `application/json` — this keeps the browser from
preflighting the request with an OPTIONS call, which Apps Script web
apps don't handle. `Code.gs` still parses the body as JSON server-side.

## On the admin login

This is a lightweight gate, not enterprise-grade auth: the
username/password pair lives in a plain (hidden) Sheet tab and is
checked as an exact text match. That's appropriate for keeping casual
visitors from editing your studies — it is not designed to withstand a
determined attacker, and the password should not be one you reuse
anywhere sensitive. If you ever need stronger protection, restricting
"Who has access" on the Apps Script deployment itself to specific
Google accounts is the more robust option.

## Where responses land

Every submission — from either frontend — appends a row to
**Responses**: `timestamp | questionnaire_id | answers_json`, where
`answers_json` is a JSON object keyed by question id (e.g.
`{"A1":"Quality staff","B1":4,...}`). This keeps the sheet
schema-agnostic, so it works the same way for every future
questionnaire without redesigning columns — expand it with a short
script or `json.loads()` per row when you're ready to analyze.
