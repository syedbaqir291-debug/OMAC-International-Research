# OMAC International — Research Forum

A Google Apps Script web app that serves JSON-defined questionnaires and
stores responses in a Google Sheet. Ships with the "Overmanaged and
Underled" study already loaded and open.

## Files

- `Code.gs` — backend: serves the page, reads/writes the Sheet, exposes
  admin functions for opening/closing/adding studies.
- `Index.html` — frontend: animated instrument-panel UI, renders
  whichever questionnaire is currently marked "open."
- `questionnaire_overmanaged_underled.json` — the schema for the first
  study, already embedded inside `Code.gs`. Kept here as the template
  to copy when you write the next one.

## One-time setup (about 5 minutes)

1. Create a new Google Sheet (any name — e.g. "OMAC Research Forum Data").
2. Extensions → Apps Script.
3. Delete the default `Code.gs` content, paste in this project's `Code.gs`.
4. File → New → HTML file, name it exactly `Index` (Apps Script adds
   `.html` itself), paste in this project's `Index.html` content.
5. In the function dropdown at the top of the editor, select `setup`,
   then click Run. The first time, Google will ask you to authorize the
   script — approve it (it only touches this one Sheet).
6. This creates two tabs in your Sheet — `Questionnaires` and
   `Responses` — and seeds the Overmanaged & Underled study as `open`.
7. Deploy → New deployment → gear icon → Web app.
   - Execute as: **Me**
   - Who has access: **Anyone with the link** (or "Anyone" if you want
     it fully public — use "Anyone with the link" for a link you send
     directly to hospital contacts)
8. Click Deploy, copy the web app URL — that's the link you send out.

## Where responses land

Every submission appends a row to the `Responses` tab:
`timestamp | questionnaire_id | answers_json`

`answers_json` is a JSON object keyed by question id (e.g.
`{"A1":"Quality staff","B1":4,"B2":5,...}`). To analyze in Excel/SPSS,
either expand it with a short script, or open it directly in Python/R
with `json.loads` per row — this keeps the sheet schema-agnostic so it
works for every future questionnaire without redesigning columns.

## Adding a new questionnaire later

You don't need to touch `Index.html` or redeploy to add a new study —
the frontend just renders whatever JSON schema is marked "open."

1. Write the new questionnaire as a JSON file, following the structure
   in `questionnaire_overmanaged_underled.json` (sections, each with
   `type: "context"` or `type: "likert"`, and a `questions` array).
2. In the Apps Script editor, open the console (View → Logs isn't
   needed — instead use the built-in script editor's "Execute function")
   or simply run this once, substituting your values:

   ```js
   addQuestionnaire(
     "your_new_id",
     "Your New Study Title",
     `PASTE YOUR MINIFIED JSON HERE`
   );
   ```

   By default this closes the previous study automatically (only one
   study is "live" at a time) — pass `false` as a 4th argument if you
   want more than one open simultaneously:
   `addQuestionnaire(id, title, json, false)`.

3. To reopen a past study instead of writing a new one:
   `openQuestionnaire("overmanaged_underled_v1")`
4. To close a study without adding a new one:
   `closeQuestionnaire("overmanaged_underled_v1")`

No redeploy is needed for any of this — the same web app URL keeps
working; it just starts serving the new schema on next page load.

## Updating the logo

The logo is embedded as base64 inside `Code.gs` (`LOGO_BASE64_`). To
change it, base64-encode the new PNG and replace that constant.
