# Notekeeper API Release Notes

Sample documentation for a fictional product.

## v1.2.0 · September 15, 2026

This release adds tag filtering and archiving, replaces page-number pagination with cursors, and fixes two bugs. **One change affects existing integrations: the default page size is now 20.** See [Action required](#action-required).

### Action required

- **Default page size reduced from 50 to 20.** If you call `GET /notes` without `limit` and expect 50 notes per page, add `limit=50` to keep the same behavior.
- **Move from `page` to `cursor` before January 1, 2027.** The `page` and `per_page` parameters are deprecated (see below).

### New

- **Filter notes by tag.** `GET /notes` now accepts a `tag` parameter. Matching is case-insensitive.
- **Archive notes.** Notes have a new `archived` field, which defaults to `false`. Set it with `PATCH /notes/{note_id}`. Archived notes are hidden from `GET /notes` unless you pass `archived=true`.
- **Cursor pagination.** `GET /notes` accepts `cursor` and returns `next_cursor`. A `null` value means there are no more pages. See [How to retrieve all notes with a given tag](how-to-guide.md).

### Changed

- **Larger notes.** The maximum `body` length increased from 10,000 to 50,000 characters.
- **Default `limit` is now 20** (previously 50). The maximum stays at 100.

### Fixed

- `PATCH /notes/{note_id}` with `"tags": []` did not remove existing tags. It now clears all tags.
- Tags containing uppercase letters could create duplicate tags. Tags are now stored in lowercase.

### Deprecated

- The `page` and `per_page` parameters on `GET /notes` are deprecated and will be **removed on January 1, 2027**. Use `cursor` and `limit` instead.

---

## v1.1.0 · July 8, 2026

### New

- Added `PATCH /notes/{note_id}` for partial updates.
- Added the `X-RateLimit-Limit` and `X-RateLimit-Remaining` response headers.

### Fixed

- `DELETE /notes/{note_id}` returned `200` with a body. It now returns `204 No Content`.
