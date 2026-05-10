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
│   │   ├── _layout.tsx           # Tab Navigator + 탭바 숨김 조건 처리
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
│   │       ├── partner.tsx       # 파트너 변경
│   │       ├── settings.tsx
│   │       ├── records.tsx       # 재구성 기록 목록
│   │       └── memory.tsx        # AI 기억 관리
│   ├── mind-explore/
│   │   ├── _layout.tsx           # 모달 스택
│   │   ├── index.tsx
│   │   ├── [stageId].tsx
│   │   └── result.tsx
│   └── _layout.tsx               # Root layout (AuthGuard — isAuthenticated + isOnboarded 분기)
│
├── src/
│   ├── components/               # 공통 컴포넌트 (05-components.md 참조)
│   │   ├── ui/
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   ├── Card/
│   │   │   ├── Slider/
│   │   │   ├── Feedback/         # ErrorState, LoadingSkeleton
│   │   │   └── Toast/
│   │   ├── layout/
│   │   ├── character/
│   │   └── emotion/
│   │
│   ├── stores/                   # Zustand stores
│   │   ├── authStore.ts
│   │   ├── onboardingStore.ts
│   │   ├── chatStore.ts
│   │   ├── checkinStore.ts
│   │   ├── partnerStore.ts       # PartnerScreen 임시 선택 상태
│   │   └── mindExploreStore.ts
│   │
│   ├── queries/                  # TanStack Query hooks
│   │   ├── queryClient.ts        # QueryClient 인스턴스 + 전역 defaultOptions
│   │   ├── useAuth.ts
│   │   ├── useHome.ts            # home-today, constellation, todos
│   │   ├── useCheckin.ts
│   │   ├── useChat.ts            # useChatSSE 훅 포함
│   │   ├── useReport.ts
│   │   ├── useMyPage.ts          # profile, partner list, settings, records, memory
│   │   ├── useTodo.ts
│   │   └── useMindExplore.ts
│   │
│   ├── api/                      # Axios 클라이언트 + API 함수
│   │   ├── client.ts             # Axios 인스턴스 + 인터셉터 (401 갱신, 403 처리)
│   │   ├── auth.api.ts
│   │   ├── checkin.api.ts
│   │   ├── chat.api.ts           # sendMessage (SSE 스트림 연결 포함)
│   │   ├── report.api.ts
│   │   ├── character.api.ts      # 캐릭터 목록 조회
│   │   ├── my.api.ts             # 프로필, 파트너 변경, 설정, 기억 관리
│   │   ├── todo.api.ts
│   │   ├── notification.api.ts   # FCM 토큰 등록·갱신
│   │   └── mindExplore.api.ts
│   │
│   ├── types/                    # 공통 TypeScript 타입
│   │   ├── auth.types.ts
│   │   ├── emotion.types.ts      # EmotionType (7종), EmotionMeta (이모지·라벨·색상)
│   │   ├── chat.types.ts
│   │   ├── report.types.ts
│   │   └── character.types.ts
│   │
│   ├── constants/
│   │   ├── emotions.ts           # EmotionType → 이모지·한글 라벨·색상 매핑
│   │   ├── characters.ts         # 캐릭터 데이터
│   │   └── colors.ts             # 디자인 토큰 (11-design-tokens.md 기준)
│   │
│   ├── hooks/                    # 커스텀 훅
│   │   ├── useAuth.ts
│   │   ├── useKeyboard.ts
│   │   ├── useDebounce.ts
│   │   └── useToast.ts           # Toast 노출 유틸 훅
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
├── tailwind.config.js            # NativeWind 커스텀 색상·폰트 (11-design-tokens.md 기준)
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
- [디자인 토큰](./11-design-tokens.md) — `tailwind.config.js` + `src/constants/colors.ts` 기준값
