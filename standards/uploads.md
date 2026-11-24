# Uploads

- **Status:** agreed 2026-11-16 (proposed by TEEHR, accepted by VeriRace). Both systems have uploads; this is the
  contract a client (CLI, SDK, web) can rely on, taken from what the two already share.

A user loads a file into a project's dataset in four steps. File bytes never pass through the API.

## Steps

| Step | Request | Response |
| --- | --- | --- |
| 1. Create | `POST /v1/projects/{project}/uploads` `{filename, kind, format?}` | 201, the upload with `upload_url` and `upload_url_expires_at` (shown once) |
| 2. Put | `PUT` the file to `upload_url` (a presigned URL for one object) | from object storage |
| 3. Inspect | `POST .../uploads/{id}/inspect` | the upload with its `preview` |
| 4. Ingest | `POST .../uploads/{id}/ingest` `{dataset, mapping}` | 202, the upload `queued`; a job loads it |

Then `GET .../uploads/{id}` until `status` is `succeeded`, `failed` or `cancelled`.
`GET .../uploads` lists a project's uploads, newest first. `POST .../uploads/{id}/cancel` stops a
load. `DELETE .../uploads/{id}` removes an upload and its staged file.

## Kinds and formats

| `kind` | Into | Formats |
| --- | --- | --- |
| `timeseries` | a timeseries dataset | `csv`, `parquet` |
| `layer` | a layer | `geojson`, `shapefile` (one `.zip`), `geopackage`, `parquet` (GeoParquet, or WKB/WKT columns), `csv` (longitude/latitude or WKT) |
| `crosswalk` | a crosswalk | `csv`, `parquet` |

`format` defaults from the file name's extension (`.csv`, `.parquet`, `.geojson` / `.json`, `.zip`,
`.gpkg`). `kind` is always given: nothing is guessed from the file.

## Status

`awaiting_file` → `inspected` → `queued` → `running` → `succeeded` | `failed` | `cancelled`.
An upload that `failed` or was `cancelled` can be inspected and ingested again.

## Inspect

Reads the start of the file (never all of it) and returns `preview`:

| Field | Meaning |
| --- | --- |
| `columns` | each column: `name`, `type`, `looks_like` (`integer`, `number`, `time`, `text`, `geometry`, `empty`), a few `sample` values |
| `sample` | the first rows |
| `rows` | the row count, or null when the format doesn't say without reading it all (CSV) |
| `mapping` | the proposed mapping (below) |
| `dataset` | the proposed dataset (at least a `name`, from the file name) |
| `needed` | what the uploader still has to give, as `mapping.<field>` / `dataset.<field>` |
| `notes` | what the uploader should know (times without a zone, a CRS that will be reprojected, ...) |
| `ingest` | an ingest request to edit: the proposal, with a `<...>` placeholder for each needed value |
| `layers` | a GeoPackage's layers (`?source_layer=` inspects another) |

CSV is read as text, so codes keep their leading zeros (`01010000`).

## Ingest

`{dataset, mapping}`: `mapping` changes the proposed one (a field left out keeps it); `dataset`
names an existing dataset of the upload's kind, or describes a new one.

- Every problem is reported at once: 422, with one `errors` entry per problem (`data-api.md`).
- A field that doesn't apply to the kind is a problem. So is a `<...>` placeholder left in place.
- The load checks the whole file before writing any of it; what it finds is the upload's
  `problems` (and `error`), and nothing is written.

### Mapping

Mapping fields are named for the column they fill.

| Kind | Fields |
| --- | --- |
| `timeseries` | `series_key` (a column) or `series_key_value` (one key for every row); `value_time`, `value`; `reference_time`, `member`, `quantile`, `approval_status`, `qualifier`; `location_key`; `layout`: `long` (default) or `wide` with `series_columns` (one value column per series, named by its key); `timezone`; `time_format` |
| `layer` | `location_key`, `name`, `geometry` or `longitude` + `latitude`, `crs`, `source_layer`, `attributes` (`[{column, name?, type?}]`; default every other column) |
| `crosswalk` | `left_key`, `right_key`, `attributes` (extra columns, as for layers) |

- **Times without a zone** need `timezone` (an IANA name); there's no default. `time_format`
  (`strptime`) reads times stored as text that aren't ISO 8601.
- **`location_key`** sets each series' location in the dataset's `location_layer`; it may be the
  series key column.
- **`crs`** is needed when the file doesn't say (a Shapefile without `.prj`, plain Parquet); any
  CRS other than EPSG:4326 is reprojected on load.

### Dataset

| Kind | New dataset | Existing |
| --- | --- | --- |
| `timeseries` | `name`, `role`, `variable`, `statistic`, `period`, `location_layer`, `time_partition`, `description`, `source`; `unit` (always: the unit of the file's values, converted on load) | `name`, `unit` |
| `layer` | `name`, `description`, `source` | `name` |
| `crosswalk` | `name`, `left`, `right` (`<name>` or `public.<name>`), `description`, `source` | `name` |

## Result

`result` on a succeeded upload: `dataset`, `created` (whether the load registered it), `rows`, and
for timeseries `series_added` (and `locations_set` with `location_key`). `notes` lists what was
loaded but may be wrong, e.g. crosswalk keys not in their side's series table. A system may add
fields of its own (e.g. `seconds`).

## Known differences (until each system conforms)

| | This standard | TEEHR today | VeriRace today |
| --- | --- | --- | --- |
| First status | `awaiting_file` | `staged` | as the standard |
| Cancel | `POST .../cancel`, status `cancelled` | as the standard | none |
| Layer formats | as above | as the standard | `csv`, `parquet`, `geojson` |
| Format from extension | as above | as the standard | timeseries also `.txt`, `.tsv`; no `.json`, `.gpkg` |
| Wide files, `time_format` | as above | none | as the standard |
| Times without a zone | `timezone` needed | as the standard | default `UTC` |
| Leading zeros | CSV read as text | as the standard | a layer CSV is type-sniffed (to be fixed) |
| Preview | `columns` with `looks_like` and `sample`; `needed`, `notes`, `ingest` | as the standard | `columns` as `{name, type}`; `needs` (bare names), `problems`, `warnings` |
| Ingest problems | 422 `errors` entries `{loc, msg, type}` | as the standard | plain strings (structured only for request validation) |
| Timeseries mapping | as above | as the standard | no `series_key_value`, `approval_status`, `qualifier`, `location_key` |
| Layer mapping | `location_key`, `name`, `longitude`, `latitude`, `source_layer`, `attributes` | as the standard | `key_column`, `lon`, `lat`; no `name`, `source_layer`; every other column an attribute |
| Crosswalk mapping | `attributes` | as the standard | none (every other column kept) |
| Crosswalk dataset sides | `left`, `right` | as the standard | `primary`, `secondary` |
| Series locations | `mapping.location_key` | as the standard | the series key, when the dataset has a layer |
| Dataset `source` | accepted | as the standard | none |
| Existing dataset, other unit | converted on load | as the standard | refused |
| Result | `dataset`, `series_added`, `notes` | as the standard | `series`; `layer` / `crosswalk` in place of `dataset`; crosswalk `primary_matched` / `secondary_matched` in place of `notes`; also `seconds` |

VeriRace conforms as one change set after this is agreed; TEEHR's changes are the "TEEHR today"
rows.
