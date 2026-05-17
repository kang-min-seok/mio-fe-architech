# 04. Session / Message (세션·메시지)

> 버전: v1.1.1 | 정책: MIO-Session-001~006, MIO-Character-006
> 

## Base Path

```
POST /v1/sessions
GET  /v1/sessions
GET  /v1/sessions/{session_id}
POST /v1/sessions/{session_id}/messages   ← SSE
GET  /v1/sessions/{session_id}/messages
POST /v1/sessions/{session_id}/end
```

## 세션 생명주기

```
[세션 시작 조건]
  - 사용자가 캐릭터 대화 화면 진입
  - 선제 개입 알림 탭
  → POST /v1/sessions

[세션 종료 조건]
  - 사용자가 대화 화면 이탈 → POST /v1/sessions/{id}/end
  - 30분 무응답 → 서버 자동 종료
```

## SSE 이벤트 종류

| 이벤트 타입 | 발생 조건 | 프론트엔드 처리 |
| --- | --- | --- |
| `session_meta` | 메시지 수신 직후 (첫 이벤트) | 메시지 ID 저장 |
| `delta` | LLM 토큰 생성 중 | 말풍선에 실시간 텍스트 추가 |
| `crisis` | 위기 감지 시 | 고정 위기 응답 표시 + **핫라인 버튼 필수** |
| `security_refusal` | 보안 공격 감지 | 거절 메시지 표시 |
| `done` | 응답 완료 | 말풍선 완료, 감정 점수 저장 |
| `error` | 서버 오류 | 오류 안내, 재시도 버튼 |

> ⚠️ **`policy_decision` 이벤트 없음**: PolicyEngine 결정 정보(generation_mode 등)는 서버 내부 로깅 전용(`AiDecisionLogger`). 클라이언트 SSE 스트림에 포함되지 않는다.
> 

> ⚠️ **`bias_detected` 이벤트 없음**: CBT 편향 감지 결과는 응답 전송 완료 후 비동기 Outbox(`BiasDetectedJob`)로 처리된다. 리포트 및 다음 턴 PolicyEngine 컨텍스트에 반영되지만 SSE 스트림에는 전송하지 않는다.
> 

> 1차 개발: `session_meta` + `delta` + `done` 만 존재 (stub)
> 

## SSE 응답 예시 — 정상 흐름 (2차)

```jsx
event: session_meta
data: {"message_id":"msg_in_abc123","received_at":"2026-05-11T09:00:00Z"}

event: delta
data: {"chunk":"오늘 발표가","msg_id":"msg_out_xyz789"}

event: delta
data: {"chunk":" 많이 힘드셨겠어요.","msg_id":"msg_out_xyz789"}

event: done
data: {"msg_id":"msg_out_xyz789","emotion_score":28,"is_crisis_flagged":false,"cbt_intervention":false,"finished_reason":"stop"}

```

> `cbt_intervention: true` → 해당 응답은 GPT-4o로 생성된 소크라테스 질문 턴. 프론트엔드는 일반 말풍선으로 표시하면 된다.
> 

## SSE 응답 예시 — 위기 감지

```
event: crisis
data: {
  "severity": 2,
  "trigger_type": "keyword",
  "fixed_response": "지금 많이 힘드시죠. 혼자 이 감정을 버티지 않아도 돼요.",
  "resources": {
    "hotlines": [
      {"name":"자살예방상담전화","number":"109","hours":"24/7"},
      {"name":"정신건강위기상담전화","number":"1577-0199","hours":"24/7"}
    ]
  }
}

event: done
data: {"is_crisis_flagged":true,"finished_reason":"crisis_flow"}

```

## done 이벤트 필드

| 필드 | 설명 |
| --- | --- |
| `emotion_score` | 0~100 (낙을수록 부정) |
| `is_crisis_flagged` | 위기 감지 여부 |
| `cbt_intervention` | 소크라테스 질문 포함 여부 |
| `finished_reason` | `stop` / `crisis_flow` / `security_refusal` / `error` |

## POST /v1/sessions/{id}/end

세션 종료. 대화 화면 이탈 시 호출.

Response 200:

```json
{
  "data": {
    "session_id": "550e8400-...",
    "ended_at": "2026-05-11T09:30:00Z",
    "message_count": 12,
    "duration_seconds": 1800
  }
}
```

## iOS/Android SSE 구현 주의

- iOS: `URLSession.dataTask` 사용 (`EventSource` 라이브러리 사용 가능)
- Android: OkHttp3 `EventSource` 또는 Retrofit + SSE adapter
- 연결 끊트림 시: `GET /v1/sessions/{id}/messages`로 마지막 메시지 확인 후 재연결