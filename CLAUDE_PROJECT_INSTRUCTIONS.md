# 지각추적기 프로젝트 지침 / Latecomer Project Instructions

---

## 한국어

### 프로젝트 개요
이 프로젝트는 직원 3명의 출근 시간을 추적하는 대시보드입니다.
사용자가 여기서 출근 정보를 말하면, Claude가 Notion DB를 직접 업데이트하고, GitHub Actions가 이를 읽어 `data.json`에 저장 후 대시보드에 반영합니다.

### 전체 흐름
```
사용자 입력 (여기 채팅)
    ↓
Claude가 Notion DB 업데이트 (Notion MCP 사용)
    ↓
GitHub Actions (평일 매시간 자동 실행)
    ↓
data.json 업데이트 → 대시보드 반영
```

### Notion DB 정보
- **DB ID:** `672112f596f74ce8979687ebfd0438fc`
- **Notion MCP tool:** `notion-update-page` 사용

| 직원 | Page ID |
|------|---------|
| 강성준 | `32b4d4adde5e81c69edacacdb266865b` |
| 이경희 | `32b4d4adde5e81d2abbbe9931d896a0d` |
| 임재우 | `32b4d4adde5e815f8930c0f4cd803f93` |

### Notion DB 필드
- `Name` (title): `"이름 YYYY-MM-DD"` 형식 (예: `"강성준 2026-03-23"`)
- `date:날짜:start`: ISO 날짜 (예: `"2026-03-23"`)
- `date:날짜:is_datetime`: `0`
- `출근시간` (text): `"AM HH:MM"` 형식 (예: `"AM 09:02"`)
- `비고` (text): 연차일 경우 `"연차"`, 그 외 `null`

### 입력 규칙

사용자가 아래와 같이 말하면 해당 날짜(오늘 KST 기준)로 Notion을 업데이트합니다.

| 사용자 입력 | 동작 |
|------------|------|
| `이름` | 현재 KST 시각으로 출근 기록 |
| `이름 지금` | 현재 KST 시각으로 출근 기록 |
| `이름 09:02` | 지정 시각으로 출근 기록 |
| `이름 연차` | 연차 처리 |
| 여러 명 동시 입력 | 병렬로 한 번에 처리 |

### 출근 기록 시 Notion 업데이트
```json
{
  "Name": "강성준 2026-03-23",
  "date:날짜:start": "2026-03-23",
  "date:날짜:is_datetime": 0,
  "출근시간": "AM 09:02",
  "비고": null
}
```

### 연차 기록 시 Notion 업데이트
```json
{
  "Name": "임재우 2026-03-23",
  "date:날짜:start": "2026-03-23",
  "date:날짜:is_datetime": 0,
  "출근시간": null,
  "비고": "연차"
}
```

### 주의사항
- Notion의 3행은 고정 틀입니다. 새 행을 만들지 말고 항상 기존 page ID를 업데이트하세요.
- 날짜는 항상 KST 기준 오늘 날짜를 사용합니다.
- 시각은 항상 `AM HH:MM` 형식으로 저장합니다 (예: `AM 09:02`, `PM 01:30`).
- 여러 명을 동시에 처리할 때는 Notion 업데이트를 병렬로 실행하세요.
- data.json은 GitHub Actions가 자동으로 업데이트하므로 직접 수정하지 않습니다.

---

## English

### Project Overview
This is an employee attendance tracking dashboard for 3 employees.
When the user provides attendance information in chat, Claude updates the Notion DB directly via the Notion MCP. GitHub Actions then reads from Notion and writes to `data.json`, which powers the dashboard.

### Workflow
```
User input (this chat)
    ↓
Claude updates Notion DB (via Notion MCP)
    ↓
GitHub Actions (runs automatically every hour on weekdays)
    ↓
Updates data.json → Dashboard reflects changes
```

### Notion DB Info
- **DB ID:** `672112f596f74ce8979687ebfd0438fc`
- **Tool to use:** `notion-update-page`

| Employee | Page ID |
|----------|---------|
| 강성준 (Kang Seongjun) | `32b4d4adde5e81c69edacacdb266865b` |
| 이경희 (Lee Gyeonghui) | `32b4d4adde5e81d2abbbe9931d896a0d` |
| 임재우 (Im Jaewoo) | `32b4d4adde5e815f8930c0f4cd803f93` |

### Notion DB Fields
- `Name` (title): Format `"Name YYYY-MM-DD"` (e.g. `"강성준 2026-03-23"`)
- `date:날짜:start`: ISO date string (e.g. `"2026-03-23"`)
- `date:날짜:is_datetime`: `0`
- `출근시간` (text): Arrival time in `"AM HH:MM"` format (e.g. `"AM 09:02"`)
- `비고` (text): `"연차"` for annual leave, otherwise `null`

### Input Rules

When the user provides attendance info, update Notion for today's date (KST).

| User Input | Action |
|-----------|--------|
| `Name` | Record arrival at current KST time |
| `Name 지금` (now) | Record arrival at current KST time |
| `Name 09:02` | Record arrival at specified time |
| `Name 연차` (annual leave) | Record as annual leave |
| Multiple names at once | Process all in parallel |

### Notion Update — Arrival
```json
{
  "Name": "강성준 2026-03-23",
  "date:날짜:start": "2026-03-23",
  "date:날짜:is_datetime": 0,
  "출근시간": "AM 09:02",
  "비고": null
}
```

### Notion Update — Annual Leave
```json
{
  "Name": "임재우 2026-03-23",
  "date:날짜:start": "2026-03-23",
  "date:날짜:is_datetime": 0,
  "출근시간": null,
  "비고": "연차"
}
```

### Important Notes
- The 3 Notion rows are fixed templates. Always update existing page IDs — never create new rows.
- Always use today's date in KST (Korea Standard Time, UTC+9).
- Always store times in `AM HH:MM` format (e.g. `AM 09:02`, `PM 01:30`).
- When updating multiple employees at once, run Notion updates in parallel.
- Do not edit `data.json` directly — GitHub Actions handles this automatically.
