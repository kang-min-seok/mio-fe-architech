# 00. 공통 및 공통 에러코드

> 버전: v1.1.1 | 모든 도메인 명세서를 읽기 전에 이 문서를 먼저 확인하세요.
> 

## Base URL

| 환경 | Base URL |
| --- | --- |
| 개발 (dev) | `https://api-dev.mio.app` |
| 스테이징 | `https://api-staging.mio.app` |
| 프로덕션 | `https://api.mio.app` |

모든 엔드포인트는 `/v1/` prefix를 가진다.

## 프로토콜

- **REST**: 일반 요청/응답 (JSON)
- **SSE**: 대화 메시지 스트리밍 전용 (`POST /v1/sessions/{id}/messages`)

## 인증 토큰 관리 흐름

```
로그인 성공
  → access_token (JWT, 15분) + refresh_token (Opaque, 30일) 수신
  → access_token : 메모리(앱 상태)에 보관
  → refresh_token : Secure Storage (iOS Keychain / Android EncryptedSharedPreferences)에 보관

매 API 요청
  → Authorization: Bearer <access_token>

401 AUTH_TOKEN_EXPIRED 수신 시
  → POST /v1/auth/refresh 자동 호출
  → 새 access_token + refresh_token 수신
  → 원래 요청 재시도 1회
  → refresh도 401 반환 시 → 로컬 토큰 모두 삭제 → 로그인 화면 이동
```

> ⚠️ Refresh Token은 사용 즉시 무효화되고 새 토큰으로 교체된다.
> 

## 필수 요청 헤더

| 헤더 | 필수 | 설명 |
| --- | --- | --- |
| `Authorization` | ✅ | `Bearer eyJhbG...` |
| `X-Device-Id` | ✅ | UUID v4, 앱 설치 시 1회 생성 |
| `X-App-Version` | ✅ | `1.0.0` |
| `X-Platform` | ✅ | `ios` / `android` |
| `Idempotency-Key` | 조건부 | 체크인·세션 종료 등 |

인증 불필요: `POST /v1/auth/login`, `GET /v1/crisis/resources`

## 성공 응답 포맷

```json
{
  "data": { ... },
  "meta": { "trace_id": "01HVZXXX" }
}
```

## Rate Limit

| 대상 | 한도 |
| --- | --- |
| `POST /v1/auth/*` | 30 req/분/IP |
| `POST /v1/sessions/{id}/messages` | 60 req/분/유저 |
| `POST /v1/checkins` | 4 req/시간/유저 |
| `GET /v1/crisis/resources` | **무제한** |
| 일반 API | 120 req/분/유저 |

## 전체 에러코드 목록

| HTTP | code | 의미 |
| --- | --- | --- |
| 400 | `VALIDATION_ERROR` | 요청 스키마 위반 |
| 401 | `AUTH_TOKEN_EXPIRED` | Access Token 만료 → refresh 후 재시도 |
| 401 | `AUTH_TOKEN_INVALID` | 토큰 검증 실패 → 로그인 이동 |
| 401 | `OAUTH_VERIFICATION_FAILED` | Apple/Kakao 토큰 검증 실패 |
| 401 | `REFRESH_TOKEN_INVALID` | Refresh Token 무효 → 로그인 이동 |
| 403 | `FORBIDDEN` | 권한 없음 |
| 403 | `ACCOUNT_SUSPENDED` | 계정 정지 |
| 403 | `ONBOARDING_REQUIRED` | 온보딩 미완료 |
| 404 | `NOT_FOUND` | 자원 없음 |
| 409 | `CONFLICT` | 중복 |
| 410 | `GONE` | 탈퇴/만료된 자원 |
| 422 | `BUSINESS_RULE_VIOLATION` | 비즈니스 규칙 위반 |
| 422 | `AGE_RESTRICTION` | 만 14세 미만 |
| 422 | `PROVIDER_MISMATCH` | OAuth provider 불일치 |
| 429 | `RATE_LIMITED` | Rate Limit 초과 |
| 500 | `INTERNAL_ERROR` | 서버 내부 오류 |
| 503 | `UPSTREAM_UNAVAILABLE` | 외부 API 장애 |
| 504 | `UPSTREAM_TIMEOUT` | 외부 API 타임아웃 |

## 공통 데이터 타입

| character_id | 이름 | 동물 |
| --- | --- | --- |
| `mio` | 미오 | 펭귄 |
| `bau` | 바우 | 강아지 |
| `rumi` | 루미 | 부엉이 |
| `momo` | 모모 | 곰 |
| `chichi` | 치치 | 고양이 |

### 인지 왜곡 유형 코드

| 코드 | 한국어 명칭 |
| --- | --- |
| `overgeneralization` | 과일반화 |
| `catastrophizing` | 파국화 |
| `mind_reading` | 독심술 |
| `all_or_nothing` | 이분법적 사고 |
| `personalization` | 개인화 |
| `emotional_reasoning` | 감정적 추론 |