# Announcement JSON — Author Spec

This is the source of truth for `announcement.json`. Copy this file verbatim into the `bienenstock-announcement-popups` repo's `README.md` whenever it changes, so the AI author always reads the latest rules.

The BienenStock admin app fetches **one** `announcement.json` from GitHub (raw) and shows it as a one-time popup to admins on the live dashboard (`/admin/aktuell`).

---

## Mental model (read this first)

- **There is only ONE announcement at any time.** The file is a single object, not a list. Publishing a new announcement means **overwriting the file**. There is no queue, no stacking, no deletion.
- **Admins see each announcement once.** When they click "Alles klar!" (or the CTA button), the `id` goes into their browser's `localStorage`. They'll never see that `id` again on that device.
- **The `id` is the identity of the announcement.** Changing `id` = new announcement = every admin sees it again. Keeping the same `id` but editing the content = **nobody re-sees it**. If you want everyone to see your update, you MUST change the `id`.
- **Admins check at most once every 24h**, and only after 10:00 Berlin time. A fresh publish isn't instant — allow up to a day for full rollout.
- **`active: false` hides the popup.** Content can stay in the file; it just won't be shown.

---

## Schema

```json
{
  "id": "2026-04-21-short-slug",
  "title": "Header text",
  "text": "Main message paragraph. Use **bold**, *italic*, [Link](https://...).",
  "bullets": ["Optional bullet 1", "Optional bullet 2"],
  "image_url": "https://example.com/image.png",
  "cta": { "label": "Mehr erfahren", "url": "https://example.com/page" },
  "tone": "info",
  "active": true
}
```

| Field       | Required | Type     | Limit                          | Notes                                                                         |
| ----------- | -------- | -------- | ------------------------------ | ----------------------------------------------------------------------------- |
| `id`        | ✅ yes   | string   | ≤ 60 chars                     | MUST be unique vs. every previous id. New id = all admins see popup again.    |
| `title`     | ✅ yes   | string   | ≤ 40 chars                     | Short bold header. No formatting, no emojis, plain text.                      |
| `text`      | ✅ yes   | string   | ≤ 280 chars                    | Paragraph. Supports `\n`, `\n\n`, `**bold**`, `*italic*`, `[label](https://)`.|
| `bullets`   | optional | string[] | ≤ 4 items, each ≤ 80 chars     | Same inline formatting as `text`. **Omit the field** if unused (no `null`/`[]`). |
| `image_url` | optional | string   | HTTPS only, ≤ 500 KB           | **Omit** if unused. Recommended 800×450, jpg/png/webp, public URL.            |
| `cta`       | optional | object   | `label` ≤ 20 chars, `url` HTTPS | Extra action button next to "Alles klar!". Opens in new tab. **Omit** if unused. |
| `tone`      | optional | enum     | `"info"` \| `"success"` \| `"warning"` | Colors the header band + icon. Defaults to `"info"` if omitted.         |
| `active`    | ✅ yes   | boolean  | —                              | `false` hides it for everyone immediately (next 24h check).                   |

Any field not listed above is silently ignored.

---

## Inline formatting (usable in `text` and `bullets`)

| Want                  | Write                                | Renders as                               |
| --------------------- | ------------------------------------ | ---------------------------------------- |
| Bold                  | `**wichtig**`                        | **wichtig**                              |
| Italic                | `*kursiv*`                           | *kursiv*                                 |
| Link (new tab)        | `[mehr erfahren](https://example.com)` | underlined honey-colored link           |
| Line break            | `\n`                                 | soft break (one line down)               |
| Paragraph break       | `\n\n`                               | blank line between paragraphs            |

**Rules:**
- Only `https://` links are rendered. `http://`, relative, or `mailto:` URLs are printed as plain text.
- Formatting does **not** nest. `**bold *italic* bold**` won't work — pick one per span.
- Formatting does **not** cross line breaks. Don't wrap `**` around `\n`.
- Markdown beyond this subset (`##`, `>`, `` ` ``, `---`, images, tables) renders literally as characters. Don't use it.
- No HTML tags. `<b>`, `<br>`, `<a>` render literally.

---

## Tone

Choose based on the *meaning*, not the mood:

| `tone`       | Header color        | Icon             | Use for                                               |
| ------------ | ------------------- | ---------------- | ----------------------------------------------------- |
| `"info"` (default) | warm honey-cream | 📢 megaphone     | New feature, general announcement, FYI                |
| `"success"`  | soft green          | ✅ check circle   | Feature shipped, bug fixed, migration complete        |
| `"warning"`  | soft red            | ⚠️ amber triangle | Action required, deadline, outage, breaking change   |

Default to `info`. Use `warning` sparingly — it stops reading flow.

---

## CTA button (optional)

If the announcement links to something meaningful (a docs page, a new feature, a form), add a `cta`:

```json
"cta": { "label": "Mehr erfahren", "url": "https://docs.bienenstock.app/report-engine" }
```

- Appears as an outlined button to the left of "Alles klar!".
- Opens `url` in a new tab (`target="_blank"`, safe `rel`).
- Clicking it also marks the announcement as seen.
- Only one CTA supported. Keep `label` ≤ 20 chars — e.g. "Mehr erfahren", "Jetzt testen", "Zum Ticket".
- If the URL isn't `https://...`, the button is hidden (silently).

Don't add a CTA just because you can — most announcements work fine with just "Alles klar!".

---

## `id` naming convention

Format: `YYYY-MM-DD-short-slug` (date = publish date, slug = 2–4 lowercase words with hyphens).

✅ `2026-04-21-report-engine`
✅ `2026-04-21-qr-scanner-fix`
❌ `update` (not unique, not descriptive)
❌ `2026-04-21` (no slug — collides if you publish twice in one day)
❌ `2026-04-21-Report Engine` (spaces, uppercase)

Before choosing an id, check the repo's git log to make sure it hasn't been used before.

---

## Actions (what to do when)

| Intent                                             | Action                                                            |
| -------------------------------------------------- | ----------------------------------------------------------------- |
| Publish a brand-new announcement                   | Overwrite the file. New `id`. `active: true`.                     |
| Fix a typo in the **currently live** announcement  | Edit `text`/`title`. Keep `id`. **Only admins who haven't dismissed yet will see the fix.** If the correction matters, bump the `id`. |
| Retract/hide the current announcement              | Set `active: false`. Leave the rest.                              |
| Replace one announcement with another              | Overwrite the file. New `id`. Old one is gone — that's fine.      |

You never need to "delete" old announcements. The file always holds exactly one.

---

## Style rules

- Language: German (the whole app is German).
- Tone: brief, warm, direct. Think "colleague note", not marketing copy.
- No emojis in `title`. A single emoji in `text` is fine if it earns its place.
- Don't shout. No ALL-CAPS titles. No more than one "!" per field.
- Keep total on-screen content under ~10 lines — the popup is ~400 px wide.

---

## Worked examples

### ✅ Simple info announcement

```json
{
  "id": "2026-04-21-report-engine",
  "title": "Neue Report-Engine",
  "text": "Die Berichte-Seite hat eine neue Engine. Ihr könnt sie im **Dev-Toggle** ausprobieren.\n\nFeedback gerne direkt an Finn.",
  "tone": "info",
  "active": true
}
```

### ✅ Success announcement with CTA

```json
{
  "id": "2026-04-22-personalplanung-beta",
  "title": "Personalplanung (Beta)",
  "text": "Ab heute könnt ihr die neue *Personalplanung* testen:",
  "bullets": [
    "Monatsansicht mit Arbeitsmodellen",
    "Live-Abgleich mit Check-ins",
    "Vorlagen zum Duplizieren"
  ],
  "image_url": "https://raw.githubusercontent.com/finnexmachina/bienenstock-announcement-popups/main/img/personalplanung-preview.png",
  "cta": { "label": "Jetzt testen", "url": "https://app.bienenstock.de/admin/personalplanung" },
  "tone": "success",
  "active": true
}
```

### ✅ Warning with deadline

```json
{
  "id": "2026-04-25-passwort-update",
  "title": "Passwort-Update bis Freitag",
  "text": "Bitte aktualisiert euer Admin-Passwort **bis Freitag, 30.04.** Danach läuft der alte Token ab.\n\nBei Fragen: [Support-Seite](https://docs.bienenstock.app/support)",
  "cta": { "label": "Passwort ändern", "url": "https://app.bienenstock.de/admin/einstellungen" },
  "tone": "warning",
  "active": true
}
```

### ❌ Common mistakes

```json
{
  "id": "update",                            // not unique, not descriptive
  "title": "WICHTIG!! NEUE FEATURES 🎉",     // shouty, emoji in title
  "text": "## Überschrift\n<br><b>fett</b>", // headers + HTML render literally
  "bullets": [],                             // empty array — omit the field
  "image_url": null,                         // null — omit the field
  "cta": { "label": "Klick", "url": "/path" }, // not https — CTA silently hidden
  "tone": "danger",                          // invalid value — silently ignored
  "active": "true"                           // string instead of boolean
}
```

---

## Publish checklist (use every time)

1. [ ] `id` is new (grep the repo's git log: `git log --all -p -- announcement.json | grep '"id"'`)
2. [ ] `id` follows `YYYY-MM-DD-slug` format
3. [ ] `title` ≤ 40 chars, no emoji, no "!!!"
4. [ ] `text` ≤ 280 chars, German, inline formatting stays inside allowed subset
5. [ ] Optional fields are either correctly filled OR omitted entirely (not `null`, not `[]`)
6. [ ] `tone` is one of `info` / `success` / `warning` (or omitted)
7. [ ] `cta.url` starts with `https://` if present
8. [ ] `active: true` is a boolean, not a string
9. [ ] JSON is valid (no trailing commas, all strings quoted)
10. [ ] Commit message describes the announcement, e.g. `announce: personalplanung beta`

---

## Behavior reference (for debugging)

- Admin app fetches `announcement.json` from `https://raw.githubusercontent.com/finnexmachina/bienenstock-announcement-popups/main/announcement.json`.
- Fetch happens at most once per 24h per browser (once per hour in dev builds), only after 10:00 Berlin time, only on `/admin/aktuell`.
- Each dismissed `id` is stored in `localStorage["seen_announcements"]` (capped at the last 50 entries).
- To force-retest as a developer: clear `seen_announcements` and `announcement_last_fetched` in DevTools → Application → Local Storage.
