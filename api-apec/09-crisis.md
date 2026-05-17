# 09. Crisis (위기)

> 버전: v1.1.1 | 정책: MIO-Character-006, MIO-Ops-004, MIO-Proactive-001
> 

## 도메인 설명

자해·극단적 표현 등 위험 발화 감지 시 또는 사용자 요청 시 위기 대응 리소스를 제공하고 위기 이벤트를 기록.

> ⚠️ **안전 최우선 정책**: `GET /v1/crisis/resources`는 **Rate Limit 없음**, **인증 불필요**
> 

## Base Path

```
GET  /v1/crisis/resources
POST /v1/crisis/flag
```

## Severity 레벨

| Severity | 상황 | 처리 |
| --- | --- | --- |
| 1 | 부정 감정 강도 높음 | 위기 응답 + 로그 저장 |
| 2 | 자해/자살 간접 표현 | 운영자 검토 큐 등록 + 전문기관 안내 |
| 3 | 명시적 자해/자살 표현 | **즉시** 운영자 알림 + 핵라인 강제 표시 |

위기 개입은 알림 동의 무관, 방해 금지 시간 **무관**하게 동작 (MIO-Proactive-001)

## 핵라인 (항상 화면 고정 표시 권장)

| 이름 | 번호 | 운영 |
| --- | --- | --- |
| 자살예방상담전화 | **109** | 24/7 무료 |
| 정신건강위기상담전화 | **1577-0199** | 24/7 무료 |

## GET /v1/crisis/resources

**Rate Limit 없음. 인증 불필요.**

Query Params: `lat`, `lng` (선택, 주변 기관 조회용, 서버에 저장되지 않음)

Response 200:

```json
{
  "data": {
    "hotlines": [
      { "name": "자살예방상담전화", "number": "109", "hours": "24/7", "is_free": true },
      { "name": "정신건강위기상담전화", "number": "1577-0199", "hours": "24/7", "is_free": true }
    ],
    "nearby_facilities": [],
    "cached": true
  }
}
```

`nearby_facilities`: lat/lng 미제공 시 빈 배열

## POST /v1/crisis/flag

위기 이벤트 기록. 시스템 자동 호출 또는 SOS 버튼 터치 시.

Request Body:

```json
{
  "session_id": "550e8400-...",
  "trigger_type": "keyword",
  "severity": 2,
  "user_message_id": "msg_in_abc123"
}
```

`trigger_type`: `keyword` / `moderation` / `pattern` / `user_sos`

Response 201:

```json
{
  "data": {
    "crisis_event_id": "550e8400-...",
    "severity": 2,
    "operator_notified": true
  }
}
```

`operator_notified`: severity >= 2이면 운영자 검토 큐 등록

## 권장 UX

```
위기 대화 후 화면 구성:
┌─────────────────────────────┐
│ 지금 많이 힘드시죠.              │
│ 혼자 이 감정을 버티지 않아도 돼요. │
│                             │
│  📞 109 자살예방상담전화   │  ← tel: 링크
│  📞 1577-0199 정신건강위기 │  ← tel: 링크
│                             │
│  [주변 전문 기관 찾기]         │
└─────────────────────────────┘
```

절대 해서는 안 될 UX: 윗라인 번호를 클릭 가능한 `tel:` 링크 없이 텍스트만 표시