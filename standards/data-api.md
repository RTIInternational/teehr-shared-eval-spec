# Data API: reading and errors

- **Status:** seeded 2026-11-15. The shared target; neither system implements all of it yet.
  VeriRace reads through `GET …/datasets/{name}/values` today and plans to add `/data`
  (see "Known differences").

## Reading a dataset

`GET .../datasets/{name}/data`

| Parameter | Meaning |
| --- | --- |
| `columns` | comma-separated columns to return |
| `filter` | an expression (`spec/expressions.md`) |
| `limit` | rows per page |
| `cursor` | opaque, from the previous page |
| `count` | `true` to include the total row count |

**Formats,** chosen by `Accept`:

| Media type | Body | Paging |
| --- | --- | --- |
| `application/json` (default) | `{rows, next_cursor, total}` | in the body |
| `application/vnd.apache.arrow.stream` | Arrow IPC stream | `X-Next-Cursor`, `X-Total-Rows` headers |
| `application/vnd.apache.parquet` | Parquet | same headers |

**Paging:**

- A cursor is opaque. It must not be parsed or built by clients.
- A null / absent `next_cursor` means the last page.
- All pages of one read see the same data: writes that land mid-read don't appear until a new
  read. Page order is stable.
- A cursor is only valid with the same `columns` and `filter` it was issued for.
- Page size caps: 10,000 rows for JSON, 1,000,000 for binary formats.

A system may add headers of its own (TEEHR sends `X-Snapshot-Id`).

## Errors

Every error is RFC 9457 `application/problem+json`:

```json
{"type": "about:blank", "title": "Not Found", "status": 404, "detail": "...", "code": "..."}
```

- `code`: optional, machine-readable, stable (e.g. `token_refresh_required`).
- `errors`: on 422, a list of the individual validation problems.
- `title` is the HTTP reason phrase; these are pinned so they don't vary by runtime:
  413 `Content Too Large`, 414 `URI Too Long`, 416 `Range Not Satisfiable`,
  422 `Unprocessable Content`.

## Known differences (until each system conforms)

| | This standard | VeriRace today |
| --- | --- | --- |
| Endpoint | `…/data` with `columns` and an expression `filter` | `…/values` with structured parameters (`series_key`, `start`, `end`, `reference_time`, `member`, `unit`, `limit`, `cursor`) |
| Paging headers | `X-Next-Cursor`, `X-Total-Rows` | `X-VeriRace-Next-Cursor`, `X-VeriRace-Total-Rows` |
| 422 `title` | `Unprocessable Content` | `Invalid request` |
| `code` member | optional | not sent yet |

## Open

- The shape of each entry in `errors`.
- A shared list of `code` values.
- Uploads (drop box → inspect → load job) once both systems have them.
