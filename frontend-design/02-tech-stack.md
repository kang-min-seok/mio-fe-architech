# 기술 스택

> [← 인덱스로 돌아가기](./index.md)

---

## 코어

| 분류 | 기술 | 선택 이유 |
|---|---|---|
| 프레임워크 | React Native + Expo SDK 51+ | 크로스플랫폼, 빠른 개발 사이클 |
| 언어 | TypeScript | 타입 안정성 |
| 네비게이션 | React Navigation | 높은 유연성, 쉬운 사용법 |
| 상태 관리 | Zustand | 경량, 보일러플레이트 없음 |
| 서버 통신 | TanStack Query v5 | 캐싱, 낙관적 업데이트, 무한스크롤 |
| 스타일링 | NativeWind | 다크 테마 우선 UI에 적합 |
| 애니메이션 | Reanimated 3 + Lottie | 감정 별자리, 페이드인 등 풍부한 모션 |
| 폼 관리 | React Hook Form + Zod | 체크인 일기, 온보딩 입력 검증 |
| 인증 자동 갱신 | TanStack Query + Axios 인터셉터 | |

> **애니메이션 역할 분담**
> - Reanimated 3 → 상호작용 애니메이션
> - Lottie → 영상형 애니메이션

---

## 기타

| 분류 | 기술 | 비고 |
|---|---|---|
| 차트 | `victory-native` | ✅ 확정 — SVG 기반, Expo/RN 공식 지원 |
| 채팅 UI | 커스텀 FlatList | |
| 슬라이더 | `@miblanchard/react-native-slider` | |
| 아이콘 | `expo-vector-icons` (Ionicons) | |
| 날짜 | `date-fns` | |
| 환경변수 | `expo-constants` + `.env` | |
| 푸시 알림 | `expo-notifications` + Firebase FCM | |
| SSE 스트리밍 | `@microsoft/fetch-event-source` | React Native 환경에서 EventSource 대체 — fetch 기반, Authorization 헤더 지원 |
| 폼 관리 | `react-hook-form` + `zod` | 사용처: 체크인 일기(200자 제한), 마음탐색 닉네임(필수·최대 20자), 마이페이지 닉네임 수정 |

---

## 테마 방향

- **기본값:** 다크 모드
- **라이트 지원:** 시스템 설정(`useColorScheme`) 또는 마이페이지 설정에서 전환 가능
- **구현 방식:** NativeWind `dark:` prefix 클래스 + `tailwind.config.js` 커스텀 색상 — 상세 토큰은 [디자인 토큰](./11-design-tokens.md) 참조

---

## 관련 문서

- [폴더 구조](./06-folder-structure.md)
- [상태 관리 설계](./07-state-management.md)
