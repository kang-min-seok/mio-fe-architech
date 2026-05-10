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
| ErrorState | | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| LoadingSkeleton | | ✓ | ✓ | ✓ | ✓ | ✓ | |

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
│   ├── Slider/
│   │   └── IntensitySlider.tsx    # 0~100, 중앙 숫자 표시
│   │                               #   → 체크인 강도 입력, 채팅 EmotionIntensityCard,
│   │                               #     생각 재구성 슬라이더
│   │
│   ├── Feedback/
│   │   ├── ErrorState.tsx         # 에러 상태 — 아이콘 + 메시지 + 재시도 버튼
│   │   │                           #   → TanStack Query isError 시 노출
│   │   └── LoadingSkeleton.tsx    # 스켈레톤 UI — 직사각형 shimmer 블록 조합
│   │                               #   → TanStack Query isLoading 시 노출
│   │
│   └── Toast/
│       └── Toast.tsx              # 하단 일시적 알림 (Mutation 에러·성공 피드백)
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
    └── EmotionEmojiRow.tsx        # 7개 이모지 행 (백엔드 7종 타입 기준)
                                    #   → 온보딩/홈/체크인
```

---

## 공통 컴포넌트 Props 인터페이스

### Button

```typescript
// PrimaryButton / SecondaryButton 공통 베이스
interface ButtonProps {
  label: string
  onPress: () => void
  disabled?: boolean          // 기본값 false
  loading?: boolean           // 로딩 스피너 표시 + 비활성
  size?: 'sm' | 'md' | 'lg'  // 기본값 'md'
}

interface IconButtonProps {
  icon: string                // Ionicons 아이콘명
  onPress: () => void
  size?: number               // px, 기본값 24
  color?: string              // 기본값 디자인 토큰 textPrimary
  accessibilityLabel: string  // 필수 — 스크린 리더 지원
}
```

### TextInput / MultilineInput

```typescript
interface TextInputProps {
  value: string
  onChangeText: (text: string) => void
  placeholder?: string
  maxLength?: number
  editable?: boolean
  autoFocus?: boolean
  returnKeyType?: 'done' | 'next' | 'search'
  onSubmitEditing?: () => void
}

interface MultilineInputProps extends Omit<TextInputProps, 'returnKeyType' | 'onSubmitEditing'> {
  maxLength: number           // 필수 (체크인: 200, 온보딩 닉네임: 20)
  showCount?: boolean         // 우측 하단 현재/최대 글자수 표시, 기본값 true
}
```

### GlassCard

```typescript
interface GlassCardProps {
  children: React.ReactNode
  variant?: 'default' | 'elevated'  // elevated — 더 밝은 배경
  padding?: 'none' | 'sm' | 'md' | 'lg'  // 기본값 'md'
  onPress?: () => void               // 터치 가능 카드
}
```

### IntensitySlider

```typescript
interface IntensitySliderProps {
  value: number               // 0~100
  onValueChange: (value: number) => void
  label?: string              // 슬라이더 상단 라벨 텍스트
}
```

### CharacterAvatar

```typescript
interface CharacterAvatarProps {
  characterId: CharacterId
  size: 'sm' | 'md' | 'lg'   // sm:32px, md:56px, lg:96px
  showOnlineDot?: boolean     // 채팅 헤더용
}
```

### ErrorState

```typescript
interface ErrorStateProps {
  message?: string            // 기본값 "오류가 발생했어요"
  onRetry?: () => void        // 재시도 버튼 (없으면 버튼 미노출)
}
```

### LoadingSkeleton

```typescript
interface LoadingSkeletonProps {
  width: number | string      // px 또는 '%'
  height: number
  borderRadius?: number       // 기본값 8
  count?: number              // 블록 반복 수, 기본값 1
}
```

---

## 에러·로딩 상태 처리 가이드

### Query 상태별 렌더링 패턴

```tsx
// 권장 패턴 — 모든 Query 사용 화면에서 일관 적용
const { data, isLoading, isError, refetch } = useQuery(...)

if (isLoading) return <LoadingSkeleton ... />
if (isError)   return <ErrorState onRetry={refetch} />
return <ActualContent data={data} />
```

### ErrorBoundary 적용 위치

| 위치 | 역할 |
|---|---|
| `app/_layout.tsx` (Root) | 앱 전역 크래시 catch — 전체 화면 에러 UI |
| `(main)/_layout.tsx` | 탭 네비게이터 영역 크래시 격리 |
| 각 페이지 컴포넌트 | Query 에러는 `isError` 패턴으로 처리 (ErrorBoundary 아님) |

### Toast 사용 기준

- Mutation 성공: 긍정 피드백 Toast (예: "체크인이 저장됐어요")
- Mutation 실패: 에러 Toast (예: "저장에 실패했어요. 다시 시도해주세요")
- Query 에러: Toast 사용 안 함 — `ErrorState` 컴포넌트 인라인 노출

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
