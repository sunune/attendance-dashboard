# 지각추적기 프로젝트 핸드오프 (2026-03-25)

## 프로젝트 개요

직원 3명의 출근 시간을 추적하는 대시보드.
- **라이브 URL:** https://sunune.github.io/attendance-dashboard/
- **로컬 경로:** `/Users/badamini/Documents/study/Claude code/latecomer`
- **브랜치:** `main`

---

## 전체 데이터 흐름

```
사용자 → Claude 채팅 (여기)
             ↓
     Claude가 Notion DB 업데이트 (Notion MCP)
             ↓
     GitHub Actions (평일 매시간 자동 실행)
             ↓
     data.json 업데이트 → GitHub Pages → 대시보드
```

**⚠ 중요: `data.json`은 GitHub Actions가 자동 관리. 절대 직접 수동 편집 금지.**

---

## 주요 파일

| 파일 | 역할 |
|------|------|
| `index.html` | 대시보드 프론트엔드 (Chart.js 사용) |
| `data.json` | 출근 데이터 (GitHub Actions가 자동 갱신) |
| `attendance.json` | 미사용 파일 (records 빈 배열) |
| `.github/workflows/update-data.yml` | Notion → data.json 자동화 워크플로우 |
| `CLAUDE_PROJECT_INSTRUCTIONS.md` | Claude용 상세 운영 지침 |
| `favicon.svg` / `favicon.png` | 파비콘 |

---

## Notion DB 정보

- **DB ID:** `672112f596f74ce8979687ebfd0438fc`
- **Notion MCP tool:** `notion-update-page`

| 직원 | Page ID |
|------|---------|
| 강성준 | `32b4d4adde5e81c69edacacdb266865b` |
| 이경희 | `32b4d4adde5e81d2abbbe9931d896a0d` |
| 임재우 | `32b4d4adde5e815f8930c0f4cd803f93` |

**Notion 필드 형식:**
- `Name` (title): `"강성준 2026-03-25"`
- `date:날짜:start`: `"2026-03-25"`
- `date:날짜:is_datetime`: `0`
- `출근시간` (text): `"AM 09:02"` 형식
- `비고` (text): 연차면 `"연차"`, 그 외 `null`

---

## 출근 입력 방법

사용자가 아래 형태로 말하면 Claude가 Notion을 업데이트:

| 입력 | 동작 |
|------|------|
| `강성준` | 현재 KST 시각으로 출근 기록 |
| `강성준 09:02` | 지정 시각으로 출근 기록 |
| `강성준 연차` | 연차 처리 |
| `강성준 이경희 임재우` | 동시에 병렬 처리 |

---

## GitHub Actions 워크플로우

- **파일:** `.github/workflows/update-data.yml`
- **실행 주기:** 평일 매시간 (`cron: '0 * * * 1-5'`)
- **동작:** Notion DB 조회 → `data.json` upsert → commit & push
- **상태:** 정상 동작 중 (36회 이상 실행, 모두 성공)

---

## 이번 세션에서 해결한 문제

### 문제: 대시보드에 데이터가 표시되지 않음

**원인:** `data.json`에서 2026-03-24 arrivals 배열 내 `강성준`과 `이경희` 항목 사이에 쉼표 누락 → JSON 파싱 오류

**왜 조용히 실패했나:**
```javascript
// index.html loadData()
catch(e) {}  // 에러를 삼켜버려서 대시보드가 빈 화면으로 표시됨
```

**수정 내용:**
```json
// 수정 전 (broken):
{"name": "강성준", "time": "07:58"}
{"name": "이경희", "time": "09:42"}

// 수정 후 (fixed):
{"name": "강성준", "time": "07:58"},
{"name": "이경희", "time": "09:42"}
```

**커밋:** `e5febd8 fix: add missing comma in data.json (2026-03-24 arrivals)`

**근본 원인:** Python `json.dump`는 항상 유효한 JSON을 생성하므로 GitHub Actions는 문제없었음. 수동으로 data.json을 편집할 때 실수한 것. 앞으로 data.json은 절대 직접 수정 금지.

---

## 로컬 ↔ GitHub 동기화

GitHub Actions는 자동으로 GitHub의 data.json을 업데이트하지만, 로컬은 자동 동기화되지 않음.

**로컬 최신화가 필요할 때:**
```bash
cd "/Users/badamini/Documents/study/Claude code/latecomer"
git pull origin main
```

**자동 동기화 설정 (선택):**
```bash
crontab -e
# 추가:
0 * * * 1-5 cd "/Users/badamini/Documents/study/Claude code/latecomer" && git pull origin main >> /tmp/git-pull.log 2>&1
```

> 대시보드는 GitHub에서 직접 data.json을 읽으므로, 로컬 동기화는 필수가 아님.

---

## 현재 데이터 상태 (2026-03-25 기준)

`data.json`에 기록된 날짜: 2026-03-20, 2026-03-23, 2026-03-24, 2026-03-25
