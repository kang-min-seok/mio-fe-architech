# 12. Admin (운영자)

> 버전: v1.1.1 | 정책: MIO-Ops-001~006, MIO-Character-006
> 

## 도메인 설명

운영자 전용 관리 기능. 별도 이메일+비밀번호 인증, 일반 사용자 JWT로 접근 불가.

## Base Path

```
POST /v1/admin/auth/login
POST /v1/admin/auth/logout
GET    /v1/admin/users
GET    /v1/admin/users/{user_id}
POST   /v1/admin/users/{user_id}/suspend
DELETE /v1/admin/users/{user_id}/suspend
GET  /v1/admin/crisis-events
GET  /v1/admin/crisis-events/{event_id}
POST /v1/admin/crisis-events/{event_id}/review
```

## 인증

Admin 전용 `admin_access_token` (JWT, 8시간). 5회 연속 실패 시 30분 잠금.

## 관리자 역할

| 역할 | 권한 |
| --- | --- |
| `super_admin` | 전체 접근 |
| `ops_admin` | 위기 이벤트 검토, 회원 제재 |
| `content_admin` | 콘텐츠 관리 |

## 위기 이벤트 검토 큐

| Severity | 검토 기한 | 운영자 조치 |
| --- | --- | --- |
| 3 (명시적) | **즉시** (실시간 알림) | 직접 확인 |
| 2 (간접) | **24시간 이내** | 검토 큐에서 순차 처리 |
| 1 (부정 강도) | **72시간 이내** | 로그 집계 검토 |

위기 이벤트 사용자 메시지 원본은 개인정보 보호 정책에 따라 마스킹된 요약만 제공 (MIO-Ops-004)

## POST /v1/admin/users/{user_id}/suspend

회원 제재

Request Body:

```json
{
  "type": "temporary",
  "reason": "반복적인 스팸 발화",
  "expires_at": "2026-05-18T00:00:00Z"
}
```

`type`: `temporary` / `permanent`

Errors: `404 NOT_FOUND`, `409 CONFLICT` (이미 제재 중)

## POST /v1/admin/crisis-events/{event_id}/review

위기 이벤트 검토 완료

Request Body:

```json
{
  "action": "no_action_needed",
  "note": "자동 감지 오분류로 판단."
}
```

`action`: `no_action_needed` / `user_contacted` / `escalated`

## 에러코드

| 코드 | HTTP | 설명 |
| --- | --- | --- |
| `UNAUTHORIZED` | 401 | 관리자 자격증명 불일치 |
| `FORBIDDEN` | 403 | 일반 사용자 토큰 접근 |
| `NOT_FOUND` | 404 | user_id 또는 event_id 없음 |
| `CONFLICT` | 409 | 이미 제재 중인 회원, 이미 검토된 이벤트 |
| `RATE_LIMIT_EXCEEDED` | 429 | 로그인 5회 연속 실패 |