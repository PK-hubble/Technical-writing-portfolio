# How to retrieve all notes with a given tag

**Applies to:** Notekeeper API v1.2 · Sample documentation for a fictional product

The list endpoint returns notes in pages, so a tag with many notes takes several requests to fetch. This guide shows how to follow the pagination cursor until you have every matching note.

**Time required:** about 10 minutes

## Before you begin

You need:

- A Notekeeper API key
- `curl`, or Python 3.8+ with the `requests` library (`pip install requests`)
- At least one note with the tag you want to retrieve

Store your key in an environment variable so it stays out of your code:

```bash
export NOTEKEEPER_API_KEY="YOUR_API_KEY"
```

## Step 1: Request the first page

Call `GET /notes` with the `tag` parameter. Set `limit` to `100`, the maximum, to reduce the number of requests.

```bash
curl "https://api.notekeeper.example/v1/notes?tag=work&limit=100" \
  -H "Authorization: Bearer $NOTEKEEPER_API_KEY"
```

The response contains a `data` array and a `next_cursor` value:

```json
{
  "data": [ { "id": "note_8f3k2a", "title": "Sprint retro", "tags": ["work", "retro"] } ],
  "next_cursor": "c_eyJvIjoxMDB9"
}
```

*(Example trimmed for length. Each item in `data` is a full note object.)*

## Step 2: Check `next_cursor`

- If `next_cursor` is a string, more notes are available.
- If `next_cursor` is `null`, this was the last page and you are done.

## Step 3: Request the next page

Pass the cursor value in the `cursor` parameter. Keep the same `tag` and `limit`.

```bash
curl "https://api.notekeeper.example/v1/notes?tag=work&limit=100&cursor=c_eyJvIjoxMDB9" \
  -H "Authorization: Bearer $NOTEKEEPER_API_KEY"
```

Repeat until `next_cursor` is `null`.

> **Note:** Use the cursor exactly as returned. Do not modify it or reuse one from a different query.

## Step 4: Automate the loop

This Python script performs Steps 1–3 and returns every matching note. It also waits and retries if it hits the rate limit.

```python
import os
import time
import requests

BASE_URL = "https://api.notekeeper.example/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['NOTEKEEPER_API_KEY']}"}


def fetch_notes_by_tag(tag):
    notes, cursor = [], None
    while True:
        params = {"tag": tag, "limit": 100}
        if cursor:
            params["cursor"] = cursor

        resp = requests.get(f"{BASE_URL}/notes", headers=HEADERS,
                            params=params, timeout=10)

        if resp.status_code == 429:  # rate limited: wait, then retry
            time.sleep(int(resp.headers.get("Retry-After", 1)))
            continue

        resp.raise_for_status()
        page = resp.json()
        notes.extend(page["data"])

        cursor = page["next_cursor"]
        if cursor is None:
            return notes


if __name__ == "__main__":
    results = fetch_notes_by_tag("work")
    print(f"Found {len(results)} notes tagged 'work'.")
```

**Expected output** (the count depends on your data):

```
Found 137 notes tagged 'work'.
```

## Troubleshooting

| Problem | Likely cause | Fix |
|---|---|---|
| `401 unauthorized` | The API key is missing, wrong, or the environment variable is not set. | Run `echo $NOTEKEEPER_API_KEY` to confirm it is set, then check the key. |
| `data` is empty on the first page | No notes have that tag, or the notes are archived. | Check the tag spelling. To search archived notes, add `archived=true`. |
| `429 rate_limited` | You exceeded 60 requests per minute. | Wait the number of seconds in the `Retry-After` header. The script above does this automatically. |
| `400 invalid_request` on a later page | The cursor was changed, or `tag` or `limit` differ from the previous request. | Reuse the exact cursor and the same query parameters. |

## Next steps

- See the [List notes](api-reference.md#list-notes) reference for all query parameters.
- Learn how to [update a note](api-reference.md#update-a-note) or archive it.
