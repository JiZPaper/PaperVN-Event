# PaperVN Event

[English](README.md) | [简体中文](README.zh-Hans.md) | [繁體中文](README.zh-Hant.md) | [日本語](README.ja.md) | [한국어](README.ko.md)

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

PaperVN Event는 [PaperVN App](https://apps.apple.com/us/app/papervn/id6793787678)에서 사용됩니다. 관련 프로젝트: [VNDB Description Translations](https://github.com/JiZPaper/VNDB-Description-Translations), [PaperVN Localizations](https://github.com/JiZPaper/PaperVN-Localizations).

### 사용 방법

```sh
BASE='https://raw.githubusercontent.com/JiZPaper/PaperVN-Event/main'
curl -L "$BASE/manifest.json"
curl -L "$BASE/indexes/actors/actors-index-000001.json"
```

이 데이터 세트는 자유 콘텐츠이며 [CC0 1.0](LICENSE)을 따릅니다.
