# Periods (temporal extent)

- **Status:** agreed 2026-11-11 (TEEHR ADR-0012, encoding developed jointly).

A value's **period** is the span of time it covers. It is separate from:

- **issuance**: when a forecast was made (`reference_time`);
- **`value_time`**: the time a value is reported at;
- **`statistic`**: how the value reduces over its period (`inst`, `mean`, `sum`, `min`, `max`);
- **resolution**: how often values recur (descriptive only).

Streamflow *hourly instantaneous* is period `instant`, resolution hourly. Precipitation *hourly
accumulation* is period `hour_ending` (or `hour_starting`), resolution hourly.

## Definitions table

Periods are rows referenced by name, never parsed:

| Column | Type | Used by form | Meaning |
| --- | --- | --- | --- |
| `name` | text | all | unique key, e.g. `instant`, `hour_starting`, `apr_jul_volume` |
| `form` | text | all | `instant` / `duration` / `calendar` / `absolute` |
| `length` | ISO-8601 duration | duration | how long each value covers |
| `value_time_at` | `start` / `end` | duration | whether `value_time` is the window's start or end |
| `start`, `end` | `--MM-DD` | calendar | first and last day of the span, both inclusive |
| `description` | text | all | free text |

## Resolving to a window

- `instant` → `[value_time, value_time]`
- `duration` → `[value_time, value_time + length)` when `value_time_at = start`, or
  `[value_time − length, value_time)` when `end`
- `calendar` → the occurrence of `[start 00:00, day-after-end 00:00)` that contains `value_time`
- `absolute` → the value's own `period_start` / `period_end`

## Rules

1. **One period per series.** Another coverage is another series.
2. **Calendar occurrence:** a calendar value belongs to the occurrence of its span that contains
   `value_time`. No separate water-year convention; spans crossing Jan 1 work the same way.
3. **Inclusive month-day ends, half-open windows:** `--04-01` … `--07-31` covers Apr 1 00:00 up to
   Aug 1 00:00.
4. **Default and validation:** `instant` is the default for `inst`; any other statistic must name a
   period. `instant` with a non-`inst` statistic, or a non-`instant` period with `inst`, is an
   error.
5. **Row checks:** a row fills exactly the columns its form uses; durations must parse; month-days
   must exist.
6. **Only `absolute` adds per-row columns** (`period_start`, `period_end`); other forms derive the
   window on read.
7. **Resolved windows are exposed** as `period_start` / `period_end` wherever values are read or
   paired.

## Shared rows

`periods.csv` holds the rows that ship fixed in both systems. Users add their own beyond these.

## Pairing

Two values pair when their series are linked, their statistics are equal, and their resolved
windows are equal. Different-length windows (daily mean vs hourly values) need an explicit
aggregation first; pairing never aggregates. An observation is complete at `period_end`.

## User-defined periods

A user-defined period travels with its dataset. A spec that names a period itself carries the
definition inline in a `periods:` block (same columns). A name that collides with a different
definition on the receiving side is rejected, never overridden.

## Open

- Whether `resolution` is stored or always derived from `value_time` spacing.
