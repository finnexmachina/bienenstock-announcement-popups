# Announcement JSON Spec

The BienenStock app reads this `announcement.json` to show a one-time popup to admin users.

## Schema

```json
{
  "id": "YYYY-MM-DD-short-name",
  "title": "Header text (max ~40 chars)",
  "text": "Main message body paragraph.",
  "bullets": ["Optional bullet 1", "Optional bullet 2"],
  "image_url": "https://example.com/image.png",
  "active": true
}
```

| Field | Required | Type | Notes |
|---|---|---|---|
| `id` | Yes | string | Must be unique. New id = all users see it again. |
| `title` | Yes | string | Bold header with icon. Keep short. |
| `text` | Yes | string | Plain text paragraph. |
| `bullets` | No | string[] | Omit entirely if unused. |
| `image_url` | No | string | Omit entirely if unused. |
| `active` | Yes | boolean | `false` = hidden for everyone. |

## Actions

- **New announcement:** Change `id`, set content, `active: true`
- **Deactivate:** Set `active: false`
- **Re-show to all:** Change `id` to a new value

## Rules

- All values are plain text, no HTML/markdown
- Omit optional fields entirely, don't set to `null` or `[]`
- Popup is ~400px wide, keep content concise
