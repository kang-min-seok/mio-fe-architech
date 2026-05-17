# API 명세서 (v1.1.1)

Mio 백엔드 API 명세서 모음 (v1.1.1 기준)

각 도메인별 엔드포인트, 요청/응답 스펙, 에러코드, 프론트엔드 연동 가이드를 포함합니다.

## 도메인 목록

| 번호 | 도메인 | 주요 기능 |
| --- | --- | --- |
| 00 | 공통 및 공통 에러코드 | 인증 헤더, 응답 형식, 에러코드 전체 |
| 01 | Auth (인증) | Kakao/Apple OAuth, JWT, 회원가입, 탈퇴 |
| 02 | Onboarding (온보딩) | 온보딩 제출, 캐릭터 매칭 |
| 03 | Character (캐릭터) | 캐릭터 목록·선택·변경 |
| 04 | Session (세션·메시지) | AI 채팅, SSE 이벤트 전체 |
| 05 | CheckIn (체크인) | 아침·점심·저녁 감정 기록 |
| 06 | Daily Test (데일리 테스트) | 심리 테스트 |
| 07 | Todo (투두) | To-do 생성·실행·기록 |
| 08 | Report (리포트) | 주간·월간 리포트 |
| 09 | Crisis (위기) | 위기 리소스·플래그 |
| 10 | Notification (알림) | 알림 설정·디바이스 토큰·선제 개입 |
| 11 | Subscription (구독) | Apple/Google 구독 검증·Webhook |
| 12 | Admin (운영자) | 회원 제재·위기 이벤트 검토 큐 |

[00. 공통 및 공통 에러코드](00-common-error-codes.md)

[01. Auth (인증)](01-auth.md)

[02. Onboarding (온보딩)](02-onboarding.md)

[03. Character (캐릭터)](03-character.md)

[04. Session / Message (세션·메시지)](04-session-message.md)

[05. CheckIn (체크인)](05-checkin.md)

[06. Daily Test (데일리 테스트)](06-daily-test.md)

[07. Todo (투두)](07-todo.md)

[08. Report (리포트)](08-report.md)

[09. Crisis (위기)](09-crisis.md)

[10. Notification (알림)](10-notification.md)

[11. Subscription (구독)](11-subscription.md)

[12. Admin (운영자)](12-admin.md)