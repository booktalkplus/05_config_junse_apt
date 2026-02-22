# 원격 설정 시스템 (Remote Config)

이 폴더는 확장 프로그램 업데이트·배포 없이 운영 설정을 실시간으로 변경하기 위한 JSON 파일을 관리합니다.
GitHub에 업로드 후 Raw URL을 `config.js`의 `REMOTE_URLS`에 등록하면 앱 실행 시마다 최신 값이 자동 반영됩니다.

---

## 파일 구성 및 역할

| 파일 | 역할 | 변경 빈도 |
|---|---|---|
| [config.json](./config.json) | 수집 전략, 딜레이, 링크, 알림 문구, 기본 키워드 | 수시 (봇 감지 대응, 링크 변경 등) |
| [sponsor.json](./sponsor.json) | 후원 배너 레이아웃·문구 (GA 분석 결과 반영) | 수시 (A/B 테스트) |
| [messages.json](./messages.json) | 수집 진행 중 화면의 롤링 응원 메시지 목록 | 가끔 (문구 추가·수정) |
| [notice.json](./notice.json) | 팝업·리포트 상단 긴급 공지사항 | 필요 시 (장애, 업데이트 안내) |

---

## 파일별 주요 항목

### config.json
- `PHASE_SETTINGS.COLLECTION_STRATEGY` — 수집 전략 긴급 전환 (`INTERCEPTION` ↔ `DIRECT`)
- `PHASE_SETTINGS.ENRICHMENT_LIMIT` — 수집 매물 수 제한 (봇 감지 강화 시 낮춤)
- `PHASE_SETTINGS.ENRICHMENT_DELAY_*` — 수집 딜레이 (봇 회피 튜닝)
- `UI_SETTINGS.KAKAO_TALK_LINK/CODE` — 고객 지원 채팅방 URL (단일 관리 지점)
- `UI_SETTINGS.NOTIFICATION_*` — 수집 완료 브라우저 알림 문구
- `KEYWORDS_DEFAULT` — 삭제 불가 기본 키워드 목록

### sponsor.json
- `SPONSOR_LAYOUT` — 배너 방식 선택 (`A`/`B`/`C`/`D1`/`D2`)
- `SPONSOR_D2_COUNTDOWN_SEC` — D2 카운트다운 초수
- `SPONSOR_LEGAL` — 쿠팡 파트너스 법적 고지 문구 (정책 변경 시 즉시 반영)
- `SPONSOR_TERMINAL_LINES` — D1·D2 터미널 타이핑 문구

### messages.json
- `messages[]` — 수집 진행 중 화면 하단에 롤링으로 표시되는 메시지 배열

### notice.json
- `show: true/false` — 공지 표시 여부 (긴급 상황 시 `true`로 변경)
- `type` — 공지 유형 (`info` / `warning` / `error`)
- `content` — 공지 본문

---

## 보안 원칙

- **COUPANG_LINK(쿠팡 파트너스 수익 링크)는 이 폴더 파일에 절대 포함하지 않습니다.**
  외부에서 이 값을 주입하면 수익이 탈취될 수 있습니다.
  반드시 `chrome-extension/config.js`에서만 수정 후 배포해야 합니다.
- `config-manager.js`는 원격 값을 병합할 때 `UTILITY.COUPANG_LINK`를 자동으로 제외합니다.

---

## 반영 방법

1. 이 폴더의 JSON 파일을 수정합니다.
2. GitHub에 푸시합니다.
3. GitHub에서 해당 파일의 **[Raw]** 버튼 클릭 → URL 복사합니다.
4. `chrome-extension/config.js`의 `SYSTEM.REMOTE_URLS` 항목에 URL을 등록합니다.
5. 이후 사용자가 앱을 실행할 때마다 최신 값이 자동으로 반영됩니다.

> `REMOTE_URLS.SPONSOR`는 현재 `""` (비어 있음) → `sponsor.json`을 GitHub에 올린 후 Raw URL 등록 필요

---

## 향후 확정 기능 (미구현)

### 네이버 부동산 API 엔드포인트·스키마 원격 관리

**배경:**
네이버 부동산은 내부 API 엔드포인트 경로, 요청 파라미터, 응답 필드명을 예고 없이 변경하는 경우가 있습니다.
현재는 이런 변경이 발생하면 확장 프로그램 전체를 재배포해야 합니다.

**목표:**
API 관련 정보를 원격 설정 파일로 분리하여, 확장 프로그램 업데이트 없이 즉시 대응 가능하도록 합니다.

**관리 대상 후보:**

| 항목 | 설명 | 예시 |
|---|---|---|
| API 엔드포인트 경로 | 매물 목록·상세 조회 API URL 패턴 | `/api/v1/articles/...` |
| 요청 파라미터 키 | 단지 코드, 거래 유형 등 쿼리 파라미터 명 | `cortarNo`, `tradeType` |
| 응답 필드명 (스키마) | 매물 가격, 층, 면적 등 JSON 키 이름 | `dealPrice` → `salePrice` |
| 전세 보증금 판별 필드 | 기보증금 존재 여부를 나타내는 필드명 | `isLeaseExist`, `leasePrice` |
| 키워드 검색 대상 필드 | 매물 설명 텍스트를 담는 필드명 목록 | `detailDescription`, `articleFeatureDesc` |

**구현 방식 (설계 필요):**
- `config.json` 또는 별도 `api-schema.json` 파일에 위 항목들을 정의
- `service-worker.js`의 수집 로직이 하드코딩 대신 원격 설정값을 참조하도록 리팩토링
- 네이버 API 변경 감지 시 JSON 파일만 수정·푸시하면 모든 사용자에게 즉시 반영

---

**최초 작성**: 2026-02-16
**최종 수정**: 2026-02-22
**관리자**: booktalkplus
