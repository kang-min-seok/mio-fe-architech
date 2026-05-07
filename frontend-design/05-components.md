# 공통 컴포넌트 (Shared)

> [← 인덱스로 돌아가기](./index.md)

**기준:** UI 원시(버튼·인풋), 앱 전반 디자인 언어(GlassCard), 2곳 이상에서 완전히 동일한 동작이 필요한 것만 공통으로 분리. 그 외 단일 화면 전용이거나 화면마다 커스터마이징이 더 중요한 컴포넌트는 페이지 로컬로 유지.

---

## 사용처 교차 참조

| 컴포넌트 | 온보딩 | 홈 | 체크인 | 채팅 | 리포트 | 마이페이지 | 마음탐색 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Button (Primary/Secondary/Icon) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| TextInput / MultilineInput | ✓ | | ✓ | ✓ | | | ✓ |
| GlassCard | ✓ | ✓ | ✓ | ✓ | ✓ | | ✓ |
| IntensitySlider | | | ✓ | ✓ | | | |
| CharacterAvatar | ✓ | | | ✓ | ✓ | ✓ | ✓ |
| EmotionEmoji / EmojiRow | ✓ | ✓ | ✓ | | | | |

---

## 컴포넌트 트리

```
components/
├── ui/
│   ├── Button/
│   │   ├── PrimaryButton.tsx
│   │   ├── SecondaryButton.tsx
│   │   └── IconButton.tsx
│   │
│   ├── Input/
│   │   ├── TextInput.tsx
│   │   └── MultilineInput.tsx
│   │
│   ├── Card/
│   │   └── GlassCard.tsx          # 다크 반투명 카드 베이스
│   │                               #   → 온보딩/홈/체크인/채팅/리포트/마음탐색
│   │
│   └── Slider/
│       └── IntensitySlider.tsx    # 0~100, 중앙 숫자 표시
│                                   #   → 체크인 강도 입력, 채팅 EmotionIntensityCard,
│                                   #     생각 재구성 슬라이더
│
├── layout/
│   ├── ScreenContainer.tsx        # SafeAreaView + 배경
│   ├── StarBackground.tsx         # 별 파티클 배경
│   └── KeyboardAwareView.tsx
│
├── character/
│   └── CharacterAvatar.tsx        # 캐릭터 아이콘 (size: sm|md|lg)
│                                   #   → 온보딩/리포트/마이페이지/채팅/마음탐색
│
└── emotion/
    ├── EmotionEmoji.tsx           # 감정 이모지 단일
    └── EmotionEmojiRow.tsx        # 5개 이모지 행
                                    #   → 온보딩/홈/체크인
```

---

## 페이지 로컬 컴포넌트 (공통 제외 목록)

> 단일 화면 전용이거나, 화면마다 커스터마이징이 더 중요한 컴포넌트는 각 페이지 폴더에 유지.

`TabBar`, `PageHeader`, `SelectableChip`, `SelectableCard`, `TagBadge`, `StepProgressBar`,  
`EmptyState`, `ListItemRow`, `ToggleSwitch`, `ButtonGroup`, `AvatarInfoRow`,  
`MessageBubble`, `TypingIndicator`, `OnlineIndicator`, `FadeInText`, `CharacterCard`

---

## 관련 문서

- [폴더 구조](./06-folder-structure.md)
- [다음 설계 단계](./08-next-steps.md) — Figma 컴포넌트 라이브러리 구축
