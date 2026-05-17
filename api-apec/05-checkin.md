# 05. CheckIn (체크인)

> 버전: v1.1.1 | 정책: MIO-Coaching-001~010
> 

## 도메인 설명

하루 최대 3번 (아침/점심/저녁) 현재 감정 상태를 이모지 + 메모로 기록.

## Base Path

```
POST /v1/checkins
PUT  /v1/checkins/{checkin_id}
GET  /v1/checkins
GET  /v1/checkins/today
```

## 슬롯 구조 (1일 3회)

| 슬롯 | 시간 기준 | time_of_day 값 |
| --- | --- | --- |
| 아침 | 00:00 ~ 11:59 | `morning` |
| 점심 | 12:00 ~ 17:59 | `afternoon` |
| 저녁 | 18:00 ~ 23:59 | `evening` |

동일 날짜 동일 슬롯 중복 체크인 시 `409 CONFLICT`

## 체크인 수정 규칙 (MIO-Coaching-009)

- **당일** 기록에 한해 수정 가능
- 익일 이후 수정 시도 시 `422 BUSINESS_RULE_VIOLATION`
- 수정 이력은 별도 저장되지 않으며, **최종 값으로 덮어씁**

## emoji_score 매핑

| emoji_score | 감정 의미 |
| --- | --- |
| 1 | 매우 힘듦 😢 |
| 2 | 힘듦 😔 |
| 3 | 보통 😐 |
| 4 | 좋음 😊 |
| 5 | 매우 좋음 😄 |

## POST /v1/checkins

Request Headers: `Authorization`, `Idempotency-Key`

Request Body:

```json
{
  "time_of_day": "morning",
  "emoji_score": 2,
  "memo": "오늘 발표가 걱정돼서 잠을 못 췄어"
}
```

Response 201:

```json
{
  "data": {
    "checkin_id": "550e8400-...",
    "time_of_day": "morning",
    "emoji_score": 2,
    "ai_response": "오늘 발표 때문에 많이 진장되셈겠어요. 🤍",
    "triggered_proactive_care": false,
    "todo_generation_suggested": true
  }
}
```

| 필드 | 설명 |
| --- | --- |
| `ai_response` | AI 공감 응답. 1차 개발에서는 null |
| `triggered_proactive_care` | 연속 3회 부정 감정 트리거 발동 여부 |
| `todo_generation_suggested` | true이면 To-do 생성 유도 UI 표시 |

## GET /v1/checkins/today

Response 200:

```json
{
  "data": {
    "date": "2026-05-11",
    "checkins": [...],
    "completed_slots": ["morning"],
    "available_slots": ["afternoon", "evening"]
  }
}
```

## 에러코드

| 코드 | HTTP | 설명 |
| --- | --- | --- |
| `CONFLICT` | 409 | 동일 슬롯 당일 중복 체크인 |
| `ONBOARDING_REQUIRED` | 403 | 온보딩 미완료 |
| `NOT_FOUND` | 404 | 존재하지 않는 checkin_id |
| `FORBIDDEN` | 403 | 타인의 체크인 접근 |
| `BUSINESS_RULE_VIOLATION` | 422 | 익일 이후 수정 시도 |
| `VALIDATION_ERROR` | 400 | emoji_score 범위, memo 길이 초과 |