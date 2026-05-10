# 네비게이션 구조

> [← 인덱스로 돌아가기](./index.md)

Expo Router의 파일 기반 라우팅 기준.

---

## 라우트 트리

```
app/
├── (auth)/                   # 비로그인 그룹
│   ├── login.tsx             # 소셜 로그인
│   └── onboarding/
│       ├── step1-emotion.tsx # 감정 선택
│       ├── step2-concern.tsx # 주요 고민
│       ├── step3-style.tsx   # 대화 방식
│       └── step4-character.tsx # 캐릭터 선택
│
├── (main)/                   # 로그인 후 탭 네비게이터
│   ├── _layout.tsx           # Bottom Tab Navigator
│   ├── home/
│   │   └── index.tsx
│   ├── checkin/
│   │   ├── index.tsx         # 오늘의 체크인
│   │   └── history.tsx       # 체크인 기록
│   ├── chat/
│   │   ├── index.tsx         # 메인 채팅
│   │   └── restructure.tsx   # 생각 재구성 상세
│   ├── report/
│   │   └── index.tsx
│   └── my/
│       ├── index.tsx
│       ├── partner.tsx       # 파트너 변경
│       ├── settings.tsx
│       ├── records.tsx       # 재구성 기록 목록
│       └── memory.tsx        # AI 기억 관리
│
└── mind-explore/             # 모달 스택 (홈에서 presentModal)
    ├── index.tsx             # 오프닝
    ├── [stageId].tsx         # 스토리 스테이지
    └── result.tsx            # 결과
```

---

## 그룹 설명

| 그룹 | 경로 | 설명 |
|---|---|---|
| 비로그인 | `(auth)/` | 로그인 전 접근 가능한 화면 |
| 메인 탭 | `(main)/` | Bottom Tab Navigator로 구성 |
| 마음 탐색 | `mind-explore/` | 홈에서 모달 스택으로 진입 (`router.push('/mind-explore')`) |

---

## AuthGuard 분기 로직

`app/_layout.tsx` (Root layout)에서 `authStore`를 구독해 화면을 분기한다.

```
앱 진입
  │
  ├─ isAuthenticated === false → /login (비로그인)
  │
  └─ isAuthenticated === true
        │
        ├─ isOnboarded === false → /onboarding/step1-emotion (최초 가입)
        │
        └─ isOnboarded === true  → /(main)/home (기존 회원)
```

- `isAuthenticated`와 `isOnboarded` 모두 `authStore`에서 관리 (07-state-management.md 참조)
- 온보딩 완료(`useMutation` 성공) 후 `setOnboarded()` 호출 → Root layout이 자동으로 `/(main)/home`으로 리다이렉트
- 온보딩 진행 중 앱 종료 후 재진입 시: `isOnboarded === false`이므로 step1부터 재시작

---

## 탭바 숨김/표시 조건

`(main)/_layout.tsx`의 Bottom Tab Navigator에서 `tabBarStyle` 조건 처리.

| 화면 | 탭바 표시 |
|---|---|
| `home`, `checkin`, `chat`, `report`, `my` index | ✅ 표시 |
| `checkin/history` | ✅ 표시 |
| `chat/restructure` | ❌ 숨김 (전체 화면 집중 모드) |
| `my/partner`, `my/settings`, `my/records`, `my/memory` | ❌ 숨김 (서브 페이지) |
| `mind-explore/*` | ❌ 숨김 (모달 스택) |

---

## 뒤로가기 예외 케이스

| 상황 | 처리 방식 |
|---|---|
| 체크인 진행 중 (`intensity`·`diary` 스텝) 뒤로가기 | Alert("체크인을 그만할까요?") 확인 → `checkinStore.reset()` 후 홈 이동 |
| 온보딩 step1에서 뒤로가기 | 뒤로가기 비활성 (하드웨어 버튼 포함) — 로그아웃 전까지 건너뛸 수 없음 |
| 생각 재구성 화면에서 뒤로가기 | Alert 없이 즉시 채팅으로 복귀 (`restructurePrompt` 유지) |
| 마음 탐색 진행 중 닫기 탭 | Alert("탐색을 그만할까요?") 확인 → `mindExploreStore.reset()` 후 홈 |

---

## 딥링크(알림) 진입 시 네비게이션 스택 처리

푸시 알림 탭 시 딥링크로 특정 화면에 직접 진입한다. 이때 뒤로가기 스택이 없으므로 아래 규칙을 적용한다.

| 알림 유형 | 딥링크 경로 | 스택 처리 |
|---|---|---|
| 체크인 리마인드 | `/(main)/checkin` | 탭 네비게이터 내에서 checkin 탭으로 이동 (스택 유지) |
| To-do 리마인드 | `/(main)/home` | 탭 네비게이터 내에서 home 탭으로 이동 |

- 딥링크 진입 후 뒤로가기 시 홈 탭으로 fallback
- 앱이 완전히 종료된 상태에서 알림 탭 → `handleNotificationTap.ts`에서 `router.replace()`로 스택 없이 이동

---

## 관련 문서

- [페이지 설계 — 온보딩](./04-pages-onboarding.md)
- [페이지 설계 — 홈](./04-pages-home.md)
- [페이지 설계 — 마음 탐색](./04-pages-mind-explore.md)
- [상태 관리 설계](./07-state-management.md) — authStore (isOnboarded)
- [폴더 구조](./06-folder-structure.md)
- [푸시 알림 설계](./09-push-notifications.md) — 딥링크 처리
