# 06. Daily Test (데일리 테스트)

> 버전: v1.1.1 | 정책: MIO-Coaching-003
> 

## 도메인 설명

시나리오 기반 감정·성향 분석 콘텐츠. 체크인과 독립된 별도 기능, 1일 1회 제한 (MIO-Coaching-003).

## Base Path

```
GET  /v1/daily-test/today
GET  /v1/daily-test/{test_id}
POST /v1/daily-test/{test_id}/answer
GET  /v1/daily-test/history
```

## 체크인과의 차이

| 항목 | 체크인 | Daily Test |
| --- | --- | --- |
| 목적 | 감정 상태 기록 (1일 3회) | 심리/성향 분석 (1일 1회) |
| 형식 | 이모지 + 메모 | 시나리오 기반 선택형 |
| 데이터 활용 | 리포트, To-do, 선제 개입 | 분석 결과, 성향 파악 |

## GET /v1/daily-test/today

오늘의 Daily Test 조회. 하루 1회만 응답 가능.

Response (미완료):

```json
{
  "data": {
    "test_id": "550e8400-...",
    "title": "스트레스를 받을 때 나는?",
    "estimated_minutes": 3,
    "completed_today": false,
    "questions": [
      {
        "question_id": "q1",
        "order": 1,
        "text": "갑자기 중요한 발표가 생겼을 때, 나는?",
        "options": [
          { "option_id": "q1_a", "text": "미리 철저하게 준비한다" },
          { "option_id": "q1_b", "text": "일단 닥치면 어떻게든 된다" },
          { "option_id": "q1_c", "text": "불안해서 집중이 안 된다" }
        ]
      }
    ]
  }
}
```

Response (오늘 이미 완료):

```json
{
  "data": {
    "completed_today": true,
    "result": {
      "summary": "당신은 계획형 대처자예요.",
      "tags": ["계획형", "분석적", "안정 추구"]
    }
  }
}
```

## POST /v1/daily-test/{test_id}/answer

Request Body:

```json
{
  "answers": [
    { "question_id": "q1", "selected_option_id": "q1_a" }
  ]
}
```

Response 201:

```json
{
  "data": {
    "result": {
      "summary": "당신은 계획형 대처자예요.",
      "description": "스트레스 상황에서 사전 준비와 체계적인 접근을 선호해요.",
      "tags": ["계획형", "분석적"],
      "character_comment": "미오가 말해요: 꽔꽔하게 준비하는 네가 멋져 🐧"
    },
    "completed_at": "2026-05-11T12:30:00Z"
  }
}
```

1차 개발: 선택지 조합 기반 룰 엔진 결과 반환 / 2차 개발: GPT-4o 맞춤형 분석

## 에러코드

| 코드 | HTTP | 설명 |
| --- | --- | --- |
| `CONFLICT` | 409 | 오늘 이미 응답 완료 |
| `NOT_FOUND` | 404 | 존재하지 않는 test_id |
| `VALIDATION_ERROR` | 400 | 필수 문항 미응답 |