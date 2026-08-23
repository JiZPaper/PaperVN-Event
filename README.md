# PaperVN Event

[English](README.md) | [简体中文](README.zh-Hans.md) | [繁體中文](README.zh-Hant.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

**PaperVN Event** is a free dataset for the PaperVN App. It contains information about nearly 500,000 offline events related to anime, visual novels, and games, including dates, venues, related performers/voice actors, and associated projects.

### Data Structure

`manifest.json` describes the snapshot, record counts, file patterns, and catalog locations. Entity records are stored as JSON arrays under `data/`; search and relationship indexes are stored as JSON arrays under `indexes/`. A missing value is represented by `null`. Entity rows retain the database fields and may include a `raw` object with the original unprocessed payload.

- `events`: one row per event. Fields include the numeric ID, `name`, `event_date` (`YYYY-MM-DD`), `weekday`, `open_time`, `start_time`, `end_time`, `place_id`, `url`, `image_url`, `official_url`, `twitter_hashtag`, `note_count`, `participants_count`, `is_past`, `raw`, `first_seen_at`, and `last_seen_at`.
- `actors`: one row per performer or voice actor. Fields include the numeric ID, `name`, `kana`, `initial`, `sex`, `favorite_count`, `event_count`, `fan_count`, `image_url`, `url`, `all_events_url`, `raw`, `first_seen_at`, and `last_seen_at`.
- `places`: one row per venue. Fields include the numeric ID, `name`, `prefecture_id`, `address`, `postal_code`, `tel`, `capacity`, `web_url`, `seat_url`, `latitude`, `longitude`, `url`, `event_count`, `map_url`, `raw`, `first_seen_at`, and `last_seen_at`.
- `event_actors`: many-to-many links with `event_id` and `actor_id`.
- `event_links`: event links with `event_id` and `url`.
- Search indexes: `indexes/actors/`, `indexes/places/`, and `indexes/events/` contain compact names, dates, IDs, and the data-shard path in `shard`.
- Relationship indexes: `events-by-actor` stores `{actor_id, event_ids}`; `actors-by-event` stores `{event_id, actor_ids}`. Their lookup directories map an ID to the relationship shard containing it.
- Date index: `events-by-date` stores `{date, shards}`, where `shards` lists event data files for that date.
- Catalogs: `indexes/catalogs/` stores `{path, count, bytes, sha256, first_id, last_id}` for each data or index shard. Use the ID range to select a file without scanning the dataset.

PaperVN Event is used by the [PaperVN App](https://apps.apple.com/us/app/papervn/id6793787678). Related projects include [VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations) and [PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations).

### Usage

```text
BASE = https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main/
1. Fetch BASE/manifest.json.
2. Read the relevant index pattern and catalog pattern from the manifest.
3. Fetch the catalog shards (000001 through the catalog's shard_count).
4. Find the catalog row whose first_id <= id <= last_id.
5. Fetch that row's path, parse the JSON array, and select the requested record.
```

For an actor schedule, use this sequence:

1. Find the actor ID in `indexes/actors/` (`{id, name, kana, initial, shard}`).
2. Use `indexes/events-by-actor-lookup/` to map the actor ID to a relationship shard (`{id, shard}`).
3. Fetch the relationship shard from `indexes/events-by-actor/` and read its `event_ids`.
4. Use the `events` catalog to locate the event data shards in `data/events/`.
5. Fetch the event records and, when needed, the place record from `data/places/`.

The reverse lookup (`event_id` to `actor_ids`) is available under `indexes/actors-by-event-lookup/` and `indexes/actors-by-event/`. Date queries use `indexes/events-by-date/`, whose records have the form `{date, shards}`. All paths in indexes are relative to the repository root. Indexes and downloaded data should be cached locally; refresh them when `manifest.json`'s `generated_at` value changes. Entity and index shard files are UTF-8 JSON arrays; `manifest.json` is a JSON object.

For example, a command-line client can start with:

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

This dataset is free content released under [CC0 1.0](LICENSE).
