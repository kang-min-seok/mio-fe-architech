# 02. Onboarding (온보딩)

> 버전: v1.1.1 | 정책: MIO-Onboarding-001~007
> 

## 도메인 설명

신규 사용자의 초기 설정 과정. 온보딩 완료 후 대화·체크인 기능 이용 가능.

- **3단계**: 감정 상태 / 주요 고민 유형 / 선호 상담 스타일
- 총 **8~12개 질문**, 완료 예상 시간 **3~5분**

## Base Path

```
POST /v1/onboarding/submit
GET  /v1/onboarding/status
```

## 온보딩 완료 전 기능 제한

미완료 상태에서 아래 접근 시 `403 ONBOARDING_REQUIRED`:

- `POST /v1/sessions`
- `POST /v1/checkins`
- `GET /v1/reports/*`

## 캐릭터 추천 방식 (1차: 룰 기반)

| emotion_state | preferred_style | 추천 Top 3 |
| --- | --- | --- |
| `anxious` / `mixed` | `warm_listener` | mio → momo → rumi |
| `depressed` / `numb` | `warm_listener` | momo → mio → bau |
| `anxious` | `logical` | rumi → mio → chichi |
| `okay` | `playful` | bau → chichi → mio |
| 기타 조합 | — | mio → bau → momo |

2차 개발: GPT-4o 기반 Top 3 추천 + 추천 이유

## POST /v1/onboarding/submit

Request Body:

```json
{
  "step1_emotion_state": "anxious",
  "step2_concern_types": ["relationship", "work", "self_esteem"],
  "step3_preferred_style": "warm_listener",
  "responses": [
    { "question_id": "q1", "answer": "anxious" }
  ]
}
```

| 필드 | 값 범위 |
| --- | --- |
| `step1_emotion_state` | `anxious` / `depressed` / `numb` / `mixed` / `okay` |
| `step2_concern_types` | `relationship` / `work` / `self_esteem` / `family` / `health` / `future` |
| `step3_preferred_style` | `warm_listener` / `logical` / `playful` / `cheerful` |

Response 200:

```json
{
  "data": {
    "onboarding_completed": true,
    "character_recommendations": [
      { "character_id": "mio", "name": "미오", "match_score": 0.92 },
      { "character_id": "momo", "name": "모모", "match_score": 0.84 },
      { "character_id": "rumi", "name": "루미", "match_score": 0.71 }
    ]
  }
}
```

Errors: `400 VALIDATION_ERROR`, `409 CONFLICT`

## GET /v1/onboarding/status

Response 200:

```json
{
  "data": {
    "onboarding_completed": false,
    "last_completed_step": 1,
    "total_steps": 3
  }
}
```

## 에러코드

| 코드 | HTTP | 설명 |
| --- | --- | --- |
| `ONBOARDING_REQUIRED` | 403 | 온보딩 미완료 상태에서 기능 접근 |
| `VALIDATION_ERROR` | 400 | 응답 형식 오류 |
| `CONFLICT` | 409 | 이미 완료된 온보딩 재제출 |