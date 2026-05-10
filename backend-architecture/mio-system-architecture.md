# mio 시스템 아키텍처

# Mio v1 시스템 아키텍처 문서

**버전:** v1.0

**범위:** AI 내부 레이어 제외, 백엔드 시스템/인프라/배포 구조 중심

**주요 구성:** Nginx, Spring Boot, PostgreSQL RDS, Flyway, GitHub Actions, GHCR, Docker Compose

---

## 1. 문서 목적

이 문서는 Mio v1 서비스의 백엔드 시스템 아키텍처를 정의한다.

본 문서는 AI 대화 생성, Safety, Security, Policy Engine, Memory Context Layer 등 AI 내부 아키텍처는 다루지 않는다. AI 기능은 Spring Boot 애플리케이션 내부의 하나의 도메인 모듈 또는 외부 API 연동 모듈로 취급한다.

본 문서의 범위는 다음과 같다.

- 클라이언트와 서버 간 요청 흐름
- Nginx 기반 HTTPS/reverse proxy 구조
- Spring Boot 애플리케이션 구조
- PostgreSQL RDS 기반 데이터 저장 구조
- Flyway 기반 DB 마이그레이션
- GitHub Actions, GHCR, Docker Compose 기반 CI/CD
- 환경변수 기반 설정 및 시크릿 관리
- 기본 로깅, 헬스체크, 운영 구조

---

## 2. 시스템 개요

Mio v1은 단일 Spring Boot 애플리케이션을 중심으로 구성된다. 외부 클라이언트는 HTTPS를 통해 Nginx에 접근하고, Nginx는 내부 네트워크에서 Spring Boot 애플리케이션으로 요청을 프록시한다.

데이터 저장소는 AWS RDS PostgreSQL을 사용한다. DB 스키마 변경은 Flyway를 통해 관리한다.

배포는 GitHub Actions에서 Docker 이미지를 빌드한 후 GHCR에 push하고, 운영 서버에서 Docker Compose를 통해 최신 이미지를 pull 및 재시작하는 방식으로 수행한다.

```
[Client]
  ├─ Mobile App
  └─ Admin Web

        ↓ HTTPS

[Nginx]
  ├─ TLS termination
  ├─ HTTP → HTTPS redirect
  ├─ Reverse proxy
  └─ Access/Error logging

        ↓ Internal HTTP

[Spring Boot Application]
  ├─ API Layer
  ├─ Auth
  ├─ User
  ├─ Conversation
  ├─ Character
  ├─ Emotion
  ├─ Todo
  ├─ Notification
  ├─ Admin
  ├─ Audit
  ├─ Outbox
  └─ AI Integration Boundary

        ↓ JDBC

[AWS RDS PostgreSQL]
  ├─ Core domain data
  ├─ Conversation data
  ├─ Emotion/Todo data
  ├─ Admin data
  ├─ Audit events
  └─ Outbox events

        ↓

[External Services]
  ├─ OAuth Provider
  ├─ Push Provider
  └─ LLM Provider
```

---

## 3. 아키텍처 원칙

### 3.1 단일 애플리케이션 우선

Mio v1은 마이크로서비스로 분리하지 않고 단일 Spring Boot 애플리케이션으로 구성한다.

```
단일 배포 단위
단일 런타임
단일 백엔드 애플리케이션
도메인별 패키지 분리
```

### 3.2 인프라 최소화

초기 운영 단계에서는 인프라 구성요소를 최소화한다.

v1 초기 구성에서 제외하는 항목은 다음과 같다.

```
ALB
ECS / EKS
Kafka / MSK
OpenSearch
Redshift / EMR
KMS
Secrets Manager
Multi-region
Service Mesh
복잡한 Observability Stack
```

### 3.3 애플리케이션 경계 명확화

인프라는 단순하게 유지하되, 애플리케이션 내부의 도메인 경계는 명확히 분리한다.

```
API 계층은 요청/응답 처리만 담당한다.
도메인 모듈은 각자의 비즈니스 책임을 가진다.
DB 접근은 Repository 계층을 통해 수행한다.
외부 서비스 연동은 Adapter 형태로 분리한다.
Audit과 Outbox는 핵심 도메인 흐름에서 분리된 운영 계층으로 둔다.
```

---

## 4. 런타임 아키텍처

### 4.1 클라이언트 요청 흐름

```
Client
  ↓ HTTPS
Nginx
  ↓ HTTP reverse proxy
Spring Boot Application
  ↓ JDBC
RDS PostgreSQL
```

### 4.2 Nginx 역할

Nginx는 애플리케이션 앞단에서 다음 책임을 가진다.

```
HTTPS TLS termination
HTTP → HTTPS redirect
Spring Boot 애플리케이션으로 reverse proxy
클라이언트 IP 전달
요청 body size 제한
proxy timeout 설정
SSE/streaming 요청에 대한 buffering 제어
access log / error log 기록
```

### 4.3 Spring Boot 역할

Spring Boot 애플리케이션은 모든 백엔드 비즈니스 기능을 담당한다.

```
사용자 인증/인가
사용자 프로필 및 동의 관리
대화 세션 및 메시지 관리
감정 체크인 관리
행동 To-do 관리
알림 설정 및 발송 트리거
관리자 기능
감사 로그 기록
비동기 이벤트 처리
AI 기능 연동 경계 제공
```

### 4.4 PostgreSQL RDS 역할

PostgreSQL RDS는 Mio v1의 주 저장소다.

```
사용자 데이터
인증 연동 데이터
대화 세션 및 메시지 데이터
감정 체크인 데이터
행동 To-do 데이터
알림 데이터
관리자 설정 데이터
정책/프롬프트 버전 메타데이터
감사 이벤트
Outbox 이벤트
```

---

## 5. Spring Boot 애플리케이션 구조

### 5.1 패키지 설계 원칙

Spring Boot 애플리케이션은 도메인 기준으로 패키지를 분리한다. 각 도메인 패키지는 해당 도메인의 API, DTO, Service, Repository, Entity를 내부에 가진다.

```
도메인 기준 패키지 분리
도메인 내부에서 controller / dto / service / repository / entity 구성
공통 관심사는 global 또는 common으로 분리
AI 영역은 별도 하위 패키지로 세분화
```

패키지 분리의 기준은 기술 계층이 아니라 비즈니스 책임이다. 예를 들어 모든 controller를 `controller` 패키지에 모으지 않고, `conversation.controller`, `todo.controller`처럼 도메인 내부에 둔다.

### 5.2 최상위 패키지 구조

```
com.mio
  ├─ auth
  ├─ user
  ├─ conversation
  ├─ character
  ├─ emotion
  ├─ todo
  ├─ notification
  ├─ admin
  ├─ audit
  ├─ outbox
  ├─ ai
  ├─ global
  └─ common
```

### 5.3 도메인 내부 표준 구조

각 도메인 패키지는 기본적으로 다음 구조를 따른다.

```
{domain}
  ├─ controller
  ├─ dto
  │   ├─ request
  │   └─ response
  ├─ service
  ├─ repository
  ├─ entity
  └─ exception optional
```

도메인별로 모든 하위 패키지를 반드시 만들 필요는 없다. 기능이 작거나 단순한 도메인은 필요한 패키지만 둔다. 단, 하나의 패키지가 여러 책임을 동시에 갖지 않도록 단일 책임 원칙을 우선한다.

예시:

```
conversation
  ├─ controller
  │   └─ ConversationController
  ├─ dto
  │   ├─ request
  │   │   ├─ CreateSessionRequest
  │   │   └─ SendMessageRequest
  │   └─ response
  │       ├─ ConversationSessionResponse
  │       └─ MessageResponse
  ├─ service
  │   ├─ ConversationService
  │   └─ ConversationQueryService
  ├─ repository
  │   ├─ ConversationSessionRepository
  │   └─ ConversationMessageRepository
  ├─ entity
  │   ├─ ConversationSession
  │   └─ ConversationMessage
  └─ exception
      └─ ConversationNotFoundException
```

### 5.4 도메인별 책임

### auth

```
OAuth 로그인 처리
JWT 발급 및 검증
Refresh token 관리
Spring Security 인증 필터
인증 실패 처리
```

예상 내부 구조:

```
auth
  ├─ controller
  ├─ dto
  ├─ service
  ├─ repository
  ├─ entity
  ├─ jwt
  └─ oauth
```

### user

```
사용자 프로필 관리
사용자 상태 관리
동의 관리
탈퇴 및 비활성화 처리
```

### conversation

```
대화 세션 생성
메시지 저장
메시지 조회
AI 모듈 호출 경계
SSE 응답 처리
```

Conversation은 대화 흐름의 진입점이다. AI 내부 로직을 직접 구현하지 않고, AI 모듈의 공개 service 또는 facade를 호출한다.

### character

```
캐릭터 프로필 관리
캐릭터 상태 관리
캐릭터별 기본 설정 관리
Prompt template 메타데이터 관리
```

### emotion

```
감정 체크인 저장
감정 점수 기록
일/주 단위 감정 데이터 조회
리포트용 감정 데이터 제공
```

### todo

```
행동 To-do 저장
To-do 완료/미완료 처리
To-do 이력 조회
행동 패턴 데이터 제공
```

### notification

```
Push token 관리
알림 설정 관리
체크인 리마인드
To-do 리마인드
알림 발송 이벤트 처리
```

### admin

```
관리자 인증/권한 확인
프롬프트 버전 관리
정책 버전 메타데이터 관리
운영 데이터 조회
Decision trace 조회
```

### audit

```
중요 이벤트 기록
관리자 변경 이력 기록
정책/프롬프트 변경 이력 기록
보안/안전 관련 이벤트 메타데이터 기록
```

### outbox

```
비동기 이벤트 저장
이벤트 polling
이벤트 retry
dead-letter 상태 관리
알림 발송 트리거
집계 작업 트리거
```

### 5.5 AI 패키지 분리 원칙

AI 영역은 하나의 도메인보다 내부 책임이 크기 때문에 `ai` 패키지 아래에서 세부 책임을 다시 분리한다. 시스템 아키텍처 문서에서는 AI 내부 동작을 상세히 정의하지 않지만, Spring Boot 코드 구조에서는 AI 영역의 책임 분리를 명확히 둔다.

```
ai
  ├─ facade
  ├─ orchestrator
  ├─ dto
  ├─ policy
  ├─ safety
  ├─ security
  ├─ memory
  ├─ prompt
  ├─ llm
  ├─ guard
  ├─ crisis
  └─ log
```

각 하위 패키지의 책임은 다음과 같다.

### ai.facade

```
conversation 도메인에서 호출하는 AI 공개 진입점
AI 내부 세부 흐름 은닉
대화 요청을 AI 처리 결과로 변환
```

### ai.orchestrator

```
AI 처리 흐름 조율
security / safety / memory / policy / prompt / llm / guard 호출 순서 제어
트랜잭션 경계가 아닌 처리 흐름 경계 담당
```

### ai.policy

```
AI 응답 생성 허용 여부 판단
스트리밍 허용 여부 판단
고정 응답 또는 위기 플로우 전환 판단
AI 판단 결과 메타데이터 생성
```

### ai.safety

```
정서 위험 신호 탐지
위기 후보 신호 탐지
Risk Router 연동 경계 제공
```

### ai.security

```
프롬프트 인젝션 및 보안 공격 신호 탐지
보안 거절 판단에 필요한 신호 생성
보안 이벤트 메타데이터 생성
```

### ai.memory

```
대화 맥락 조회
감정/행동 이력 조회
사용자 맥락 조회
PromptBuilder에 전달할 context 구성
```

### ai.prompt

```
캐릭터 프롬프트 조립
런타임 지시문 조립
사용자 맥락 wrapper 적용
prompt version 관리 연계
```

### ai.llm

```
LLM provider 호출
streaming / non-streaming 호출 분리
timeout / retry / 응답 매핑
provider adapter 관리
```

### ai.guard

```
출력 검증
응답 전송 가능 여부 판단 보조
rewrite / replace / fallback action 전달
```

### ai.crisis

```
고정 위기 대응 플로우 처리
위기 리소스 선택
위기 이벤트 기록 연계
```

### ai.log

```
AI decision metadata 기록
LLM call metadata 기록
AI 처리 latency 기록
```

AI 내부의 상세 판단 정책, Safety/Security Layer, Memory Context Layer, LLM Gateway 구조는 별도 AI 아키텍처 문서에서 정의한다.

### 5.6 global 패키지

`global`은 애플리케이션 전역에 적용되는 설정과 횡단 관심사를 담당한다.

```
global
  ├─ config
  ├─ security
  ├─ exception
  ├─ response
  ├─ filter
  └─ logging
```

### global.config

```
Spring configuration
Web configuration
JPA configuration
Async configuration
CORS configuration
```

### global.security

```
Spring Security 설정
SecurityFilterChain
AuthenticationEntryPoint
AccessDeniedHandler
```

### global.exception

```
전역 예외 처리
GlobalExceptionHandler
공통 ErrorCode
공통 ErrorResponse
```

### global.response

```
공통 API 응답 포맷
성공 응답 wrapper
페이지 응답 wrapper
```

### global.filter

```
requestId 생성
logging MDC 설정
요청/응답 공통 필터
```

### global.logging

```
구조화 로그 설정
민감정보 마스킹
요청 tracing context
```

### 5.7 common 패키지

`common`은 특정 도메인에 속하지 않는 재사용 가능한 순수 유틸리티와 공통 타입만 담당한다.

```
common
  ├─ id
  ├─ time
  ├─ util
  └─ type
```

`global`과 `common`의 역할이 겹치지 않도록 다음 기준을 적용한다.

```
global:
  Spring framework, web, security, exception, logging처럼 애플리케이션 전역 동작에 관여하는 코드

common:
  framework 의존성이 낮고 여러 도메인에서 재사용 가능한 값 객체, 유틸리티, 공통 타입
```

역할이 겹치는 경우 `global`과 `common`을 모두 유지하지 않고 하나로 통합한다. 전역 예외 처리, 공통 응답, security 설정은 `global`에 둔다. ID 생성, 시간 유틸리티, 공통 enum, 단순 helper는 `common`에 둔다.

### 5.8 의존 방향

도메인 간 의존은 최소화한다. 다른 도메인의 Entity나 Repository를 직접 참조하지 않고, 필요한 경우 해당 도메인의 Service 또는 QueryService를 통해 접근한다.

```
controller → service → repository → entity
```

허용되는 일반 의존 방향:

```
domain → common
domain → global exception/response type 최소 사용
conversation → ai.facade
notification → outbox
audit → common
```

피해야 하는 의존:

```
controller → repository 직접 호출
다른 도메인의 repository 직접 호출
다른 도메인의 entity를 상태 변경 목적으로 직접 조작
ai 내부 세부 패키지를 conversation에서 직접 호출
global이 특정 domain에 의존
common이 domain 또는 Spring Web에 의존
```

### 5.9 단일 책임 원칙 적용 기준

패키지와 클래스 분리는 다음 기준으로 수행한다.

```
Controller:
  HTTP 요청/응답 처리만 담당

Request/Response DTO:
  외부 API 계약 담당

Service:
  유스케이스 및 트랜잭션 경계 담당

Repository:
  DB 접근 담당

Entity:
  영속 상태와 도메인 기본 규칙 담당

Facade:
  복잡한 내부 모듈을 외부 도메인에 단순한 API로 제공

Adapter:
  외부 서비스 연동 담당
```

Service가 비대해질 경우 다음 기준으로 분리한다.

```
CommandService:
  상태 변경 유스케이스

QueryService:
  조회 유스케이스

Reader:
  내부 조회 보조

Writer:
  내부 저장 보조

Validator:
  도메인 검증

Mapper:
  DTO 변환
```

초기에는 과도하게 분리하지 않는다. 하나의 클래스가 명확한 하나의 책임을 넘어서기 시작할 때 분리한다.

---

## 6. API 계층 구조

API 계층은 별도 최상위 `api` 패키지에 모든 Controller를 모으지 않고, 각 도메인 패키지 내부의 `controller`와 `dto`에서 관리한다. API URL 그룹은 도메인 경계를 기준으로 정의한다.

### 6.1 주요 API 그룹

```
/auth
/users
/conversations
/characters
/emotions
/todos
/notifications
/admin
```

### 6.2 API 책임 분리

Controller는 다음 책임만 가진다.

```
요청 DTO 바인딩
입력값 검증
인증 사용자 식별
Application Service 호출
응답 DTO 변환
HTTP status 반환
```

Controller에서 직접 처리하지 않는 항목은 다음과 같다.

```
비즈니스 정책 판단
DB 직접 접근
외부 API 직접 호출
트랜잭션 복합 처리
도메인 상태 변경 로직
```

---

## 7. 데이터베이스 아키텍처

### 7.1 데이터베이스 선택

Mio v1은 AWS RDS PostgreSQL을 사용한다.

```
DB Engine: PostgreSQL
Deployment: AWS RDS
Migration: Flyway
Application Access: JDBC / JPA
```

### 7.2 핵심 테이블 그룹

```
사용자/인증
- users
- user_oauth_accounts
- user_consents
- refresh_tokens

대화
- conversation_sessions
- conversation_messages

캐릭터
- character_profiles
- prompt_templates

감정
- emotion_checkins

행동 To-do
- todo_items

알림
- push_tokens
- notifications
- notification_settings

관리/정책
- policy_versions
- policy_decisions

감사
- audit_events

비동기 이벤트
- outbox_events
```

### 7.3 스키마 관리 원칙

DB 스키마 변경은 Flyway migration 파일로 관리한다.

```
수동 DB 변경 금지
모든 DDL은 migration 파일로 관리
migration 파일은 Git으로 버전 관리
배포 시 애플리케이션 시작 과정에서 migration 적용
```

### 7.4 Flyway 디렉터리 구조

```
src/main/resources/db/migration
  ├─ V1__init_users.sql
  ├─ V2__init_conversation.sql
  ├─ V3__init_emotion_todo.sql
  ├─ V4__init_notification.sql
  ├─ V5__init_admin_policy.sql
  ├─ V6__init_audit_outbox.sql
  └─ ...
```

### 7.5 Flyway 설정

```yaml
spring:
flyway:
enabled:true
locations: classpath:db/migration
baseline-on-migrate:false
```

운영 DB에서는 이미 적용된 migration 파일을 수정하지 않는다. 변경이 필요한 경우 새로운 migration 파일을 추가한다.

---

## 8. 인증 및 권한 구조

### 8.1 인증 방식

Mio v1은 OAuth 기반 소셜 로그인을 사용한다.

```
Apple Login
Kakao Login
JWT Access Token
Refresh Token
```

### 8.2 권한 모델

초기 권한 모델은 단순하게 유지한다.

```
USER
ADMIN
```

### 8.3 인증 흐름

```
Client
  ↓ OAuth authorization
OAuth Provider
  ↓ authorization code / token
Spring Boot Auth Module
  ↓ user lookup or create
JWT 발급
  ↓
Client stores token
```

### 8.4 API 인증 처리

```
Client Request
  ↓ Authorization: Bearer {accessToken}
Spring Security Filter
  ↓ JWT 검증
Authenticated Principal 생성
  ↓ Controller 진입
```

---

## 9. 배포 아키텍처

### 9.1 배포 구성요소

```
GitHub Repository
GitHub Actions
GHCR
운영 서버
Docker Compose
Nginx Container
Spring Boot App Container
AWS RDS PostgreSQL
```

### 9.2 CI/CD 흐름

```
Developer Commit
  ↓
Git Push
  ↓
GitHub Actions
  ├─ checkout
  ├─ setup JDK
  ├─ test
  ├─ build
  ├─ docker build
  ├─ docker login ghcr.io
  ├─ docker push GHCR
  └─ deploy via SSH

        ↓

Production Server
  ├─ docker login ghcr.io
  ├─ docker compose pull
  ├─ docker compose up -d
  ├─ health check
  └─ old image cleanup
```

### 9.3 이미지 저장소

Docker 이미지는 GHCR에 저장한다.

```
ghcr.io/{owner}/{repository}:latest
ghcr.io/{owner}/{repository}:{git-sha}
```

이미지는 `latest`와 `git-sha` 태그를 함께 사용한다.

```
latest: 현재 운영 배포 대상
git-sha: 특정 커밋 기반 롤백 대상
```

### 9.4 Docker Compose 구성

운영 서버의 Docker Compose는 Nginx와 Spring Boot 애플리케이션을 관리한다.

```yaml
services:
nginx:
image: nginx:stable
ports:
-"80:80"
-"443:443"
volumes:
- ./nginx/conf.d:/etc/nginx/conf.d
- ./certbot/www:/var/www/certbot
- ./certbot/conf:/etc/letsencrypt
depends_on:
- app
restart: always

app:
image: ghcr.io/{owner}/{repository}:latest
env_file:
- .env
expose:
-"8080"
restart: always
```

PostgreSQL은 Docker Compose에 포함하지 않고 RDS를 사용한다.

---

## 10. Nginx 구성

### 10.1 기본 역할

```
80 포트 HTTP 요청 수신
443 포트 HTTPS 요청 수신
HTTP 요청을 HTTPS로 redirect
Spring Boot 애플리케이션으로 reverse proxy
```

### 10.2 Spring Boot 프록시 설정 예시

```
server {
    listen 80;
    server_name api.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;

    location / {
        proxy_pass http://app:8080;

        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_buffering off;
        proxy_cache off;
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;
    }
}
```

### 10.3 Spring Boot reverse proxy 설정

```yaml
server:
forward-headers-strategy: framework
```

---

## 11. 환경변수 및 시크릿 관리

### 11.1 관리 방식

Mio v1은 별도 Secrets Manager를 사용하지 않는다. 환경변수와 서버 내 `.env` 파일을 사용한다.

```
GitHub Actions Secrets
운영 서버 .env
Docker Compose env_file
```

### 11.2 운영 서버 환경변수 파일

운영 서버에는 다음 경로에 `.env` 파일을 둔다.

```
/opt/mio/.env
```

예시:

```
SPRING_PROFILES_ACTIVE=prod

DB_URL=jdbc:postgresql://{rds-endpoint}:5432/mio
DB_USERNAME=mio_app
DB_PASSWORD=********

JWT_SECRET=********
JWT_ACCESS_TOKEN_TTL=3600
JWT_REFRESH_TOKEN_TTL=1209600

APPLE_CLIENT_ID=********
APPLE_TEAM_ID=********
APPLE_KEY_ID=********
APPLE_PRIVATE_KEY=********

KAKAO_CLIENT_ID=********
KAKAO_CLIENT_SECRET=********

OPENAI_API_KEY=********

FCM_CREDENTIALS_JSON=********

APP_ENCRYPTION_KEY=********

LOG_LEVEL=INFO
```

### 11.3 시크릿 관리 규칙

```
.env 파일은 Git repository에 포함하지 않는다.
시크릿 값은 CI/CD 로그에 출력하지 않는다.
운영 서버의 .env 파일 접근 권한을 제한한다.
GitHub Actions Secrets에는 배포에 필요한 최소값만 저장한다.
DB 계정은 애플리케이션 전용 계정을 사용한다.
```

---

## 12. 로깅 및 관측성

### 12.1 로깅 방식

초기 운영에서는 별도 고도화된 observability stack을 사용하지 않는다. 애플리케이션 로그와 Nginx 로그를 기본으로 사용한다.

```
Spring Boot application log
Nginx access log
Nginx error log
Docker container log
GitHub Actions deploy log
```

### 12.2 애플리케이션 로그 필드

애플리케이션 로그는 구조화된 형태로 남긴다.

```
timestamp
level
requestId
method
path
status
latencyMs
userId 또는 userHash
sessionId
messageId
errorCode
exceptionClass
```

### 12.3 로그 금지 항목

다음 데이터는 로그에 남기지 않는다.

```
대화 원문
LLM raw prompt
OAuth access token
JWT 원문
API key
DB password
개인식별정보 원문
민감한 사용자 입력
```

### 12.4 헬스체크

Spring Boot Actuator를 사용해 헬스체크 endpoint를 제공한다.

```
GET /actuator/health
```

배포 후 GitHub Actions 또는 서버 deploy script에서 해당 endpoint를 확인한다.

---

## 13. Outbox 기반 비동기 처리

### 13.1 Outbox 사용 목적

초기 버전에서는 Kafka를 사용하지 않는다. 비동기 작업은 PostgreSQL outbox 테이블과 애플리케이션 내부 worker로 처리한다.

### 13.2 Outbox 처리 대상

```
알림 발송
감정/행동 통계 집계
Audit 후처리
외부 API 재시도
운영 이벤트 후처리
```

### 13.3 Outbox 테이블 개념

```
outbox_events
  ├─ id
  ├─ aggregate_type
  ├─ aggregate_id
  ├─ event_type
  ├─ payload
  ├─ status
  ├─ retry_count
  ├─ next_retry_at
  ├─ created_at
  └─ published_at
```

### 13.4 Outbox 상태

```
PENDING
PROCESSING
PUBLISHED
FAILED
DEAD_LETTER
```

---

## 14. Audit 구조

### 14.1 Audit 대상

```
관리자 로그인
관리자 설정 변경
Prompt template 변경
Policy version 변경
사용자 상태 변경
중요한 보안/안전 이벤트 메타데이터
중요한 시스템 오류
```

### 14.2 Audit 테이블 개념

```
audit_events
  ├─ id
  ├─ actor_type
  ├─ actor_id
  ├─ action
  ├─ target_type
  ├─ target_id
  ├─ payload
  ├─ request_id
  ├─ created_at
```

### 14.3 Audit 저장 원칙

```
원문 대화 내용은 audit payload에 저장하지 않는다.
민감 입력 원문은 저장하지 않는다.
이벤트 재현에 필요한 메타데이터 중심으로 저장한다.
관리자 변경 이벤트는 반드시 audit에 남긴다.
```

---

## 15. 장애 및 운영 고려사항

### 15.1 단일 서버 장애

Nginx와 Spring Boot 애플리케이션은 동일 서버에서 Docker Compose로 실행된다. 해당 서버 장애 시 API 서비스가 중단된다.

### 15.2 DB 장애

PostgreSQL은 RDS를 사용한다. 애플리케이션은 DB 연결 실패 시 오류를 반환하고, DB 복구 후 정상 동작한다.

### 15.3 인증서 만료

Nginx는 Let’s Encrypt 인증서를 사용한다. 인증서 갱신은 certbot 또는 별도 갱신 스크립트를 통해 수행한다.

### 15.4 배포 실패

Docker 이미지는 `git-sha` 태그를 함께 유지한다. 배포 실패 또는 장애 발생 시 이전 이미지 태그로 롤백한다.

### 15.5 외부 API 장애

OAuth, Push Provider, LLM Provider 등 외부 서비스 장애는 애플리케이션 레벨에서 timeout과 fallback 정책으로 처리한다.

---

## 16. 백업 및 복구

### 16.1 RDS 백업

PostgreSQL 데이터는 RDS의 자동 백업 기능을 사용한다.

```
자동 백업 활성화
백업 보존 기간 설정
필요 시 수동 snapshot 생성
운영 배포 전 snapshot 생성 가능
```

### 16.2 애플리케이션 서버 복구

애플리케이션 서버는 Docker Compose와 `.env` 파일을 기준으로 복구한다.

복구에 필요한 항목:

```
Docker
Docker Compose
/opt/mio/docker-compose.yml
/opt/mio/.env
Nginx 설정 파일
인증서 파일 또는 재발급 절차
GHCR 접근 권한
```

---

## 17. 운영 디렉터리 구조

운영 서버의 기본 디렉터리 구조는 다음과 같다.

```
/opt/mio
  ├─ docker-compose.yml
  ├─ .env
  ├─ nginx
  │   └─ conf.d
  │       └─ api.conf
  ├─ certbot
  │   ├─ www
  │   └─ conf
  └─ scripts
      ├─ deploy.sh
      ├─ rollback.sh
      └─ healthcheck.sh
```

---

## 18. 배포 스크립트 개념

### 18.1 deploy.sh

```bash
#!/usr/bin/env bash
set -e

cd /opt/mio

docker login ghcr.io -u "$GHCR_USERNAME" -p "$GHCR_TOKEN"
docker compose pull app
docker compose up -d app
docker image prune -f

./scripts/healthcheck.sh
```

### 18.2 healthcheck.sh

```bash
#!/usr/bin/env bash
set -e

curl -f https://api.example.com/actuator/health
```

### 18.3 rollback.sh

```bash
#!/usr/bin/env bash
set -e

TARGET_IMAGE_TAG=$1

if [ -z "$TARGET_IMAGE_TAG" ]; then
  echo "Usage: ./rollback.sh <image-tag>"
  exit 1
fi

cd /opt/mio

export APP_IMAGE="ghcr.io/{owner}/{repository}:${TARGET_IMAGE_TAG}"
docker compose up -d app
./scripts/healthcheck.sh
```

---

## 19. 시스템 아키텍처 최종 형태

```
[Mobile Client / Admin Web]
        ↓ HTTPS
[Nginx]
        ↓ Internal HTTP
[Spring Boot Application]
        ↓ JDBC
[AWS RDS PostgreSQL]

[CI/CD]
Git Push
  ↓
GitHub Actions
  ↓
GHCR
  ↓
SSH Deploy
  ↓
Docker Compose Pull/Up

[Ops]
- Flyway migration
- Spring Actuator health check
- Docker logs
- Nginx logs
- RDS automated backup
- Audit events
- Outbox events
```

---

## 20. 구성요소 요약

| 영역 | 선택 기술 |
| --- | --- |
| Backend Framework | Spring Boot |
| Reverse Proxy | Nginx |
| HTTPS | Nginx + Let’s Encrypt |
| Database | AWS RDS PostgreSQL |
| DB Migration | Flyway |
| Container | Docker |
| Runtime Deploy | Docker Compose |
| CI/CD | GitHub Actions |
| Image Registry | GHCR |
| Secret 관리 | GitHub Actions Secrets + 서버 `.env` |
| Logging | Spring Boot logs + Nginx logs + Docker logs |
| Health Check | Spring Boot Actuator |
| Async 처리 | PostgreSQL Outbox |
| Audit | PostgreSQL audit_events |

---

## 21. 제외 범위

본 아키텍처의 v1 초기 구성에서 제외하는 항목은 다음과 같다.

```
ALB
ECS
EKS
Kafka / MSK
OpenSearch
Redshift / EMR
KMS
Secrets Manager
Service Mesh
Multi-region
전용 Observability Platform
별도 DB Proxy
별도 API Gateway
```

---

## 22. 최종 정리

Mio v1 시스템 아키텍처는 Nginx, Spring Boot, RDS PostgreSQL, Flyway, GitHub Actions, GHCR, Docker Compose를 중심으로 구성한다.

클라이언트 요청은 HTTPS로 Nginx에 도달하고, Nginx는 Spring Boot 애플리케이션으로 요청을 프록시한다. Spring Boot 애플리케이션은 단일 배포 단위로 동작하며, 내부적으로 인증, 사용자, 대화, 감정, To-do, 알림, 관리자, 감사, Outbox 모듈을 가진다.

데이터는 RDS PostgreSQL에 저장하며, 모든 스키마 변경은 Flyway migration으로 관리한다. 배포는 GitHub Actions에서 Docker 이미지를 빌드하여 GHCR에 push하고, 운영 서버에서 Docker Compose로 최신 이미지를 배포하는 방식으로 수행한다.