# 백엔드 아키텍처 분석 — 프론트엔드 연동 가이드

> [← 인덱스로 돌아가기](./index.md)
>
> 작성일: 2026-05-10
> 기준 문서: `mio-system-architecture.md` v1.0 · `mio-ai-architecture.md` v1.0

---

## 1. 인증 흐름

### 1.1 소셜 로그인 → JWT 발급

```
앱(클라이언트)
  ↓ Apple / Kakao OAuth 처리
OAuth Provider (Apple / Kakao)
  ↓ authorization code 또는 identity token
Spring Boot /auth 모듈
  ↓ 사용자 조회 or 신규 생성
JWT Access Token + Refresh Token 발급
  ↓
앱이 토큰 저장 (SecureStore)
```

authStore에서 관리하는 `accessToken`은 이 흐름에서 받는 JWT Access Token이다.

### 1.2 토큰 스펙

| 항목 | 값 |
|---|---|
| Access Token TTL | 3600초 (1시간) |
| Refresh Token TTL | 1209600초 (14일) |
| 요청 헤더 형식 | `Authorization: Bearer {accessToken}` |
| 권한 모델 | USER / ADMIN (앱은 USER만 사용) |

### 1.3 토큰 갱신 처리

Access Token 만료 시 Axios 인터셉터에서 Refresh Token으로 재발급 후 `authStore.setAccessToken()`을 호출한다.
Refresh Token도 만료된 경우 `authStore.logout()`을 호출하고 로그인 화면으로 이동한다.

---

## 2. API 엔드포인트 그룹

백엔드 문서가 정의한 URL 그룹은 다음과 같다. 각 그룹의 HTTP 메서드(GET/POST 등) 및 세부 경로는 API 명세 확정 후 반영한다.

```
/auth           OAuth 로그인 처리, JWT 발급/검증, Refresh Token 관리
/users          사용자 프로필, 동의, 탈퇴·비활성화
/conversations  대화 세션 생성, 메시지 저장·조회, SSE 응답
/characters     캐릭터 프로필·상태·기본 설정
/emotions       감정 체크인 저장, 감정 점수 기록, 일/주 단위 조회
/todos          To-do 생성·완료·미완료, 이력 조회
/notifications  Push 토큰 관리, 알림 설정
```

모든 요청은 Nginx를 통해 HTTPS로 진행된다. HTTP 요청은 자동으로 HTTPS로 redirect된다.

---

## 3. 공통 응답 포맷

백엔드는 `global.response` 모듈에서 공통 응답 래퍼를 제공한다.

문서에 명시된 구성:
- 성공 응답 wrapper
- 페이지 응답 wrapper
- (에러는 `global.exception`의 `ErrorCode` · `ErrorResponse` 사용)

구체적인 JSON 필드 구조(키 이름·타입)는 API 명세 확정 후 반영한다. Axios 인터셉터와 응답 파싱 코드는 명세 확정 전에 확정하지 않는다.

---

## 4. 채팅 SSE 스트리밍

### 4.1 스트리밍 방식 확정 — SSE

`08-next-steps.md`에 "SSE vs WebSocket 확정" 이슈가 남아 있었으나 **백엔드는 SSE로 설계가 확정**되어 있다.

근거:
- Nginx 설정에 `proxy_buffering off`, `proxy_cache off` 명시
- Spring Boot 패키지에 SSE 응답 처리가 conversation 도메인 책임으로 정의됨
- Nginx `proxy_read_timeout 300s` (스트리밍 장시간 연결 대응)

WebSocket은 v1 범위에 없다. chatStore의 SSE 기반 `appendStreamChunk()` 설계 방향이 올바르다.

### 4.2 Delivery Mode — 백엔드 AI가 결정하는 응답 방식

백엔드 Policy Engine이 메시지를 분석한 뒤 아래 5가지 전달 방식 중 하나를 선택한다.

| Delivery Mode | 동작 | 프론트엔드 처리 |
|---|---|---|
| `NORMAL_STREAM` | 즉시 SSE 스트리밍 | `appendStreamChunk()` 정상 호출 |
| `SAFE_STREAM` | 안전 지침 주입 후 SSE 스트리밍 | `appendStreamChunk()` 정상 호출 |
| `BUFFER_AND_JUDGE` | 서버에서 전체 응답 생성 후 한 번에 전달 | 스트리밍 없이 단일 응답 도착 — 이 경우도 처리해야 함 |
| `SECURITY_REFUSAL` | 고정 보안 거절 메시지 즉시 전달 | 단일 응답 — 스트리밍 없음 |
| `CRISIS_FLOW` | 위기 대응 고정 메시지 즉시 전달 | 단일 응답 — 스트리밍 없음 |

**핵심:** 메시지 전송 후 응답이 반드시 스트리밍으로 오는 것이 아니다. `BUFFER_AND_JUDGE`, `SECURITY_REFUSAL`, `CRISIS_FLOW`는 스트리밍 없이 한 번에 응답이 온다. SSE 연결 이후 단일 이벤트로 전체 메시지가 오는 경우도 정상으로 처리해야 한다.

### 4.3 First Token Latency 예상치

Risk 수준에 따라 응답 시작 전 mini 모델 판단이 끼어든다.

| 상황 | 추가 레이턴시 |
|---|---|
| 위험 신호 없음 (clear_low) | 0 — mini 생략, 즉시 스트리밍 |
| 위험 신호 있음 (mini 동기 판단) | p50: +500ms~+900ms |
| 위험 신호 있음 (mini 동기 판단) | p95: +1.2s~+2.0s |
| 장애·혼잡·재시도 포함 | +2.5s 이상 가능 |

일반 대화에서 응답이 느리게 시작되는 경우 오류가 아니라 AI 안전 판단이 진행 중인 것이다.
`isTyping` 상태를 타임아웃 없이 유지하고, 타임아웃은 서버 `proxy_read_timeout(300s)` 기준으로 설정한다.

### 4.4 생각 재구성 (CBT) 응답 구조

백엔드 `ai.memory` 모듈이 CBT 패턴을 감지하면 응답에 재구성 제안 관련 필드가 포함된다.
백엔드 CBT 패턴 타입 목록:

```typescript
type CognitiveDistortionType =
  | 'catastrophizing'      // 파국화
  | 'mind_reading'         // 독심술
  | 'all_or_nothing'       // 흑백논리
  | 'overgeneralization'   // 과잉일반화
  | 'self_blame'           // 자기비난
  | 'fortune_telling';     // 미래 예언
```

이 중 무엇이 감지됐는지는 chatStore의 `restructurePrompt`에 담겨 올 수 있다.
API 명세 확정 시 `restructurePrompt` 필드에 `distortionType`도 포함할지 백엔드와 협의한다.

---

## 5. 감정 타입 정합성 ✅ 확정

프론트엔드 `checkinStore`의 `EmotionType`을 **백엔드 타입으로 통일**한다.

| | 타입 목록 |
|---|---|
| **확정 타입** (프론트엔드 + 백엔드 동일) | `anxiety`, `sadness`, `anger`, `shame`, `stress`, `neutral`, `positive` |
| ~~이전 프론트엔드 타입~~ (폐기) | ~~`happy`, `calm`, `neutral`, `anxious`, `tired`~~ |

- `checkinStore.EmotionType` 정의 업데이트 완료 (07-state-management.md)
- UI 이모지·한글 라벨은 `constants/emotions.ts`에서 EmotionType → 이모지·라벨·색상으로 매핑
- 체크인 API 요청 시 이 타입 값을 그대로 전송 (변환 레이어 없음)

---

## 6. 캐릭터 ID

백엔드 character 도메인과 AI 메모리에서 사용하는 캐릭터 ID는 다음과 같다.

```typescript
type CharacterId = 'mio' | 'bau' | 'rumi' | 'momo' | 'chichi';
```

`/characters` 엔드포인트에서 캐릭터 목록을 받아 올 때 이 ID를 기준으로 매핑한다.
`characters` 쿼리의 `staleTime: Infinity` 설정은 올바르다 — 캐릭터 데이터는 정적이다.

---

## 7. To-do 상태 값

백엔드 `behavior_tasks` 테이블의 `status` 컬럼에서 사용하는 값:

```typescript
type TodoStatus = 'suggested' | 'accepted' | 'completed' | 'skipped' | 'failed';
```

`/todos` API 요청 시 이 값을 그대로 사용한다.

---

## 8. 메모리·개인정보 UI 요구사항

백엔드 Privacy/Consent Layer가 다음 기능을 제공해야 한다고 명세하고 있다. 프론트엔드 마이페이지(또는 별도 화면)에서 구현이 필요하다.

| 기능 | 설명 |
|---|---|
| 기억 목록 조회 | Mio가 기억하고 있는 항목 화면 |
| 개별 메모리 삭제 | 사용자가 특정 기억 삭제 |
| 카테고리별 기억 끄기 | 감정 패턴, 행동 이력 등 카테고리 단위 토글 |
| 민감 기억 자동 만료 | 서버에서 처리, UI는 안내 문구만 표시 |
| 데이터 내보내기 | 사용자 데이터 export 요청 |
| 장기 메모리 초기화 | 전체 메모리 리셋 |

현재 `04-pages-mypage.md`에 이 항목들이 설계되어 있는지 확인 후 추가 설계가 필요하면 반영한다.

---

## 9. 푸시 알림 FCM 토큰 관리

백엔드 notification 도메인은 `push_tokens` 테이블에서 FCM 토큰을 관리한다.

토큰 등록 시점: 앱 최초 실행 또는 로그인 후 `expo-notifications`에서 FCM 토큰 획득 → `/notifications` 토큰 등록 API 호출.
토큰 갱신: FCM 토큰은 변경될 수 있으므로 앱 재실행 시마다 최신 토큰을 서버에 업데이트한다.

백엔드가 발송하는 알림 종류:
- 체크인 리마인드
- To-do 리마인드

각 알림의 딥링크 처리는 `09-push-notifications.md`와 연계한다.

---

## 10. 보안·안전 이벤트와 UI

백엔드 AI는 사용자 메시지를 분석해 보안·위기 수준에 따라 고정 응답을 보내는 경우가 있다.
이 응답들은 일반 캐릭터 응답과 다른 형태일 수 있으므로 채팅 UI에서 구분 처리를 고려한다.

| 상황 | 백엔드 동작 | 프론트엔드 처리 방향 |
|---|---|---|
| 보안 공격 감지 | 고정 보안 거절 메시지 반환 | 일반 AI 버블로 표시 (특별 UI 불필요) |
| 위기 신호 감지 | Crisis Flow — 위기 안내 + 리소스 연결 메시지 반환 | 일반 AI 버블로 표시, 링크 포함될 수 있음 |

백엔드는 위험 라벨이나 risk_level 값을 응답에 직접 노출하지 않는다. 프론트엔드는 AI 응답 내용 자체만 표시하면 된다.

---

## 11. AI 응답 관련 UI 제약

백엔드 AI 설계에서 명시된 금지 사항 — 아래 정보는 API 응답에 절대 포함되지 않는다.

```
- risk_level, router 결과, 내부 판단 점수
- Security Harness / Safety Harness 판단 원문
- Policy Engine 내부 점수
- 시스템·개발자 프롬프트 내용
```

프론트엔드에서 이 값들을 파싱하거나 표시하려는 시도는 의미 없다.

---

## 12. 에러 처리 공통 전략

백엔드 `global.exception` 모듈이 `GlobalExceptionHandler`로 모든 예외를 처리하고 공통 `ErrorCode` · `ErrorResponse`를 반환한다.

`global.security` 모듈에는 `AuthenticationEntryPoint`(인증 실패)와 `AccessDeniedHandler`(권한 부족)가 정의되어 있으므로, 각각 HTTP 401 · 403 응답이 발생할 수 있다.

그 외 세부 에러 코드 목록, 재시도 정책, 특정 HTTP 상태별 처리 방식은 API 명세 확정 후 반영한다.

---

## 13. 요약 — 프론트엔드 설계 반영 필요 항목

| # | 항목 | 우선순위 | 상태 | 관련 문서 |
|---|---|---|---|---|
| 1 | **채팅 SSE 확정** — WebSocket 검토 불필요 | 높음 | ✅ 반영 완료 | `04-pages-chat.md`, `08-next-steps.md` |
| 2 | **BUFFER_AND_JUDGE 처리** — 스트리밍 없는 단일 응답 케이스 구현 | 높음 | ✅ 반영 완료 | `04-pages-chat.md` |
| 3 | **감정 타입 정합성** — 백엔드 타입으로 통일 (`checkinStore.EmotionType` 확정) | 높음 | ✅ 반영 완료 | `07-state-management.md`, `04-pages-checkin.md` |
| 4 | **공통 응답 포맷 확정** — API 명세 나오면 Axios 인터셉터 및 파싱 코드 반영 | 높음 | ⏳ API 명세 대기 | — |
| 5 | **First Token Latency** — isTyping 타임아웃 정책 설정 | 중간 | ✅ 반영 완료 | `04-pages-chat.md` |
| 6 | **메모리·개인정보 UI** — 마이페이지에 기억 관리 화면 추가 | 중간 | ✅ 반영 완료 | `04-pages-mypage.md` |
| 7 | **엔드포인트 HTTP 메서드 확정** — API 명세 확정 후 각 도메인 경로·메서드 반영 | 중간 | ⏳ API 명세 대기 | — |
| 8 | **캐릭터 ID 매핑** — 5종 ID 확인 (mio/bau/rumi/momo/chichi) | 낮음 | ✅ 반영 완료 | `07-state-management.md` |
| 9 | **To-do 상태 값 확인** — 5종 status 값 API 명세와 맞추기 | 낮음 | ⏳ API 명세 대기 | — |
