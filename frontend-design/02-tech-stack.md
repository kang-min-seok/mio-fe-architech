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

| 분류 | 기술 |
|---|---|
| 차트 | `victory-native` 또는 `react-native-gifted-charts` |
| 채팅 UI | 커스텀 FlatList |
| 슬라이더 | `@miblanchard/react-native-slider` |
| 아이콘 | `expo-vector-icons` (Ionicons) |
| 날짜 | `date-fns` |
| 환경변수 | `expo-constants` + `.env` |
| 푸시 알림 | `expo-notifications` + Firebase FCM |

---

## 관련 문서

- [폴더 구조](./06-folder-structure.md)
- [상태 관리 설계](./07-state-management.md)
