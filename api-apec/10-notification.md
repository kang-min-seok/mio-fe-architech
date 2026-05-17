# 10. Notification (알림)

> 버전: v1.1.1 | 정책: MIO-Notification-001~007, MIO-Proactive-001~015
> 

## 도메인 설명

Push 알림: iOS APNs HTTP/2 직접 연동 (Firebase 미사용), Android FCM Firebase Admin SDK 사용.

## Base Path

```
GET    /v1/notifications/settings
PUT    /v1/notifications/settings
POST   /v1/notifications/devices
DELETE /v1/notifications/devices/{token}
GET    /v1/notifications
```

## 알림 발송 시간 (MIO-Notification-003)

| 슬롯 | 발송 시간 |
| --- | --- |
| 아침 체크인 리마인더 | 08:00 |
| 점심 체크인 리마인더 | 12:00 |
| 저녁 체크인 리마인더 | 21:00 |

## 방해 금지 시간 (MIO-Notification-004)

`quiet_start` ~ `quiet_end` 구간에는 모든 알림 차단. **단, `event: crisis` 인앱 개입은 무관** (MIO-Proactive-001)

## 선제 개입 트리거

| 트리거 | 조건 | 정책 |
| --- | --- | --- |
| 연속 부정 감정 | 최근 3회 체크인 모두 emoji_score 1~2 | MIO-Proactive-002 |
| 장기 미접속 | 3일 이상 앱 미접속 | MIO-Proactive-003 |
| 고위험 발화 | 세션 중 위기 감지 (severity >= 2) | MIO-Proactive-001 |

## GET /PUT /v1/notifications/settings

Response 200:

```json
{
  "data": {
    "checkin_reminder_enabled": true,
    "todo_reminder_enabled": true,
    "proactive_care_enabled": true,
    "report_enabled": true,
    "character_message_enabled": false,
    "quiet_hours_enabled": true,
    "quiet_start": "23:00",
    "quiet_end": "08:00",
    "timezone": "Asia/Seoul"
  }
}
```

PUT Request Body (부분 업데이트 지원):

```json
{
  "proactive_care_enabled": false,
  "quiet_start": "22:00",
  "quiet_end": "07:30"
}
```

## POST /v1/notifications/devices

디바이스 토큰 등록/갱신. 동일 `device_id`의 기존 토큰은 자동 교체.

Request Body:

```json
{
  "device_id": "iphone-uuid-abc123",
  "push_token": "APNS_TOKEN",
  "platform": "ios",
  "app_version": "1.2.0"
}
```

## DELETE /v1/notifications/devices/{token}

로그아웃 시 호출. 해당 기기로의 push 발송을 중단.

## 에러코드

| 코드 | HTTP | 설명 |
| --- | --- | --- |
| `NOT_FOUND` | 404 | 등록되지 않은 토큰 |
| `VALIDATION_ERROR` | 400 | platform 오류, HH:MM 형식 오류 |