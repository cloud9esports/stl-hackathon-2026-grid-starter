# Sky's The Limit 2026: Cloud9 x JetBrains Hackathon — GRID Data Starter

This repository contains the official match data for the **Sky's The Limit 2026 – Cloud9 x JetBrains Hackathon**. It holds professional **VALORANT** and **League of Legends** matches recorded event by event by [GRID](https://grid.gg), the official esports data platform. For VALORANT there is also a supplement that gives each ability cast's precise position and aim.

Every match (a GRID *series*, such as a best-of-3) is a single compressed [JSON Lines](https://jsonlines.org) file. Pick a match, stream its events, and build something.

> **License:** the data is proprietary and licensed **only for building your hackathon project, during the hackathon**. No redistribution and no commercial use. See [LICENSE](LICENSE.md) and [License in plain words](#license-in-plain-words).
>
> **Never commit these data files to your own project repository.** Pushing them is redistribution, even by accident. See [Keep the data out of your project repo](#keep-the-data-out-of-your-project-repo).

**Using an AI assistant or coding agent?** You're encouraged to. Point it at this README and at [AGENTS.md](AGENTS.md), which has the rules and facts it needs, written for agents.

---

## Contents

- [What's inside](#whats-inside)
- [Quick start](#quick-start)
- [Keep the data out of your project repo](#keep-the-data-out-of-your-project-repo)
- [Opening `.zst` files](#opening-zst-files)
- [Repository layout](#repository-layout)
- [Metadata: teams, players, tournaments](#metadata-teams-players-tournaments)
- [The GRID event format](#the-grid-event-format)
- [VALORANT ability supplement](#valorant-ability-supplement)
- [Recipes](#recipes)
- [Data caveats](#data-caveats)
- [For AI agents → AGENTS.md](AGENTS.md)
- [License in plain words](#license-in-plain-words)
- [Coverage by event](#coverage-by-event)

---

## What's inside

<!-- COVERAGE:START -->
| Folder | Game | What | Files |
|---|---|---|---|
| [`val/`](val/) | VALORANT | GRID series event feeds | 857 |
| [`lol/`](lol/) | League of Legends | GRID series event feeds | 745 |
| [`val-ability-supplements/`](val-ability-supplements/) | VALORANT | Ability-cast position/aim supplement | 856 |
| [`meta/`](meta/) | both | Team, player, tournament, character, game and roster tables (CSV) | 6 |
<!-- COVERAGE:END -->

The data covers the 2025 and 2026 seasons. **VALORANT:** VCT Americas, VALORANT Masters, and VCT Game Changers (North America, Brazil, LATAM, EMEA, Japan, Korea, Pacific, SEA and Oceania, plus the Game Changers Championship). **League of Legends:** LCK, LEC, LTA North/South and LCS. Each game folder's `series.csv` lists every match with its event, stage, date and teams. [Coverage by event](#coverage-by-event) has per-event counts.

Cloud9's teams appear as **Cloud9** (VCT Americas), **Cloud9 GC** (VALORANT Game Changers) and **Cloud9 Kia** (LCS).

---

## Quick start

**1. Get the data.** The full repository is about 1.4 GB (no file is over 3 MB). You can clone all of it:

```bash
git clone https://github.com/cloud9esports/stl-hackathon-2026-grid-starter.git
```

or just one game, using a sparse checkout:

```bash
git clone --filter=blob:none --sparse https://github.com/cloud9esports/stl-hackathon-2026-grid-starter.git
cd stl-hackathon-2026-grid-starter
git sparse-checkout set val val-ability-supplements   # or: lol
```

**2. Find a match.** Open `val/series.csv` or `lol/series.csv` in any spreadsheet or editor:

```csv
series_id,event,stage,scheduled_start,teams,file,ability_supplement
2748743,VCT Americas - Kickoff 2025,Knockouts,2025-01-16T22:00:00Z,Evil Geniuses (1) vs LOUD,val/events_2748743_grid.jsonl.zst,val-ability-supplements/2748743.jsonl.zst
```

**3. Read it.** With Python 3.14 or newer, zstd support is built in:

```python
import json
from compression import zstd          # Python 3.14+ standard library

with zstd.open("val/events_2748743_grid.jsonl.zst", "rt", encoding="utf-8") as f:
    for line in f:
        envelope = json.loads(line)
        for event in envelope["events"]:
            print(envelope["occurredAt"], event["type"])
```

On Python 3.13 or older, `pip install zstandard` and swap one line:

```python
import zstandard
with zstandard.open("val/events_2748743_grid.jsonl.zst", "rt", encoding="utf-8") as f:
    ...
```

---

## Keep the data out of your project repo

Committing any of this data to your project's repository, and pushing it to GitHub or anywhere else, **redistributes it**, which the [LICENSE](LICENSE.md) prohibits. This applies even to a private repository: repositories are often shared or made public for judging, and git history keeps every file that was ever committed. Judges get the data from this repository, so your project doesn't need to include it.

**1. Clone this repository next to your project, not inside it,** and point your code at it:

```
hackathon/
├── stl-hackathon-2026-grid-starter/   ← this data repo (never commit it)
└── my-project/                        ← your project repo
```

```python
import os, pathlib
DATA_DIR = pathlib.Path(os.environ.get("STL_DATA_DIR", "../stl-hackathon-2026-grid-starter"))
feed = DATA_DIR / "val" / "events_2748743_grid.jsonl.zst"
```

**2. Add this to your project's `.gitignore`** anyway, in case data ends up in your project folder:

```gitignore
# Sky's The Limit 2026 hackathon data: licensed, must not be committed
stl-hackathon-2026-grid-starter/
*.zst
*.jsonl
# Decompressed copies or extracts built from the data (adjust to your project)
*.parquet
*.sqlite
*.duckdb
```

This also covers files **derived** from the data that contain it: decompressed `.jsonl` files, CSV or Parquet extracts, databases, caches and copies of `meta/`. Keep those out of git too. Your code, charts, screenshots and aggregate results are fine to commit.

**3. Check before you push:**

```bash
git ls-files | grep -Ei '\.(zst|jsonl|parquet|sqlite|duckdb)$|stl-hackathon-2026-grid-starter'   # should print nothing
```

If data was already committed, deleting it in a new commit is **not** enough, because it stays in history. Remove it from history (for example with [`git filter-repo`](https://github.com/newren/git-filter-repo)) before pushing. If you have already pushed, rewrite and force-push, then let the organizers know.

---

## Opening `.zst` files

`.zst` is [Zstandard](https://facebook.github.io/zstd/), a modern compression format, like `.zip` or `.gz` but faster and smaller. Each `.zst` file here holds exactly **one** `.jsonl` file. It's a single compressed file, not an archive or folder.

**You usually don't need to decompress anything to disk.** Most languages can read `.zst` as a stream (see [Quick start](#quick-start)). That's the recommended approach, because the data is **50–150× larger uncompressed**: roughly 160 GB for VALORANT alone, with single matches up to about 1 GB.

If you want to look at a file by hand:

| Platform | Install | Decompress one file | Peek without decompressing |
|---|---|---|---|
| macOS | `brew install zstd` | `zstd -d file.jsonl.zst` | `zstdcat file.jsonl.zst \| head -n 1` |
| Linux | `sudo apt install zstd` (or `dnf`, `pacman`) | `zstd -d file.jsonl.zst` | `zstdcat file.jsonl.zst \| head -n 1` |
| Windows | Download `zstd` from the [official releases](https://github.com/facebook/zstd/releases) (the `win64` zip), or use the [PeaZip](https://peazip.github.io) GUI | `zstd -d file.jsonl.zst` | `zstd -dc file.jsonl.zst \| more` |

`zstd -d` writes `file.jsonl` next to the original and keeps the `.zst`. Add `--rm` to delete the compressed copy, or `-o other.jsonl` to choose the output name.

**Other languages:** any Zstandard library works, since the files use standard settings and need no special flags. Language-neutral fallback: pipe the decompressed stream into your program, for example `zstd -dc file.jsonl.zst | node script.js`, and read JSON Lines from standard input.

---

## Repository layout

```
.
├── README.md                      ← you are here
├── AGENTS.md                      ← rules and facts for AI coding agents
├── LICENSE.md                     ← proprietary hackathon data license
├── val/
│   ├── series.csv                 ← index of every VALORANT match
│   └── events_{seriesId}_grid.jsonl.zst
├── lol/
│   ├── series.csv                 ← index of every League of Legends match
│   └── events_{seriesId}_grid.jsonl.zst
├── val-ability-supplements/
│   ├── README.md                  ← full supplement schema
│   └── {seriesId}.jsonl.zst
└── meta/                          ← lookup tables for both games (plain CSV)
    ├── tournaments.csv
    ├── teams.csv
    ├── players.csv
    ├── characters.csv
    ├── games.csv
    └── rosters.csv
```

**Terminology:** a **series** is a match between two teams (for example a best-of-3) and is one file. A **game** is one map within it: a VALORANT map, or one game on Summoner's Rift in League of Legends.

**`series.csv` columns**

| Column | Example | Meaning |
|---|---|---|
| `series_id` | `2748743` | GRID series ID, the same number as in the file name and in every event's `seriesId` |
| `event` | `VCT Americas - Kickoff 2025` | Event (tournament) name |
| `stage` | `Knockouts` | Stage within the event, such as `Regular Season`, `Swiss` or `Playoffs` |
| `scheduled_start` | `2025-01-16T22:00:00Z` | Scheduled start in UTC. The actual start is in the feed. |
| `teams` | `Evil Geniuses (1) vs LOUD` | Team names as GRID records them |
| `file` | `val/events_2748743_grid.jsonl.zst` | Path to the GRID event feed, relative to the repo root |
| `ability_supplement` | `val-ability-supplements/2748743.jsonl.zst` | VALORANT only: path to the supplement, empty if there is none |

---

## Metadata: teams, players, tournaments

`meta/` holds small CSV lookup tables, built from the final state of every feed in this repository, so every ID matches the data exactly. Use them to find matches and to put names to IDs without opening any `.zst` file. Every table has a `title` column (`val` or `lol`). IDs are unique only within a title, so join on `(title, id)`.

| File | One row per | Key columns |
|---|---|---|
| `tournaments.csv` | GRID tournament stage | `tournament_id`, `tournament_name`, `event`, `event_tournament_id` (the parent event), `stage`, `series_count`, `first_scheduled`, `last_scheduled` |
| `teams.csv` | team | `team_id`, `name` (most recent), `other_names` (earlier names, `; `-separated), `series_count`, `events`, `first_seen`, `last_seen` |
| `players.csv` | player | `player_id`, `name` (in-game name, most recent), `other_names`, `latest_team_id`, `latest_team_name`, `team_ids` (every team, oldest first), `series_count`, `first_seen`, `last_seen` |
| `characters.csv` | VALORANT agent / LoL champion | `character_id` (the ID used in the feeds), `name`, `games_played` |
| `games.csv` | game (map) | `series_id`, `game_id`, `game_number`, `map`, `patch`, `started_at`, `finished`, `winner_team_id`, `team{1,2}_id/name/side/score` |
| `rosters.csv` | player in a game | `series_id`, `game_id`, `game_number`, `team_id`, `team_name`, `side`, `player_id`, `player_name`, `character_id`, `character_name`, `participation_status` |

```python
import csv
games = list(csv.DictReader(open("meta/games.csv", encoding="utf-8")))
c9_maps = [g for g in games if g["title"] == "val" and "Cloud9" in (g["team1_name"] + g["team2_name"])]
```

Player names are in-game nicknames only. The tables deliberately contain no real names or account identifiers.

---

## The GRID event format

A file is the complete GRID event feed for one series, from `grid-started-feed` to `grid-ended-feed`. Each **line** is a JSON **envelope**, and envelopes are in `sequenceNumber` order. Each envelope carries one or more **events** that happened at the same moment.

```jsonc
{
  "id": "49ccbcb3-…",              // envelope ID
  "correlationId": "d1ef0926-…",
  "occurredAt": "2025-02-09T20:19:00.969Z",   // UTC, millisecond precision
  "seriesId": "2767056",
  "sequenceNumber": 400,           // strictly increasing within the file
  "events": [
    {
      "id": "…",                   // event ID
      "type": "player-killed-player",          // always "{actor.type}-{action}-{target.type}"
      "actor":  { "type": "player", "id": "…", "state": { … }, "stateDelta": { … } },
      "action": "killed",
      "target": { "type": "player", "id": "…", "state": { … }, "stateDelta": { … } },
      "seriesState":      { … },   // full snapshot of the series AFTER this event
      "seriesStateDelta": { … },   // only what this event changed
      "includesFullState": false
    }
  ]
}
```

**Key ideas**

- **Event types read as sentences:** `{actor}-{action}-{target}`, for example `player-used-ability`, `team-won-round`, `player-destroyed-tower` or `series-ended-game`. A `!` before the action **reverses an earlier event**. For example, `player-!purchased-item` is an undone purchase, and `series-!started-game` rolls back a game start. Undo the matching event when you replay a feed.
- **`seriesState` is a full snapshot.** It holds the series (`teams`, `games`, `format`, `started`, `finished`, …), then each game (`map`, `clock`, `teams`, `segments` such as VALORANT rounds, `draftActions`, …), each team in a game (`name`, `side`, `score`, `won`, `kills`, `money`, …), and each player (`name`, `character`, `kills`, `deaths`, `inventory`, `abilities`, `position`, `alive`, `currentHealth`, …). **The last event in a file holds the final box score.** Use `seriesStateDelta` (and `actor`/`target` `stateDelta`) if you only care about what changed.
- **IDs are strings.** Player, team and game IDs are stable across the series, so join on them, not on names.
- **Positions** are `{x, y}` in game units: centimetres in VALORANT, Summoner's Rift units in League of Legends. Each game's `map.bounds` gives the playable area.
- **Draft (League of Legends):** picks and bans are `team-picked-character` and `team-banned-character` events, also in `draftActions`.

**Common event types**

| VALORANT | League of Legends |
|---|---|
| `player-used-ability`, `player-damaged-player`, `player-killed-player` | `player-purchased-item`, `player-used-ability`, `player-completed-increaseLevel` |
| `game-started-round`, `team-won-round`, `game-ended-round` | `player-killed-player`, `game-respawned-player`, `player-multikilled-player` |
| `round-started-freezetime`, `round-ended-freezetime` | `player-destroyed-tower`, `player-destroyed-fortifier`, `player-destroyed-base` |
| `player-completed-plantBomb`, `player-completed-defuseBomb`, `player-completed-explodeBomb` | `player-completed-slayDragon` (and each drake type), `player-completed-slayBaron`, `player-completed-slayVoidGrub`, `player-completed-slayRiftHerald` |
| `player-pickedUp-item`, `player-dropped-item`, `player-completed-captureUltimateOrb` | `team-picked-character`, `team-banned-character` |
| `series-started-game`, `team-won-game`, `series-ended-game`, `team-won-series` | `series-started-game`, `team-won-game`, `series-ended-game`, `team-won-series` |

To see every type in a file, run the "count event types" recipe below.

---

## VALORANT ability supplement

GRID records *that* an ability was used. The supplement adds *where from and in which direction*: for each cast, the caster's 3D position, aim direction and velocity at the exact moment, interpolated from movement samples taken about 5 times a second. It joins to the GRID feed on `grid_event_id`, which equals the `events[].id` of the GRID `player-used-ability` event.

```jsonc
{"series_id": "2748743", "game_id": "…", "grid_event_id": "…", "in_grid": true,
 "occurred_at": "2025-01-16T22:14:03.512Z", "player_id": "…", "ability": "toxic-screen", "slot": "ABILITY_2",
 "interpolated": {"position": {"x": …, "y": …, "z": …}, "view": {"x": …, "y": …, "z": …}, "velocity": {…}, "weight": 0.27},
 "frame_before": {…}, "frame_after": {…}, "quality": "ok", "charges_consumed": 1}
```

The full schema, quality flags and caveats are in [`val-ability-supplements/README.md`](val-ability-supplements/README.md).

---

## Recipes

All recipes are Python 3.14+. On older versions, replace `from compression import zstd` with `import zstandard as zstd` after `pip install zstandard`.

**Stream any file**

```python
import json
from compression import zstd

def read_jsonl_zst(path):
    with zstd.open(path, "rt", encoding="utf-8") as f:
        for line in f:
            yield json.loads(line)
```

**Count event types in a match**

```python
from collections import Counter
counts = Counter(e["type"] for env in read_jsonl_zst("lol/events_2754338_grid.jsonl.zst") for e in env["events"])
print(counts.most_common(20))
```

**Final result of every game in a match** (from the last snapshot)

```python
last = None
for env in read_jsonl_zst("val/events_2748743_grid.jsonl.zst"):
    for e in env["events"]:
        last = e.get("seriesState") or last
for game in last["games"]:
    print(game["map"]["name"], {t["name"]: t["score"] for t in game["teams"]})
```

**All kills with killer, victim and time**

```python
for env in read_jsonl_zst("val/events_2748743_grid.jsonl.zst"):
    for e in env["events"]:
        if e["type"] == "player-killed-player":
            print(env["occurredAt"], e["actor"]["state"]["name"], "→", e["target"]["state"]["name"])
```

**Every Cloud9 match**

```python
import csv
with open("val/series.csv", newline="", encoding="utf-8") as f:
    c9 = [r for r in csv.DictReader(f) if "Cloud9" in r["teams"]]
```

**Join ability casts to their GRID events**

```python
sid = "2748743"
casts = {c["grid_event_id"]: c for c in read_jsonl_zst(f"val-ability-supplements/{sid}.jsonl.zst") if c["in_grid"]}
for env in read_jsonl_zst(f"val/events_{sid}_grid.jsonl.zst"):
    for e in env["events"]:
        if e["type"] == "player-used-ability" and e["id"] in casts:
            pos = casts[e["id"]]["interpolated"]["position"]
            ...
```

---

## Data caveats

- **Not every scheduled match is here.** Matches that GRID has no feed for (cancelled, not covered, or placeholders such as `TBD-1 vs TBD-2`) aren't included, so a bracket can have gaps.
- **Restarted and abandoned maps.** A few VALORANT series contain a map that was started, abandoned after a technical issue, and replayed. The abandoned attempt's events stay in the stream and are then rolled back by `series-!started-game` for that game ID. The replay is a new game with a new ID. The final `seriesState` and `meta/games.csv` list only the games actually played. If you process events one by one, drop events for game IDs that were rolled back.
- **Occasional feed gaps.** A small number of series are missing a game or have no `team-won-series` event. Derive results from the per-game state rather than assuming the series feed is complete.
- **Low-rate movement.** In VCT Americas 2025, VALORANT positions are sampled once per second instead of about five times. The supplement flags this; see its README.
- **Team names** are recorded as broadcast and can change between events (for example `MIBR (2)`), and two different teams can share a name (for example two VALORANT teams called `Shopify Rebellion Gold`). Use team IDs to follow a team over time; `meta/teams.csv` lists each team's earlier names.
- **One invalidated series.** GRID invalidated VALORANT series `2855902` (Game Changers Pacific 2025). Its feed holds only a rolled-back game start and no match data, so it has no rows in `meta/games.csv` and no ability supplement.
- **Times are UTC.**

---

## License in plain words

This summary is not a substitute for the [LICENSE](LICENSE.md), which is what applies.

- ✅ Use the data to build, demo and submit your hackathon project, during the hackathon.
- ✅ Share it with your own team members, and show results, charts and demos to the judges.
- ✅ Use AI assistants and coding agents on it. If a tool lets you opt out of training on or retaining your inputs, you must turn that on.
- ❌ Don't redistribute it: no re-uploading, mirroring, or sharing it outside your team.
- ❌ Don't commit any data files (or decompressed or extracted copies) to your project repo, public or private. Clone this repo alongside your project and add it to `.gitignore`.
- ❌ Don't use it commercially, including betting.
- ❌ Don't use it for any other projects after the hackathon ends.

**Copyright:** the GRID event feeds (`val/` and `lol/` `events_*_grid.jsonl.zst`) are © GRID eSports GmbH and Riot Games, Inc. Everything else (ability supplements, indexes, `meta/` and documentation) is © 2026 Cloud9 Esports, Inc. VALORANT and League of Legends are trademarks of Riot Games, Inc. Match data is provided by GRID. This project is not endorsed by Riot Games.

---

## Coverage by event

<!-- EVENTS:START -->
**VALORANT**: 857 series, 2078 games

| Event | Series | Games | Dates | With ability supplement |
|---|---:|---:|---|---:|
| VCT Americas - Kickoff 2025 | 22 | 56 | 2025-01-16 – 2025-02-08 | 22 |
| Game Changers BR - Kickoff 2025 | 14 | 34 | 2025-02-09 – 2025-02-17 | 14 |
| Game Changers LATAM - Split 1 2025 | 63 | 139 | 2025-03-03 – 2025-05-01 | 63 |
| Game Changers SEA - Split 1 2025 | 10 | 25 | 2025-03-12 – 2025-03-16 | 10 |
| Game Changers BR - Split 1 2025 | 22 | 49 | 2025-03-18 – 2025-05-16 | 22 |
| VCT Americas - Stage 1 2025 | 42 | 104 | 2025-03-21 – 2025-05-04 | 42 |
| Game Changers NA - Split 1 2025 | 48 | 121 | 2025-04-14 – 2025-06-01 | 48 |
| Game Changers JPN - Split 1 2025 | 3 | 9 | 2025-04-28 – 2025-04-30 | 3 |
| Game Changers KR - Split 1 2025 | 1 | 4 | 2025-05-15 – 2025-05-15 | 1 |
| VALORANT Masters - Masters Toronto | 24 | 59 | 2025-06-07 – 2025-06-22 | 24 |
| Game Changers EMEA - Split 2 2025 | 8 | 25 | 2025-06-15 – 2025-06-19 | 8 |
| Game Changers LATAM - Split 2 2025 | 63 | 137 | 2025-06-18 – 2025-07-29 | 63 |
| Game Changers BR - Split 2 2025 | 22 | 52 | 2025-06-24 – 2025-07-24 | 22 |
| Game Changers JPN - Split 2 2025 | 13 | 33 | 2025-06-25 – 2025-06-29 | 13 |
| Game Changers Oceania - 2025 | 6 | 16 | 2025-06-27 – 2025-06-29 | 6 |
| Game Changers SEA - Split 2 2025 | 14 | 34 | 2025-06-30 – 2025-07-06 | 14 |
| Game Changers KR - Split 2 2025 | 1 | 3 | 2025-07-04 – 2025-07-04 | 1 |
| VCT Americas - Stage 2 2025 | 42 | 105 | 2025-07-18 – 2025-08-31 | 42 |
| Game Changers NA - Split 2 2025 | 59 | 147 | 2025-07-28 – 2025-10-12 | 59 |
| Game Changers LATAM - Main Event 2025 | 17 | 48 | 2025-08-18 – 2025-10-12 | 17 |
| Game Changers BR - Season Finals 2025 | 14 | 33 | 2025-09-08 – 2025-10-11 | 14 |
| Game Changers Pacific - 2025 | 14 | 32 | 2025-10-06 – 2025-10-12 | 13 |
| Game Changers EMEA - Split 3 2025 | 8 | 24 | 2025-10-16 – 2025-11-02 | 8 |
| Game Changers Championship - 2025 | 18 | 47 | 2025-11-20 – 2025-11-30 | 18 |
| Game Changers BR - Kickoff 2026 | 14 | 35 | 2026-02-03 – 2026-02-17 | 14 |
| Game Changers LATAM - Kickoff 2026 | 2 | 4 | 2026-02-08 – 2026-02-09 | 2 |
| Game Changers LATAM - Split 1 2026 | 42 | 94 | 2026-02-11 – 2026-04-07 | 42 |
| Game Changers EMEA - Stage 1 2026 | 8 | 21 | 2026-02-18 – 2026-02-22 | 8 |
| Game Changers NA - Kickoff 2026 | 14 | 39 | 2026-02-20 – 2026-03-01 | 14 |
| Game Changers BR - Split 1 2026 | 22 | 53 | 2026-03-21 – 2026-04-30 | 22 |
| Game Changers NA - Split 1 2026 | 59 | 136 | 2026-04-15 – 2026-06-05 | 59 |
| Game Changers KR - Split 1 2026 | 6 | 14 | 2026-04-27 – 2026-05-06 | 6 |
| Game Changers EMEA - Split 2 2026 | 51 | 132 | 2026-04-27 – 2026-05-24 | 51 |
| Game Changers JPN - Split 1 2026 | 6 | 13 | 2026-05-04 – 2026-05-06 | 6 |
| Game Changers LATAM - Split 2 2026 | 42 | 98 | 2026-05-04 – 2026-07-14 | 42 |
| Game Changers BR - Split 2 2026 | 22 | 54 | 2026-05-25 – 2026-07-08 | 22 |
| Game Changers JPN - Split 2 2026 | 6 | 14 | 2026-07-02 – 2026-07-04 | 6 |
| Game Changers KR - Split 2 2026 | 6 | 14 | 2026-07-06 – 2026-07-20 | 6 |
| Game Changers NA - Split 2 2026 | 9 | 21 | 2026-07-13 – 2026-09-02 | 9 |

**League of Legends**: 745 series, 1739 games

| Event | Series | Games | Dates |
|---|---:|---:|---|
| LCK - LCK Cup 2025 | 38 | 109 | 2025-01-15 – 2025-02-23 |
| LEC - Winter 2025 | 59 | 84 | 2025-01-18 – 2025-03-02 |
| LTA South - Split 1 2025 | 12 | 30 | 2025-01-25 – 2025-02-09 |
| LTA North - Split 1 2025 | 12 | 26 | 2025-01-25 – 2025-02-10 |
| LTA Cross-Conference - Split 1 2025 | 7 | 18 | 2025-02-15 – 2025-02-23 |
| LEC - Spring 2025 | 54 | 139 | 2025-03-29 – 2025-06-08 |
| LCK - Split 2 2025 | 97 | 240 | 2025-04-02 – 2025-06-15 |
| LTA South - Split 2 2025 | 52 | 97 | 2025-04-05 – 2025-06-15 |
| LTA North - Split 2 2025 | 52 | 95 | 2025-04-05 – 2025-06-15 |
| LCK - Split 3 2025 | 75 | 206 | 2025-07-23 – 2025-09-28 |
| LTA South - Split 3 2025 | 22 | 67 | 2025-07-26 – 2025-09-07 |
| LTA North - Split 3 2025 | 22 | 68 | 2025-07-26 – 2025-09-07 |
| LEC - Summer 2025 | 30 | 85 | 2025-08-02 – 2025-09-28 |
| LTA Cross-Conference - Regional Championship 2025 | 6 | 22 | 2025-09-13 – 2025-09-28 |
| LEC - LEC Versus 2026 | 79 | 101 | 2026-01-17 – 2026-03-01 |
| LCS - Split 1 2026 | 21 | 58 | 2026-01-24 – 2026-03-01 |
| LEC - Spring 2026 | 53 | 141 | 2026-03-28 – 2026-06-07 |
| LCS - Split 2 2026 | 36 | 98 | 2026-04-04 – 2026-06-14 |
| LCS - Split 3 2026 | 11 | 32 | 2026-07-26 – 2026-10-03 |
| LCS - Promotion 2026 | 7 | 23 | 2026-10-05 – 2026-10-12 |
<!-- EVENTS:END -->
