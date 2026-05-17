# 07. Todo (투두)

> 버전: v1.1.1 | 정책: MIO-Coaching-004~010, MIO-CBT-015
> 

## 도메인 설명

AI가 사용자의 감정 상태·CBT 결과·행동 이력을 기반로 오늘 실청할 행동 To-do 3건을 자동 생성.

## Base Path

```
POST /v1/todos/generate
GET  /v1/todos
GET  /v1/todos/today
POST /v1/todos/{todo_id}/checkin
DELETE /v1/todos/{todo_id}
```

## 주요 정책

- To-do는 체크인 완료 후 매일 새로 생성
- 당일 미완료 To-do는 **다음날로 이월되지 않음** (MIO-Coaching-010)
- `after_emotion` 값이 2차 개발에서 행동 효과 학습에 활용됨

## POST /v1/todos/generate

Request Body:

```json
{ "source": "checkin", "source_id": "550e8400-..." }
```

Response 201:

```json
{
  "data": {
    "todos": [
      {
        "todo_id": "550e8400-...-001",
        "action_text": "지금 이 순간 천천히 숫을 3번 쉬어보요",
        "category": "심리_안정",
        "difficulty": 1,
        "estimated_minutes": 3,
        "character_comment": "미오가 응원해요! 폴만한 대회!"
      },
      {
        "todo_id": "550e8400-...-002",
        "action_text": "오늘 있었던 일을 최악/현실/최선 세 가지 시각으로 적어보기",
        "category": "인지_재구성",
        "difficulty": 2,
        "estimated_minutes": 10
      },
      {
        "todo_id": "550e8400-...-003",
        "action_text": "밖에 나가서 5분만 산책해보기",
        "category": "행동_활성화",
        "difficulty": 1,
        "estimated_minutes": 5
      }
    ]
  }
}
```

## POST /v1/todos/{todo_id}/checkin

실행 결과 기록

Request Body:

```json
{
  "status": "completed",
  "before_emotion": 2,
  "after_emotion": 3,
  "memo": "호흡하고 나니까 조금 진정이 뤀어"
}
```

| status 값 | 설명 |
| --- | --- |
| `suggested` | 제안됨 |
| `accepted` | 수락 |
| `completed` | 완료 |
| `skipped` | 건너눠 |
| `failed` | 실패 |

Response 200:

```json
{
  "data": {
    "status": "completed",
    "before_emotion": 2,
    "after_emotion": 3,
    "character_reaction": "잘했어! 작은 것부터 하나씩 해나가는 게 진짜 대단한 거야 펉🎉"
  }
}
```

## 에러코드

| 코드 | HTTP | 설명 |
| --- | --- | --- |
| `NOT_FOUND` | 404 | 존재하지 않는 todo_id |
| `FORBIDDEN` | 403 | 타인의 To-do 접근 |
| `CONFLICT` | 409 | 이미 완료 수력된 To-do 재기록 |
| `BUSINESS_RULE_VIOLATION` | 422 | 익일 이후 삭제 시도 |