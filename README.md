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

This dataset is free content released under [CC0 1.0](LICENSE).

## 简体中文

“PaperVN活动”（**PaperVN Event**）是供 PaperVN App 使用的自由数据集，包含动画、视觉小说与游戏等近 50 万场线下活动的时间、地点、相关演职人员与相关企划等信息。

数据由 Eventernote 导出为小型 JSON 数组分片。`manifest.json` 是入口文件；实体数据位于 `data/`，搜索和关系索引位于 `indexes/`。请先读取 `manifest.json` 中引用的目录清单，再按需下载对应分片。目标分片大小为 32 KiB；如果单条原始记录本身大于该大小，对应文件会超过目标，但不会截断记录。

主要数据结构如下：`events` 活动；`actors` 演职人员/声优；`places` 场地；`event_actors` 活动与演职人员关系；`event_links` 活动相关链接；`indexes` 搜索、日期和关系索引。数据库字段均予以保留，并在可用时保留 Eventernote 原始 JSON 字段 `raw`。

本数据集用于 [PaperVN App](https://github.com/JiZPaper/PaperVN)。相关项目：[VNDB简介翻译](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN本地化](https://github.com/JiZPaper/PaperVN-Localizations)。

“PaperVN活动”是自由内容，遵循 [CC0 1.0](LICENSE) 协议。

## 繁體中文

「PaperVN活動」（**PaperVN Event**）是供 PaperVN App 使用的自由資料集，包含動畫、視覺小說與遊戲等近 50 萬場線下活動的時間、地點、相關演職人員與相關企劃等資訊。

資料由 Eventernote 匯出為小型 JSON 陣列分片。`manifest.json` 是入口檔案；實體資料位於 `data/`，搜尋與關係索引位於 `indexes/`。請先讀取 `manifest.json` 所引用的目錄清單，再按需下載對應分片。目標分片大小為 32 KiB；若單筆原始記錄本身大於該大小，對應檔案會超過目標，但不會截斷記錄。

主要資料結構如下：`events` 活動；`actors` 演職人員/聲優；`places` 場地；`event_actors` 活動與演職人員關係；`event_links` 活動相關連結；`indexes` 搜尋、日期與關係索引。資料庫欄位均予以保留，並在可用時保留 Eventernote 原始 JSON 欄位 `raw`。

本資料集用於 [PaperVN App](https://github.com/JiZPaper/PaperVN)。相關專案：[VNDB簡介翻譯](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN本地化](https://github.com/JiZPaper/PaperVN-Localizations)。

「PaperVN活動」是自由內容，遵循 [CC0 1.0](LICENSE) 協議。

## 日本語

「PaperVNイベント」（英語名 **PaperVN Event**）は PaperVN App 用の自由なデータセットです。アニメ、ビジュアルノベル、ゲームなどに関する約 50 万件のオフラインイベントについて、日時、会場、出演者・声優、関連企画などを収録しています。

Eventernote から小さな JSON 配列の分割ファイルとして書き出しています。`manifest.json` が入口で、実体データは `data/`、検索・関連インデックスは `indexes/` にあります。`manifest.json` のカタログから必要な分割ファイルだけを取得してください。目標サイズは 32 KiB です。元の 1 レコードが目標より大きい場合はファイルが大きくなりますが、レコードは切り詰めません。

主な構造：`events` イベント、`actors` 出演者・声優、`places` 会場、`event_actors` イベントと出演者の関係、`event_links` 関連 URL、`indexes` 検索・日付・関係インデックス。データベースの全フィールドを保持し、利用可能な場合は Eventernote の元 JSON を `raw` に保存しています。

PaperVN Event は [PaperVN App](https://github.com/JiZPaper/PaperVN) で使用します。関連プロジェクト：[VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations)。

本データセットは自由なコンテンツであり、[CC0 1.0](LICENSE) に従います。

## 한국어

**PaperVN Event**는 PaperVN App을 위한 자유 데이터 세트입니다. 애니메이션, 비주얼 노벨, 게임 등과 관련된 약 50만 건의 오프라인 이벤트에 대해 날짜와 시간, 장소, 관련 출연자·성우, 관련 기획 정보를 제공합니다.

Eventernote에서 작은 JSON 배열 조각으로 내보냈습니다. `manifest.json`이 진입점이며, 실제 데이터는 `data/`, 검색 및 관계 인덱스는 `indexes/`에 있습니다. `manifest.json`에 지정된 카탈로그에서 필요한 조각만 가져오세요. 목표 조각 크기는 32 KiB입니다. 원본 레코드 하나가 목표보다 크면 파일이 더 커질 수 있지만 레코드는 잘리지 않습니다.

주요 구조: `events` 이벤트, `actors` 출연자·성우, `places` 장소, `event_actors` 이벤트와 출연자 관계, `event_links` 관련 URL, `indexes` 검색·날짜·관계 인덱스. 데이터베이스의 모든 필드를 보존하며, 가능한 경우 Eventernote 원본 JSON을 `raw` 필드에 보존합니다.

PaperVN Event는 [PaperVN App](https://github.com/JiZPaper/PaperVN)에서 사용됩니다. 관련 프로젝트: [VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations), [PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations).

이 데이터 세트는 자유 콘텐츠이며 [CC0 1.0](LICENSE)을 따릅니다.
