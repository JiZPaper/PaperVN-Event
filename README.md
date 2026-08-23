# PaperVN Event

## English

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

### 資料結構

`manifest.json` 描述資料快照、記錄數量、檔案匹配模式與目錄清單。實體記錄以 JSON 陣列存放於 `data/`，搜尋與關係索引以 JSON 陣列存放於 `indexes/`。缺少的值使用 `null` 表示。實體記錄保留資料庫欄位，也可能包含未加工的原始資料物件 `raw`。

- `events`：每筆記錄對應一場活動，包含數字 ID、`name`、`event_date`（`YYYY-MM-DD`）、`weekday`、開放與開始/結束時間、`place_id`、各種 URL、標籤、留言與參與人數、`is_past`、`raw` 及抓取時間欄位。
- `actors`：每筆記錄對應一名演職人員或聲優，包含數字 ID、`name`、`kana`、`initial`、性別與統計數值、圖片與活動 URL、`raw` 及抓取時間欄位。
- `places`：每筆記錄對應一個活動場地，包含數字 ID、名稱、都道府縣 ID、地址、郵遞區號、電話、容量、網站、座位圖、座標、地圖 URL、活動數量、`raw` 及抓取時間欄位。
- `event_actors`：活動與演職人員的多對多關係，欄位為 `event_id` 與 `actor_id`。
- `event_links`：活動相關連結，欄位為 `event_id` 與 `url`。
- 搜尋索引：`indexes/actors/`、`indexes/places/`、`indexes/events/` 保存名稱、日期、ID 及指向實體分片的 `shard` 路徑。
- 關係索引：`events-by-actor` 保存 `{actor_id, event_ids}`；`actors-by-event` 保存 `{event_id, actor_ids}`。lookup 目錄把 ID 對應到關係分片。
- 日期索引：`events-by-date` 保存 `{date, shards}`，`shards` 列出該日期的活動資料檔案。
- 目錄清單：`indexes/catalogs/` 保存每個資料或索引分片的 `{path, count, bytes, sha256, first_id, last_id}`，可依 ID 範圍直接選擇檔案。

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

### データ構造

`manifest.json` にはスナップショット、件数、ファイルパターン、カタログの場所が記録されています。実体レコードは `data/`、検索・関連インデックスは `indexes/` に UTF-8 の JSON 配列として保存されています。値がない場合は `null` です。各レコードのデータベースフィールドを保持し、利用できる場合は未加工のデータを `raw` オブジェクトに保存します。

- `events`：イベント ID、`name`、日付（`event_date`）、曜日、開場・開始・終了時刻、`place_id`、各種 URL、ハッシュタグ、メモ数・参加者数、`is_past`、`raw`、取得時刻。
- `actors`：出演者・声優の ID、`name`、`kana`、`initial`、性別と統計値、画像・イベント URL、`raw`、取得時刻。
- `places`：会場 ID、名称、都道府県 ID、住所、郵便番号、電話、収容人数、Web・座席・地図 URL、緯度経度、イベント数、`raw`、取得時刻。
- `event_actors`：`event_id` と `actor_id` による多対多関係。
- `event_links`：`event_id` と `url` による関連 URL。
- 検索インデックス：`indexes/actors/`、`indexes/places/`、`indexes/events/` に検索用の名称・日付・ID と実体分割ファイルへの `shard` パスを収録。
- 関連インデックス：`events-by-actor` は `{actor_id, event_ids}`、`actors-by-event` は `{event_id, actor_ids}` を収録し、lookup ディレクトリで ID から関連分割ファイルを特定できます。
- 日付インデックス：`events-by-date` の各レコードは `{date, shards}` 形式です。
- カタログ：`indexes/catalogs/` に各分割ファイルの `{path, count, bytes, sha256, first_id, last_id}` を収録し、ID の範囲から対象ファイルを選べます。

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

### 데이터 구조

`manifest.json`에는 스냅샷, 레코드 수, 파일 패턴, 카탈로그 위치가 기록됩니다. 실제 레코드는 `data/`, 검색 및 관계 인덱스는 `indexes/`에 UTF-8 JSON 배열로 저장됩니다. 값이 없으면 `null`입니다. 각 레코드는 데이터베이스 필드를 보존하며, 가능한 경우 가공되지 않은 데이터 객체를 `raw`에 저장합니다.

- `events`: 이벤트 ID, `name`, `event_date`（`YYYY-MM-DD`）, 요일, 입장·시작·종료 시간, `place_id`, 각종 URL, 해시태그, 메모·참가자 수, `is_past`, `raw`, 수집 시각.
- `actors`: 출연자·성우 ID, `name`, `kana`, `initial`, 성별과 통계, 이미지·이벤트 URL, `raw`, 수집 시각.
- `places`: 장소 ID, 이름, 도도부현 ID, 주소, 우편번호, 전화번호, 수용 인원, 웹·좌석·지도 URL, 위도·경도, 이벤트 수, `raw`, 수집 시각.
- `event_actors`: `event_id`와 `actor_id`로 구성된 다대다 관계.
- `event_links`: `event_id`와 `url`로 구성된 관련 링크.
- 검색 인덱스: `indexes/actors/`, `indexes/places/`, `indexes/events/`에 이름, 날짜, ID와 실제 데이터 조각을 가리키는 `shard` 경로가 있습니다.
- 관계 인덱스: `events-by-actor`는 `{actor_id, event_ids}`, `actors-by-event`는 `{event_id, actor_ids}`를 저장하며 lookup 디렉터리에서 ID로 관계 조각을 찾습니다.
- 날짜 인덱스: `events-by-date`의 각 레코드는 `{date, shards}` 형식입니다.
- 카탈로그: `indexes/catalogs/`에 각 데이터·인덱스 조각의 `{path, count, bytes, sha256, first_id, last_id}`가 있어 ID 범위로 파일을 선택할 수 있습니다.

PaperVN Event는 [PaperVN App](https://github.com/JiZPaper/PaperVN)에서 사용됩니다. 관련 프로젝트: [VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations), [PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations).

### 사용 방법

정적 데이터 세트이므로 저장소 전체를 clone하거나 `data/`를 한 번에 내려받지 마세요. 먼저 `manifest.json`과 필요한 인덱스를 가져온 뒤, catalog의 `first_id`와 `last_id`로 대상 분할 파일의 `path`를 찾고 해당 JSON 배열만 요청합니다. 성우 일정은 `indexes/actors/` → `indexes/events-by-actor-lookup/` → `indexes/events-by-actor/` → `data/events/` 순서로 조회합니다. 날짜 검색은 `indexes/events-by-date/`, 이벤트에서 출연자를 찾는 역방향 조회는 `indexes/actors-by-event-lookup/` 및 `indexes/actors-by-event/`를 사용합니다. `manifest.json`의 `generated_at`을 확인하고 manifest, 인덱스, 다운로드한 데이터를 앱 캐시에 저장하세요.

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

이 데이터 세트는 자유 콘텐츠이며 [CC0 1.0](LICENSE)을 따릅니다.
