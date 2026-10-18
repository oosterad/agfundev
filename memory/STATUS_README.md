# status.json

The Memory app reads `status.json` on start and whenever it comes back to the foreground. It's
hosted here on GitHub Pages instead of on Heroku, so it can still be read when the backend is down.

- `status.json` is read by release builds.
- `status_test.json` is read by dev builds, so you can try changes without real users seeing them.

An edit reaches users as soon as the site is pushed: the app adds a query string to get past the
GitHub Pages cache. If the file can't be fetched, the app uses the last copy it fetched.

## Example

Both entries below ended on 2026-10-01, so the app shows nothing for this file.

```json
{
  "maintenance": {
    "start": "2026-09-30T22:00:00+02:00",
    "end": "2026-10-01T00:00:00+02:00",
    "message": {
      "en": "We're moving to a new server.",
      "nl": "We verhuizen naar een nieuwe server."
    }
  },
  "announcements": [
    {
      "id": "2026-09-update-42",
      "from": "2026-09-25T00:00:00+02:00",
      "until": "2026-10-01T00:00:00+02:00",
      "minBuild": 42,
      "maxBuild": 41,
      "action": "update",
      "title": {
        "en": "Update available",
        "nl": "Update beschikbaar"
      },
      "body": {
        "en": "A new version is in the Play Store.",
        "nl": "Er staat een nieuwe versie in de Play Store."
      }
    }
  ]
}
```

## maintenance

One object, not a list. Set it to `null` when there's no maintenance.

| Field     | Required | Meaning                                                             |
|-----------|----------|---------------------------------------------------------------------|
| `start`   | no       | When it starts. Without it, maintenance has already started.        |
| `end`     | no       | When it ends. Without it, it lasts until further notice. When it's  |
|           |          | set, the app also shows the end time.                               |
| `message` | no       | Text to show. Without it, the app shows "We're doing maintenance."  |

## announcements

A list. The app shows one announcement at a time: the first one that is live, not seen yet on the
device, and meant for its build. The next one comes up on a later fetch.

| Field      | Required | Meaning                                                            |
|------------|----------|--------------------------------------------------------------------|
| `id`       | yes      | Each device shows an announcement only once, so a new announcement |
|            |          | needs a new id.                                                    |
| `from`     | no       | Shown from this time. Without it, shown right away.                |
| `until`    | no       | Shown until this time. Without it, shown until further notice.     |
| `minBuild` | no       | Only for builds at or above this number.                           |
| `maxBuild` | no       | Only for builds at or below this number.                           |
| `action`   | no       | Only value is `"update"`: the button then says "Update" and opens  |
|            |          | the Play Store.                                                    |
| `title`    | no       | Heading. Without it, the app uses a default heading.               |
| `body`     | no       | Text of the announcement.                                          |

`minBuild` and `maxBuild` compare against the Android build number (the same number as the
backend's `minAndroidBuild`). In practice, you use one of them:

- `maxBuild`: show an "update available" message only on builds older than the new release.
- `minBuild`: show a message only on builds that have a new feature.

The example uses both with `minBuild` above `maxBuild`, so that announcement never shows. It's only
there to show both fields.

Builds from before `maxBuild` and `action` were added ignore both fields. They would show an update
announcement to everyone, with a plain OK button. Until every user has a build that knows these
fields, add a `minBuild` set to that build to update announcements.

## Notes

- Times are ISO 8601. Add the time zone offset (`+02:00` in summer, `+01:00` in winter). A date the
  app can't read means the entry never shows.
- `message`, `title` and `body` can be `{"en": ..., "nl": ...}` objects or plain strings. A missing
  language falls back to `en`.
- Use `\n` for a line break in a text.
- Put `**...**` around words in an announcement `body` to make them bold: `"Try **Falling tiles**"`.
  An unclosed `**` shows as is.
