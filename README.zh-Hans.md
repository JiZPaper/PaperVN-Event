# PaperVN活动

[English](README.md) | [简体中文](README.zh-Hans.md) | [繁體中文](README.zh-Hant.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

“PaperVN活动”（**PaperVN Event**）是供 PaperVN App 使用的自由数据集，包含动画、视觉小说与游戏等近 50 万场线下活动的时间、地点、相关演职人员与相关企划等信息。

### 数据结构

`manifest.json` 描述数据快照、记录数量、文件匹配模式和目录清单。实体记录以 JSON 数组存放在 `data/`，搜索与关系索引以 JSON 数组存放在 `indexes/`。缺失值使用 `null` 表示。实体记录保留数据库字段，并且可能包含未加工的原始数据对象 `raw`。

- `events`：每条记录对应一场活动。字段包括数字 ID、`name`、`event_date`（`YYYY-MM-DD`）、`weekday`、`open_time`、`start_time`、`end_time`、`place_id`、`url`、`image_url`、`official_url`、`twitter_hashtag`、`note_count`、`participants_count`、`is_past`、`raw`、`first_seen_at` 和 `last_seen_at`。
- `actors`：每条记录对应一名演职人员或声优。字段包括数字 ID、`name`、`kana`、`initial`、`sex`、`favorite_count`、`event_count`、`fan_count`、`image_url`、`url`、`all_events_url`、`raw`、`first_seen_at` 和 `last_seen_at`。
- `places`：每条记录对应一个活动场地。字段包括数字 ID、`name`、`prefecture_id`、`address`、`postal_code`、`tel`、`capacity`、`web_url`、`seat_url`、`latitude`、`longitude`、`url`、`event_count`、`map_url`、`raw`、`first_seen_at` 和 `last_seen_at`。
- `event_actors`：活动与演职人员的多对多关系，字段为 `event_id` 和 `actor_id`。
- `event_links`：活动相关链接，字段为 `event_id` 和 `url`。
- 搜索索引：`indexes/actors/`、`indexes/places/` 和 `indexes/events/` 保存用于搜索的名称、日期、ID，以及指向实体分片的 `shard` 路径。
- 关系索引：`events-by-actor` 保存 `{actor_id, event_ids}`；`actors-by-event` 保存 `{event_id, actor_ids}`。对应的 lookup 目录把 ID 映射到包含关系记录的分片。
- 日期索引：`events-by-date` 保存 `{date, shards}`，其中 `shards` 列出该日期对应的活动数据文件。
- 目录清单：`indexes/catalogs/` 保存每个数据或索引分片的 `{path, count, bytes, sha256, first_id, last_id}`。客户端可通过 ID 范围直接选择文件，而不必扫描整个数据集。

本数据集用于 [PaperVN App](https://apps.apple.com/us/app/papervn/id6793787678)。相关项目：[VNDB简介翻译](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN本地化](https://github.com/JiZPaper/PaperVN-Localizations)。

### 使用方法

```text
BASE = https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main/
1. 下载 BASE/manifest.json。
2. 从 manifest 读取对应索引的 pattern 和 catalog pattern。
3. 按 catalog 的 shard_count 下载 000001 至最后一个 catalog 分片。
4. 找到满足 first_id <= id <= last_id 的 catalog 记录。
5. 下载该记录的 path，解析 JSON 数组并取出目标记录。
```

查询某位声优的活动日程时：先在 `indexes/actors/` 找到声优 ID；再从 `indexes/events-by-actor-lookup/` 找到关系分片；读取 `indexes/events-by-actor/` 中的 `event_ids`；最后通过 `events` catalog 找到 `data/events/` 中的活动记录。需要场地信息时，再按活动的 `place_id` 查询 `data/places/`。

反向查询活动的声优使用 `indexes/actors-by-event-lookup/` 和 `indexes/actors-by-event/`。按日期查询使用 `indexes/events-by-date/`，其中每条记录形如 `{date, shards}`。索引中的路径都相对于仓库根目录。建议将 `manifest.json`、索引和已下载数据保存到 App 缓存；当 `manifest.json` 的 `generated_at` 发生变化时再更新。实体和索引分片均为 UTF-8 编码的 JSON 数组；`manifest.json` 是 JSON 对象。

命令行示例：

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

“PaperVN活动”是自由内容，遵循 [CC0 1.0](LICENSE) 协议。
