# 폴더 구조 (전체)

> [← 인덱스로 돌아가기](./index.md)

---

```
📦 project-root/
├── app/                          # Expo Router 라우트
│   ├── (auth)/
│   │   ├── login.tsx
│   │   └── onboarding/
│   │       ├── _layout.tsx
│   │       ├── step1-emotion.tsx
│   │       ├── step2-concern.tsx
│   │       ├── step3-style.tsx
│   │       └── step4-character.tsx
│   ├── (main)/
│   │   ├── _layout.tsx           # Tab Navigator
│   │   ├── home/index.tsx
│   │   ├── checkin/
│   │   │   ├── index.tsx
│   │   │   └── history.tsx
│   │   ├── chat/
│   │   │   ├── index.tsx
│   │   │   └── restructure.tsx
│   │   ├── report/index.tsx
│   │   └── my/
│   │       ├── index.tsx
│   │       ├── partner.tsx
│   │       └── settings.tsx
│   ├── mind-explore/
│   │   ├── _layout.tsx           # 모달 스택
│   │   ├── index.tsx
│   │   ├── [stageId].tsx
│   │   └── result.tsx
│   └── _layout.tsx               # Root layout (AuthGuard)
│
├── src/
│   ├── components/               # 공통 컴포넌트 (05-components.md 참조)
│   │   ├── ui/
│   │   ├── layout/
│   │   ├── character/
│   │   └── emotion/
│   │
│   ├── stores/                   # Zustand stores
│   │   ├── authStore.ts
│   │   ├── onboardingStore.ts
│   │   ├── chatStore.ts
│   │   ├── checkinStore.ts
│   │   └── mindExploreStore.ts
│   │
│   ├── queries/                  # TanStack Query hooks
│   │   ├── useAuth.ts
│   │   ├── useCheckin.ts
│   │   ├── useChat.ts
│   │   ├── useReport.ts
│   │   └── useMindExplore.ts
│   │
│   ├── api/                      # Axios 클라이언트 + API 함수
│   │   ├── client.ts             # Axios 인스턴스 + 인터셉터
│   │   ├── auth.api.ts
│   │   ├── checkin.api.ts
│   │   ├── chat.api.ts
│   │   ├── report.api.ts
│   │   └── mindExplore.api.ts
│   │
│   ├── types/                    # 공통 TypeScript 타입
│   │   ├── auth.types.ts
│   │   ├── emotion.types.ts
│   │   ├── chat.types.ts
│   │   ├── report.types.ts
│   │   └── character.types.ts
│   │
│   ├── constants/
│   │   ├── emotions.ts           # 감정 이모지 + 감정명 매핑
│   │   ├── characters.ts         # 캐릭터 데이터
│   │   └── colors.ts             # 디자인 토큰
│   │
│   ├── hooks/                    # 커스텀 훅
│   │   ├── useAuth.ts
│   │   ├── useKeyboard.ts
│   │   └── useDebounce.ts
│   │
│   ├── notifications/            # FCM 푸시 알림
│   │   ├── fcm.ts                # 토큰 등록/갱신
│   │   ├── handleNotificationTap.ts  # 딥링크 라우팅
│   │   └── notificationTypes.ts  # 알림 타입 상수 + payload 타입
│   │
│   └── utils/
│       ├── date.ts               # date-fns 래퍼
│       ├── storage.ts            # expo-secure-store 래퍼
│       └── format.ts
│
├── assets/
│   ├── images/
│   ├── animations/               # Lottie JSON
│   └── fonts/
│
├── app.json
├── babel.config.js
├── tsconfig.json
└── package.json
```

---

## 관련 문서

- [네비게이션 구조](./03-navigation.md) — `app/` 라우트 상세
- [공통 컴포넌트](./05-components.md) — `src/components/` 상세
- [상태 관리 설계](./07-state-management.md) — `src/stores/` + `src/queries/` 상세
