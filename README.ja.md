# PaperVN Event

[English](README.md) | [简体中文](README.zh-Hans.md) | [繁體中文](README.zh-Hant.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

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

PaperVN Event は [PaperVN App](https://apps.apple.com/us/app/papervn/id6793787678) で使用します。関連プロジェクト：[VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations)、[PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations)。

### 使い方

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

本データセットは自由なコンテンツであり、[CC0 1.0](LICENSE) に従います。
