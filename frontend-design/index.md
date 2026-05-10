# AI 정서 케어 앱 — 프론트엔드 설계 문서 인덱스

> 작성일: 2026-05-07  
> 플랫폼: React Native + Expo  
> 상태: 초안 (v0.1)

---

## 문서 목록

| # | 문서 | 설명 |
|---|---|---|
| 01 | [프로젝트 개요](./01-overview.md) | 서비스 소개, 핵심 사용자 플로우 |
| 02 | [기술 스택](./02-tech-stack.md) | 코어 라이브러리 선택 이유, 기타 라이브러리 |
| 03 | [네비게이션 구조](./03-navigation.md) | Expo Router 파일 기반 라우트 트리 |
| 04-01 | [페이지 설계 — 온보딩](./04-pages-onboarding.md) | 스텝 구성, 컴포넌트, 상태/쿼리 |
| 04-02 | [페이지 설계 — 홈](./04-pages-home.md) | 섹션 구성, 컴포넌트, 쿼리 |
| 04-03 | [페이지 설계 — 체크인](./04-pages-checkin.md) | 체크인 플로우, 기록 화면, 쿼리 |
| 04-04 | [페이지 설계 — 채팅](./04-pages-chat.md) | AI 대화, 생각 재구성, 메시지 타입, 쿼리 |
| 04-05 | [페이지 설계 — 리포트](./04-pages-report.md) | 기간 탭, 차트 구성, 쿼리 |
| 04-06 | [페이지 설계 — 마이페이지](./04-pages-mypage.md) | 프로필, AI 파트너 변경, 설정, 재구성 기록, 감정 통계 |
| 04-07 | [페이지 설계 — 마음 탐색](./04-pages-mind-explore.md) | 스테이지 플로우, 결과 화면, 상태/쿼리 |
| 05 | [공통 컴포넌트](./05-components.md) | 사용처 교차 참조, 컴포넌트 트리 |
| 06 | [폴더 구조](./06-folder-structure.md) | 전체 프로젝트 디렉토리 구조 |
| 07 | [상태 관리 설계](./07-state-management.md) | Zustand store 설계, 페이지별 의존성, TanStack Query 캐시 전략 |
| 08 | [다음 설계 단계](./08-next-steps.md) | 남은 작업 체크리스트 |
| 09 | [푸시 알림 설계](./09-push-notifications.md) | FCM 알림 타입, 토큰 관리, 딥링크 처리 |
| 10 | [백엔드 API 연동 가이드](./10-backend-api-guide.md) | 인증 흐름, API 그룹, SSE 확정, 응답 포맷, 감정 타입 정합성, AI 동작 |

---

## 핵심 사용자 플로우 (요약)

```
소셜 로그인 → 온보딩 (감정/고민/대화방식/캐릭터 선택)
  └→ 홈 (감정 별자리, 체크인, 마음탐색, 리포트, TODO)
       ├→ 체크인 (감정 선택 → 강도 → 일기)
       ├→ 채팅 (AI 캐릭터 대화 → CBT 생각 재구성)
       ├→ 리포트 (감정 추이 그래프 + 트리거 분석)
       ├→ 마음 탐색 (스토리형 심리 테스트 → 결과)
       └→ 마이페이지 (파트너 변경, 설정)
```

---

## 기술 스택 한눈에 보기

| 분류 | 기술 |
|---|---|
| 프레임워크 | React Native + Expo SDK 51+ |
| 언어 | TypeScript |
| 네비게이션 | React Navigation (Expo Router) |
| 상태 관리 | Zustand |
| 서버 통신 | TanStack Query v5 + Axios |
| 스타일링 | NativeWind |
| 애니메이션 | Reanimated 3 + Lottie |
| 푸시 알림 | Firebase FCM + expo-notifications |

---

## 원본 문서

- [frontend-design-plan.md](../frontend-design-plan.md) — 통합 원본 문서 (분리 전)
