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
│       └── settings.tsx
│
└── mind-explore/             # 모달 or 별도 스택
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
| 마음 탐색 | `mind-explore/` | 홈에서 모달 스택으로 진입 |

---

## 관련 문서

- [페이지 설계 — 온보딩](./04-pages-onboarding.md)
- [페이지 설계 — 홈](./04-pages-home.md)
- [페이지 설계 — 마음 탐색](./04-pages-mind-explore.md)
- [폴더 구조](./06-folder-structure.md)
