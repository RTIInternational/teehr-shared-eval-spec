# Data API: reading and errors

- **Status:** seeded 2026-11-15. The shared target. TEEHR implements it, except that its `filter`
  uses PyIceberg's expression syntax rather than `spec/expressions.md`. VeriRace reads through
  `GET …/datasets/{name}/values` today and plans to add `/data`. See "Known differences".

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

## Layers

A layer's geometry is WKB in EPSG:4326 (longitude, latitude), in a column named `geometry`:

- **Arrow:** a binary column marked as GeoArrow WKB (`ARROW:extension:name` `geoarrow.wkb`).
- **Parquet:** GeoParquet 1.1 metadata (`geo`), primary column `geometry`; no `crs` means
  OGC:CRS84.
- **JSON:** each row's `geometry` is a GeoJSON geometry object.
- **`Accept: application/geo+json`:** a FeatureCollection, each feature's `id` its `location_key`;
  paging in the `X-Next-Cursor` / `X-Total-Rows` headers.
- **`bbox=minx,miny,maxx,maxy`** (layers only): the locations whose bounds overlap it.

## Errors

Every error is RFC 9457 `application/problem+json`:

```json
{"type": "about:blank", "title": "Not Found", "status": 404, "detail": "...", "code": "..."}
```

- `code`: optional, machine-readable, stable (e.g. `token_refresh_required`).
- `errors`: on 422, one entry per problem, all of them at once:
  `{"loc": [...], "msg": "...", "type": "..."}`. `loc` is the path to the problem as strings, its
  first item where it is (`body`, `query`, `path`): `["body", "mapping", "timezone"]`.
- Shared `code` values:

  | `code` | Status | Meaning |
  | --- | --- | --- |
  | `token_refresh_required` | 403 | the access token predates a role the caller now has; get a new one and retry |
- `title` is the HTTP reason phrase; these are pinned so they don't vary by runtime:
  413 `Content Too Large`, 414 `URI Too Long`, 416 `Range Not Satisfiable`,
  422 `Unprocessable Content`.

## Known differences (until each system conforms)

| | This standard | TEEHR today | VeriRace today |
| --- | --- | --- | --- |
| Endpoint | `…/data` with `columns` and an expression `filter` | as the standard | `…/values` with structured parameters (`series_key`, `start`, `end`, `reference_time`, `member`, `unit`, `limit`, `cursor`) |
| `filter` syntax | `spec/expressions.md` | PyIceberg expressions (moving to `spec/expressions.md`) | no expression filter |
| Paging headers | `X-Next-Cursor`, `X-Total-Rows` | as the standard | `X-VeriRace-Next-Cursor`, `X-VeriRace-Total-Rows` |
| 422 `title` | `Unprocessable Content` | as the standard | `Invalid request` |
| `code` member | optional | sent | not sent yet |
| Layer geometry in Arrow | `geoarrow.wkb` extension | plain binary | as the standard |
| GeoJSON, `bbox` | as above | as the standard | not yet |

## Open

- More shared `code` values as they come up.
