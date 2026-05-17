# 01. Auth (인증)

> 버전: v1.1.1 | 정책: MIO-User-001~017
> 

## 도메인 설명

Firebase 없이 **Kakao OAuth REST API**와 **Apple Sign In REST API**를 백엔드에서 직접 검증.

### 지원 OAuth Provider

| Provider | 플랫폼 | 방식 |
| --- | --- | --- |
| `apple` | iOS **필수** | id_token (JWT) → Apple JWKS 검증 |
| `kakao` | Android 전용, iOS 선택 | access_token → Kakao User Info API |

## Base Path

```
POST /v1/auth/login
POST /v1/auth/signup/complete
POST /v1/auth/refresh
POST /v1/auth/logout
DELETE /v1/auth/withdraw
```

## 가입 및 로그인 통합 플로우

```
신규 유저
  POST /v1/auth/login
    → { is_new_user: true }
    → 닉네임·약관 동의 화면
    → POST /v1/auth/signup/complete
    → 온보딩 화면

기존 유저
  POST /v1/auth/login
    → { is_new_user: false }
    → 홈 또는 온보딩 화면 (onboarding_completed 기준)
```

## 토큰 유효기간

| 토큰 | 유효기간 | 보관 위치 |
| --- | --- | --- |
| access_token (JWT) | **15분** | 메모리 |
| refresh_token (Opaque) | **30일** | Secure Storage |

## POST /v1/auth/login

**인증 불필요**

Request Body:

```json
{
  "provider": "apple",
  "id_token": "eyJhbGci...",
  "access_token": null,
  "device_id": "550e8400-..."
}
```

Response 200:

```json
{
  "data": {
    "access_token": "eyJhbGci...",
    "refresh_token": "mio_refresh_...",
    "expires_in": 900,
    "is_new_user": false,
    "user": {
      "id": "550e8400-...",
      "nickname": "효찬",
      "onboarding_completed": true,
      "preferred_character_id": "mio"
    }
  }
}
```

Errors: `400 INVALID_PROVIDER`, `401 OAUTH_VERIFICATION_FAILED`, `403 ACCOUNT_SUSPENDED`, `410 GONE`, `422 PROVIDER_MISMATCH`

## POST /v1/auth/signup/complete

`is_new_user: true`인 경우에만 호출.

Request Body:

```json
{
  "nickname": "효찬",
  "birth_year": 2000,
  "gender": "male",
  "consents": [
    { "type": "terms", "agreed": true, "version": "v1.0" },
    { "type": "privacy", "agreed": true, "version": "v1.0" }
  ]
}
```

Errors: `400 CONSENT_REQUIRED`, `409 CONFLICT`, `422 AGE_RESTRICTION`

## POST /v1/auth/refresh

```json
{ "refresh_token": "mio_refresh_..." }
```

Response: 새 access_token + refresh_token 쌍 반환. 기존 토큰 즉시 무효화.

Error: `401 REFRESH_TOKEN_INVALID` → 모든 세션 강제 로그아웃

## DELETE /v1/auth/withdraw

탈퇴 처리:

1. 즉시 로그아웃 (모든 Refresh Token 무효화)
2. 식별 정보 비식별화
3. status = withdrawn, deleted_at = now + 30일
4. 30일 후 하드 삭제 Job 등록

## JWT Payload 구조

```json
{
  "sub": "550e8400-...",
  "iat": 1746489600,
  "exp": 1746490500,
  "is_minor": false,
  "device_id": "device-uuid",
  "scope": ["user"]
}
```