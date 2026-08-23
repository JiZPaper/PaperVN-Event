# PaperVN Event

## English

**PaperVN Event** is a free dataset for the PaperVN App. It contains information about nearly 500,000 offline events related to anime, visual novels, and games, including dates, venues, related performers/voice actors, and associated projects.

The data is exported from Eventernote into small JSON-array shards. `manifest.json` is the entry point. Entity records are in `data/`; search and relationship indexes are in `indexes/`. Use the catalog files referenced by `manifest.json` to locate a shard, then download only the required JSON file. The target shard size is 32 KiB. A shard may be larger when one individual source record is larger than the target; records are never truncated.

Main entity fields are preserved from the database, including the original `raw` Eventernote JSON when available:

- `events`: event ID, title, date/time, venue ID, URL, project information, and source fields.
- `actors`: performer/voice-actor ID, name, kana, initial, URL, and source fields.
- `places`: venue ID, name, address/location fields, URL, and source fields.
- `event_actors`: event-to-actor relationships.
- `event_links`: event-related URLs.
- `indexes`: compact search, date, and relationship lookups that point to data shards.

PaperVN Event is used by the [PaperVN App](https://github.com/JiZPaper/PaperVN). Related projects include [VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations) and [PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations).

### Usage

The repository is a static dataset. A client should fetch only the files needed for a query instead of cloning the repository or downloading all of `data/`.

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

Swift clients can use `URLSession` with `Decodable`, persist the manifest and index shards in the app's cache directory, and issue requests only for the data shards returned by the indexes.

This dataset is free content released under [CC0 1.0](LICENSE).

## 简体中文

“PaperVN活动”（**PaperVN Event**）是供 PaperVN App 使用的自由数据集，包含动画、视觉小说与游戏等近 50 万场线下活动的时间、地点、相关演职人员与相关企划等信息。

数据由 Eventernote 导出为小型 JSON 数组分片。`manifest.json` 是入口文件；实体数据位于 `data/`，搜索和关系索引位于 `indexes/`。请先读取 `manifest.json` 中引用的目录清单，再按需下载对应分片。目标分片大小为 32 KiB；如果单条原始记录本身大于该大小，对应文件会超过目标，但不会截断记录。

主要数据结构如下：`events` 活动；`actors` 演职人员/声优；`places` 场地；`event_actors` 活动与演职人员关系；`event_links` 活动相关链接；`indexes` 搜索、日期和关系索引。数据库字段均予以保留，并在可用时保留 Eventernote 原始 JSON 字段 `raw`。

本数据集用于 [PaperVN App](https://github.com/JiZPaper/PaperVN)。相关项目：[VNDB简介翻译](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN本地化](https://github.com/JiZPaper/PaperVN-Localizations)。

### 使用方法

这是一个静态数据集。客户端不应克隆整个仓库，也不应一次性下载 `data/`；应先读取索引，只获取当前查询所需的文件。

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

Swift 客户端可以使用 `URLSession` 和 `Decodable`，先缓存 manifest 与索引，再只请求索引返回的活动分片；不要在 App 启动时下载整个仓库。

“PaperVN活动”是自由内容，遵循 [CC0 1.0](LICENSE) 协议。

## 繁體中文

「PaperVN活動」（**PaperVN Event**）是供 PaperVN App 使用的自由資料集，包含動畫、視覺小說與遊戲等近 50 萬場線下活動的時間、地點、相關演職人員與相關企劃等資訊。

資料由 Eventernote 匯出為小型 JSON 陣列分片。`manifest.json` 是入口檔案；實體資料位於 `data/`，搜尋與關係索引位於 `indexes/`。請先讀取 `manifest.json` 所引用的目錄清單，再按需下載對應分片。目標分片大小為 32 KiB；若單筆原始記錄本身大於該大小，對應檔案會超過目標，但不會截斷記錄。

主要資料結構如下：`events` 活動；`actors` 演職人員/聲優；`places` 場地；`event_actors` 活動與演職人員關係；`event_links` 活動相關連結；`indexes` 搜尋、日期與關係索引。資料庫欄位均予以保留，並在可用時保留 Eventernote 原始 JSON 欄位 `raw`。

本資料集用於 [PaperVN App](https://github.com/JiZPaper/PaperVN)。相關專案：[VNDB簡介翻譯](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN本地化](https://github.com/JiZPaper/PaperVN-Localizations)。

### 使用方式

這是靜態資料集。客戶端不應複製整個儲存庫，也不應一次下載 `data/`；請先讀取索引，只取得目前查詢需要的檔案。流程與簡體中文章節相同：讀取 `manifest.json`，下載 catalog 分片，依 `first_id` 與 `last_id` 找到資料路徑，再取得對應 JSON 陣列。

查詢聲優行程時，依序使用 `indexes/actors/`、`indexes/events-by-actor-lookup/`、`indexes/events-by-actor/`，再透過 `events` catalog 取得 `data/events/` 的活動資料。日期索引位於 `indexes/events-by-date/`；反向活動查詢位於 `indexes/actors-by-event-lookup/` 與 `indexes/actors-by-event/`。請在 App 快取 manifest 與索引，並在 `generated_at` 改變時更新。

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

「PaperVN活動」是自由內容，遵循 [CC0 1.0](LICENSE) 協議。

## 日本語

「PaperVNイベント」（英語名 **PaperVN Event**）は PaperVN App 用の自由なデータセットです。アニメ、ビジュアルノベル、ゲームなどに関する約 50 万件のオフラインイベントについて、日時、会場、出演者・声優、関連企画などを収録しています。

Eventernote から小さな JSON 配列の分割ファイルとして書き出しています。`manifest.json` が入口で、実体データは `data/`、検索・関連インデックスは `indexes/` にあります。`manifest.json` のカタログから必要な分割ファイルだけを取得してください。目標サイズは 32 KiB です。元の 1 レコードが目標より大きい場合はファイルが大きくなりますが、レコードは切り詰めません。

主な構造：`events` イベント、`actors` 出演者・声優、`places` 会場、`event_actors` イベントと出演者の関係、`event_links` 関連 URL、`indexes` 検索・日付・関係インデックス。データベースの全フィールドを保持し、利用可能な場合は Eventernote の元 JSON を `raw` に保存しています。

PaperVN Event は [PaperVN App](https://github.com/JiZPaper/PaperVN) で使用します。関連プロジェクト：[VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations)。

### 使い方

静的データセットのため、リポジトリ全体を clone したり `data/` を一括取得したりせず、`manifest.json` と必要なインデックスだけを取得してください。catalog の `first_id` と `last_id` で対象分割ファイルの `path` を特定し、その JSON 配列を取得します。出演者・声優の予定は `indexes/actors/` → `indexes/events-by-actor-lookup/` → `indexes/events-by-actor/` → `data/events/` の順に参照します。日付検索は `indexes/events-by-date/`、イベントから出演者を探す場合は `indexes/actors-by-event-lookup/` と `indexes/actors-by-event/` を使用します。`manifest.json` の `generated_at` を確認し、インデックスと取得済みデータをアプリのキャッシュに保存してください。

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

本データセットは自由なコンテンツであり、[CC0 1.0](LICENSE) に従います。

## 한국어

**PaperVN Event**는 PaperVN App을 위한 자유 데이터 세트입니다. 애니메이션, 비주얼 노벨, 게임 등과 관련된 약 50만 건의 오프라인 이벤트에 대해 날짜와 시간, 장소, 관련 출연자·성우, 관련 기획 정보를 제공합니다.

Eventernote에서 작은 JSON 배열 조각으로 내보냈습니다. `manifest.json`이 진입점이며, 실제 데이터는 `data/`, 검색 및 관계 인덱스는 `indexes/`에 있습니다. `manifest.json`에 지정된 카탈로그에서 필요한 조각만 가져오세요. 목표 조각 크기는 32 KiB입니다. 원본 레코드 하나가 목표보다 크면 파일이 더 커질 수 있지만 레코드는 잘리지 않습니다.

주요 구조: `events` 이벤트, `actors` 출연자·성우, `places` 장소, `event_actors` 이벤트와 출연자 관계, `event_links` 관련 URL, `indexes` 검색·날짜·관계 인덱스. 데이터베이스의 모든 필드를 보존하며, 가능한 경우 Eventernote 원본 JSON을 `raw` 필드에 보존합니다.

PaperVN Event는 [PaperVN App](https://github.com/JiZPaper/PaperVN)에서 사용됩니다. 관련 프로젝트: [VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations), [PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations).

### 사용 방법

정적 데이터 세트이므로 저장소 전체를 clone하거나 `data/`를 한 번에 내려받지 마세요. 먼저 `manifest.json`과 필요한 인덱스를 가져온 뒤, catalog의 `first_id`와 `last_id`로 대상 분할 파일의 `path`를 찾고 해당 JSON 배열만 요청합니다. 성우 일정은 `indexes/actors/` → `indexes/events-by-actor-lookup/` → `indexes/events-by-actor/` → `data/events/` 순서로 조회합니다. 날짜 검색은 `indexes/events-by-date/`, 이벤트에서 출연자를 찾는 역방향 조회는 `indexes/actors-by-event-lookup/` 및 `indexes/actors-by-event/`를 사용합니다. `manifest.json`의 `generated_at`을 확인하고 manifest, 인덱스, 다운로드한 데이터를 앱 캐시에 저장하세요.

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

이 데이터 세트는 자유 콘텐츠이며 [CC0 1.0](LICENSE)을 따릅니다.
