# PaperVN活動

[English](README.md) | [简体中文](README.zh-Hans.md) | [繁體中文](README.zh-Hant.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

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

本資料集用於 [PaperVN App](https://apps.apple.com/us/app/papervn/id6793787678)。相關專案：[VNDB簡介翻譯](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN本地化](https://github.com/JiZPaper/PaperVN-Localizations)。

### 使用方式

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

「PaperVN活動」是自由內容，遵循 [CC0 1.0](LICENSE) 協議。
