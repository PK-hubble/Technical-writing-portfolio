# Notekeeper API Reference

**Version 1.2** · Sample documentation for a fictional product

The Notekeeper API lets you create, read, update, and delete notes, and organize them with tags. It is a REST API that accepts and returns JSON.

**Base URL**

```
https://api.notekeeper.example/v1
```

## Authentication

Authenticate every request with an API key in the `Authorization` header.

```
Authorization: Bearer YOUR_API_KEY
```

Requests without a valid key return `401 unauthorized`. Keep your key secret and never include it in client-side code.

## Conventions

- Request bodies must be JSON with the header `Content-Type: application/json`.
- All timestamps are ISO 8601 in UTC, for example `2026-09-15T09:30:00Z`.
- Object IDs are opaque strings. Do not parse or assume a format.

## The Note object

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique identifier, for example `note_8f3k2a`. |
| `title` | string | Title of the note. 1–200 characters. |
| `body` | string | Note content in Markdown. Up to 50,000 characters. Empty string if not set. |
| `tags` | array of strings | Up to 10 tags. Each tag is 1–32 characters and stored in lowercase. |
| `archived` | boolean | `true` if the note is archived. Defaults to `false`. |
| `created_at` | string | When the note was created. |
| `updated_at` | string | When the note was last modified. |

```json
{
  "id": "note_8f3k2a",
  "title": "Sprint retro",
  "body": "What went well: fewer hotfixes.",
  "tags": ["work", "retro"],
  "archived": false,
  "created_at": "2026-09-15T09:30:00Z",
  "updated_at": "2026-09-15T09:30:00Z"
}
```

---

## Create a note

`POST /notes`

Creates a new note.

**Body parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `title` | string | Yes | Title of the note. |
| `body` | string | No | Note content in Markdown. |
| `tags` | array of strings | No | Tags to apply. |

**Example request**

```bash
curl -X POST https://api.notekeeper.example/v1/notes \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"title": "Sprint retro", "body": "What went well: fewer hotfixes.", "tags": ["work", "retro"]}'
```

**Response:** `201 Created` with the new [Note object](#the-note-object).

---

## Retrieve a note

`GET /notes/{note_id}`

Returns a single note.

**Path parameters**

| Parameter | Type | Description |
|---|---|---|
| `note_id` | string | The ID of the note to retrieve. |

**Example request**

```bash
curl https://api.notekeeper.example/v1/notes/note_8f3k2a \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Response:** `200 OK` with the [Note object](#the-note-object). Returns `404 not_found` if the note does not exist.

---

## List notes

`GET /notes`

Returns a paginated list of notes, newest first. Archived notes are excluded unless you set `archived=true`.

**Query parameters**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `tag` | string | none | Return only notes with this tag. Matching is case-insensitive. |
| `archived` | boolean | `false` | Set to `true` to return archived notes instead. |
| `limit` | integer | `20` | Notes per page. Minimum 1, maximum 100. |
| `cursor` | string | none | Pagination cursor from a previous response's `next_cursor`. |

**Example request**

```bash
curl "https://api.notekeeper.example/v1/notes?tag=work&limit=2" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Example response**

```json
{
  "data": [
    { "id": "note_8f3k2a", "title": "Sprint retro", "body": "What went well: fewer hotfixes.", "tags": ["work", "retro"], "archived": false, "created_at": "2026-09-15T09:30:00Z", "updated_at": "2026-09-15T09:30:00Z" },
    { "id": "note_2b9x1q", "title": "Roadmap draft", "body": "", "tags": ["work"], "archived": false, "created_at": "2026-09-14T16:05:00Z", "updated_at": "2026-09-14T16:05:00Z" }
  ],
  "next_cursor": "c_eyJvIjoyfQ"
}
```

**Response fields**

| Field | Type | Description |
|---|---|---|
| `data` | array | Notes on this page. |
| `next_cursor` | string or null | Pass as `cursor` to get the next page. `null` means there are no more pages. |

---

## Update a note

`PATCH /notes/{note_id}`

Updates only the fields you send. Omitted fields are left unchanged.

**Body parameters**

| Parameter | Type | Description |
|---|---|---|
| `title` | string | New title. |
| `body` | string | New content. Replaces the existing body. |
| `tags` | array of strings | New tags. **Replaces the entire tag list.** Send `[]` to remove all tags. |
| `archived` | boolean | Archive or unarchive the note. |

**Example request**

```bash
curl -X PATCH https://api.notekeeper.example/v1/notes/note_8f3k2a \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"archived": true}'
```

**Response:** `200 OK` with the updated [Note object](#the-note-object).

---

## Delete a note

`DELETE /notes/{note_id}`

Permanently deletes a note. This cannot be undone. To keep a note but hide it from default lists, [archive it](#update-a-note) instead.

**Response:** `204 No Content` with an empty body.

---

## Errors

Errors use standard HTTP status codes and return a JSON body:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "title is required.",
    "param": "title"
  }
}
```

`param` is included when the error relates to a specific parameter.

| Status | `code` | Meaning |
|---|---|---|
| 400 | `invalid_request` | A parameter is missing, malformed, or out of range. |
| 401 | `unauthorized` | The API key is missing or invalid. |
| 404 | `not_found` | The requested note does not exist. |
| 429 | `rate_limited` | Too many requests. See [Rate limits](#rate-limits). |
| 500 | `server_error` | Something went wrong on our side. Retry after a short delay. |

## Rate limits

Each API key can make **60 requests per minute**. Every response includes these headers:

| Header | Description |
|---|---|
| `X-RateLimit-Limit` | Your per-minute request limit. |
| `X-RateLimit-Remaining` | Requests left in the current window. |
| `Retry-After` | Seconds to wait before retrying. Sent only with `429` responses. |
