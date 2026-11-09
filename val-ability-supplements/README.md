# Ability spatial supplement

This supplement adds precise caster position, aim and movement at the moment each ability is used. It is meant to be used alongside the GRID series feed.

There is one file per series, `{seriesId}.jsonl.zst`, where the number is the GRID series ID. Each file is zstd-compressed JSON Lines (see [Opening `.zst` files](../README.md#opening-zst-files)). Each line is one ability cast, and lines are sorted by `occurred_at`.

The matching GRID feed is `../val/events_{seriesId}_grid.jsonl.zst`. [`../val/series.csv`](../val/series.csv) lists which series have a supplement. One downloaded series (2855902) has none, because none of its maps could be matched to a GRID game.

## Joining to GRID

- **`grid_event_id`** is the `events[].id` of the matching GRID `player-used-ability` event.
- **`occurred_at`** is identical to that GRID envelope's `occurredAt`.
- **`player_id`** and **`game_id`** are GRID IDs. Look up the player's name, team, side and agent in the GRID feed, along with the map and round.

Some casts are not in the GRID feed. They have `in_grid: false` and `grid_event_id: null`. These are mostly signature abilities used between rounds or during the buy phase. Filter on `in_grid` if you only want casts that appear in GRID.

## Fields

| Field | Type | Description |
|---|---|---|
| `series_id` | string | GRID series ID |
| `game_id` | string | GRID game ID |
| `grid_event_id` | string \| null | GRID `player-used-ability` event ID |
| `in_grid` | bool | Whether GRID reports this cast |
| `occurred_at` | string | UTC timestamp of the cast, with millisecond precision |
| `player_id` | string | GRID player ID of the caster |
| `ability` | string \| null | GRID ability ID, e.g. `toxic-screen` |
| `slot` | string | `ABILITY_1`, `ABILITY_2`, `GRENADE` or `ULTIMATE` |
| `charges_consumed` | int | Number of charges used by this cast |
| `frame_before` | frame \| null | The closest movement sample at or before the cast |
| `frame_after` | frame \| null | The closest movement sample after the cast |
| `interpolated` | object \| null | The caster's state interpolated to the exact moment of the cast |
| `quality` | string | See [Quality](#quality) |

**Frame:** Movement samples arrive about every 200 ms (5 Hz). Occasionally an extra sample sits between them; when one is closer to the cast, it is used. Some series only have one sample per second; see [Low-rate series](#low-rate-series).

```json
{"offset_ms": -54, "position": {"x":..,"y":..,"z":..}, "view": {"x":..,"y":..,"z":..}, "velocity": {"x":..,"y":..,"z":..}}
```

- `offset_ms` is the frame's time relative to the cast. It is negative for a frame before the cast.
- `position` is in game units, where 1 unit = 1 cm. It uses the same x/y frame as GRID positions, and `z` is up.
- `view` is the unit aim direction.
- `velocity` is in cm/s.

**`interpolated`** holds `weight`, `position`, `view` and `velocity`:

```
weight = (t_cast - t_before) / (t_after - t_before)
value  = before * (1 - weight) + after * weight
```

The formula is applied per component, and `view` is then re-normalized to unit length. For `missing_after` records, `interpolated` may be extrapolated instead and `weight` is then null; see [Quality](#quality).

## Quality

| Value | Meaning |
|---|---|
| `ok` | Both frames are present and each is within 500 ms of the cast (1200 ms in [low-rate series](#low-rate-series)) |
| `gap_exceeded` | Both frames are present, but one is more than 500 ms (low-rate: 1200 ms) from the cast. This typically happens when the caster died right after casting, so the next frame comes from the next round. Don't use `interpolated`; use whichever frame has the small `offset_ms` instead. |
| `missing_after` | No frame after the cast. If `frame_before` is within 500 ms (low-rate: 1200 ms), `interpolated` is extrapolated from it: `position = frame_before.position + frame_before.velocity × Δt` (Δt = −`frame_before.offset_ms` / 1000 s), with `view` and `velocity` held from `frame_before`, and `weight` is null. Beyond that limit, `interpolated` copies `frame_before` and `weight` is 0. |
| `missing_before` | No frame before the cast, and `interpolated` copies `frame_after` |
| `missing_both` | No frames exist, and `interpolated` is null |

## Low-rate series

The VCT Americas 2025 series (Kickoff, Stage 1 and Stage 2) have movement samples only once per second (1 Hz) instead of 5 Hz. For these series the distance limit used for `quality` is loosened from 500 ms to 1200 ms, so that a cast between two normal samples is still `ok`. Without this, almost every cast in these series would be `gap_exceeded`.

The limit is set per game: it is 1.2 × the game's median sample interval, but never less than 500 ms. All other series are 5 Hz and keep the 500 ms limit.

In these series a frame can be up to about 1 s from the cast, so `interpolated` is much less precise: a running player covers several metres in a second, and quick aim changes between samples are lost. `missing_after` extrapolation can also span up to 1.2 s. If you need precise positions, filter these series out or weight them lower. You can recognise them by `frame_after.offset_ms - frame_before.offset_ms` being about 1000 rather than about 200.

## Caveats

- **Landing locations are not provided.** No feed records where an ability lands. You need to estimate it from cast position, view direction and velocity, e.g. by projecting the aim ray, or with a per-ability throw model fitted to damage events in GRID.
- **Precision.** GRID's own `position` on ability events is 2D and can lag the true cast position by several metres. Prefer the `interpolated` position from this supplement.
- **A few maps are left out.** Casts are only written for maps that can be matched to a GRID game. These are left out:
  - maps that GRID doesn't have, such as Bind in series 2855906 and Split in 2877148;
  - maps that GRID merged into the previous game, such as Abyss in 2890334 and Split in 2890361, where GRID game 1 also carries map 2's events.

  The other maps of these series are included.
