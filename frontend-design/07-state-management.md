# 상태 관리 설계

> [← 인덱스로 돌아가기](./index.md)

---

## Zustand Store 책임 분리

| Store | 관리 상태 | 비고 |
|---|---|---|
| `authStore` | 유저 정보, 액세스 토큰, 인증 상태 | SecureStore 연동 |
| `onboardingStore` | 온보딩 스텝별 선택값 | 완료 후 서버 저장 + 초기화 |
| `chatStore` | 현재 세션 메시지 목록, 타이핑 상태, 재구성 제안 상태 | 실시간 업데이트 |
| `checkinStore` | 오늘 체크인 임시 입력값 | 완료 후 초기화 |
| `mindExploreStore` | 탐색 세션, 스테이지별 선택, 결과 | 탐색 종료 후 초기화 |

---

## Zustand Store 상세 인터페이스

### authStore

```typescript
interface AuthState {
  // 상태
  userId: string | null
  nickname: string | null
  accessToken: string | null
  isAuthenticated: boolean
  isOnboarded: boolean                       // 온보딩 완료 여부 — AuthGuard 분기 기준 (03-navigation.md §AuthGuard)
  currentPartnerId: CharacterId | null       // 현재 선택된 AI 파트너 ID — 채팅·마이페이지 캐시 무효화 기준
  // 액션
  setAuth: (userId: string, nickname: string, accessToken: string) => void
  setAccessToken: (token: string) => void   // Axios 인터셉터에서 토큰 갱신 시 호출
  setOnboarded: () => void                   // 온보딩 useMutation 성공 후 호출
  setCurrentPartner: (characterId: CharacterId) => void  // 파트너 변경 확정(useMutation 성공) 후 호출
  logout: () => void                         // 상태 초기화 + SecureStore 삭제
}
```

### 공유 타입

```typescript
// 캐릭터 ID — 백엔드 character 도메인과 동일한 5종 값 사용 (mio-system-architecture.md 기준)
type CharacterId = 'mio' | 'bau' | 'rumi' | 'momo' | 'chichi'
```

### onboardingStore

```typescript
interface OnboardingState {
  // 상태
  emotion: string | null
  concerns: string[]                          // max 3
  conversationStyle: 'empathy' | 'analysis' | 'solution' | 'balanced' | null
  selectedCharacter: CharacterId | null
  currentStep: 1 | 2 | 3 | 4
  // 액션
  setEmotion: (emotion: string) => void
  toggleConcern: (concern: string) => void    // 추가/제거 토글, max 3 초과 시 무시
  setConversationStyle: (style: ConversationStyle) => void
  setCharacter: (characterId: CharacterId) => void
  nextStep: () => void
  prevStep: () => void
  reset: () => void                           // 온보딩 완료 후 서버 저장 뒤 호출
}
```

### chatStore

```typescript
interface ChatMessage {
  id: string
  type: MessageType                           // 'text' | 'ai-text' | 'restructure-prompt' | 'emotion-intensity' | 'typing'
  content: string
  sender: 'user' | 'ai'
  createdAt: Date
}

interface RestructurePrompt {
  triggerMessageId: string                    // 재구성 제안을 트리거한 메시지 ID
  originalThought: string                     // 사용자 발화 인용
  aiQuestion: string                          // AI의 CBT 질문
  alternativeThought: string                  // AI가 생성한 대안 생각
}

interface ChatState {
  // 상태
  messages: ChatMessage[]                     // 현재 세션 메시지 (히스토리 로드 후 append)
  isTyping: boolean                           // AI 응답 대기 중
  isConnected: boolean                        // SSE 연결 활성 여부 (useChatSSE 훅이 관리)
  streamingMessageId: string | null           // 현재 스트리밍 수신 중인 AI 메시지 ID
  restructurePrompt: RestructurePrompt | null // 인라인 재구성 제안 카드 데이터
  // 액션
  addMessage: (message: ChatMessage) => void
  appendStreamChunk: (messageId: string, chunk: string) => void  // SSE 청크 수신 시 해당 메시지에 append
  addSingleResponse: (message: ChatMessage) => void              // BUFFER_AND_JUDGE·SECURITY_REFUSAL·CRISIS_FLOW 처리
  setTyping: (isTyping: boolean) => void
  setConnected: (connected: boolean) => void
  setRestructurePrompt: (prompt: RestructurePrompt | null) => void
  reset: () => void
}
```

### checkinStore

```typescript
// 백엔드 EmotionalStateMemory 타입과 동일한 값 사용 — 확정 (10-backend-api-guide.md §5)
// UI 이모지·한글 라벨은 constants/emotions.ts 에서 관리
type EmotionType = 'anxiety' | 'sadness' | 'anger' | 'shame' | 'stress' | 'neutral' | 'positive'
type CheckinStep = 'emotion' | 'intensity' | 'diary'

interface CheckinState {
  // 상태
  selectedEmotion: EmotionType | null
  intensity: number                           // 0~100, 기본값 50
  diaryText: string
  currentStep: CheckinStep
  // 액션
  setEmotion: (emotion: EmotionType) => void
  setIntensity: (value: number) => void
  setDiaryText: (text: string) => void
  nextStep: () => void
  prevStep: () => void
  reset: () => void                           // 체크인 저장 완료 후 호출
}
```

### partnerStore

```typescript
interface PartnerState {
  // 상태
  selectedPartnerId: CharacterId | null   // 변경 확정 전 임시 선택 상태 (PartnerScreen 로컬)
  // 액션
  setSelectedPartner: (characterId: CharacterId) => void
  reset: () => void                        // PartnerScreen 이탈(취소·성공) 시 임시 선택 초기화
}
```

### mindExploreStore

```typescript
interface MindExploreResult {
  recommendedCharacterId: CharacterId
  recommendedCharacterReason: string
  personalityAnalysis: string
}

interface MindExploreState {
  // 상태
  sessionId: string | null
  nickname: string
  currentStageId: string | null
  choices: Record<string, string>             // stageId → choiceId
  result: MindExploreResult | null
  // 액션
  initSession: (sessionId: string) => void
  setNickname: (nickname: string) => void
  setChoice: (stageId: string, choiceId: string) => void
  setCurrentStage: (stageId: string) => void
  setResult: (result: MindExploreResult) => void
  reset: () => void                           // 탐색 종료(닫기) 후 호출
}
```

---

## 페이지별 Zustand 의존성 맵

> **범례:** R = 읽기(구독), W = 액션 호출(쓰기)

| 페이지 | Store | 요약 |
|---|---|---|
| `app/_layout.tsx` | authStore | `isAuthenticated` R → (auth)/(main) 분기 |
| `(auth)/login.tsx` | authStore | `isAuthenticated` R, `setAuth()` W |
| `onboarding/step1~4` | onboardingStore | 각 스텝 선택값 R/W, `reset()` W |
| `(main)/home` | authStore | `nickname` R |
| `(main)/checkin` | checkinStore | 모든 입력 상태 R/W, `reset()` W |
| `(main)/checkin/history` | — | TanStack Query만 사용 |
| `(main)/chat` | chatStore | messages/isTyping/streaming/restructure R/W |
| `(main)/chat/restructure` | chatStore | restructurePrompt R, `setRestructurePrompt(null)` W |
| `(main)/report` | — | useState(period) + TanStack Query만 사용 |
| `(main)/my` | authStore | `nickname` R, `logout()` W |
| `(main)/my/partner` | partnerStore, authStore | `selectedPartnerId` R/W, `reset()` W / 변경 확정 후 `setCurrentPartner()` W |
| `(main)/my/settings` | — | TanStack Query만 사용 |
| `mind-explore/index` | mindExploreStore | `reset()` W, `initSession()` W |
| `mind-explore/[stageId]` | mindExploreStore | sessionId/nickname/choices R/W |
| `mind-explore/result` | mindExploreStore | sessionId/result R, `setResult()` W, `reset()` W |

---

## TanStack Query 캐시 전략

### QueryClient 전역 기본값

```typescript
// src/queries/queryClient.ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 1,                   // 실패 시 1회 재시도
      staleTime: 1000 * 60 * 5,  // 전역 기본 5분
      gcTime: 1000 * 60 * 10,    // 비활성 캐시 10분 후 GC
      refetchOnWindowFocus: false, // 모바일 앱 포커스 복귀 시 자동 재요청 비활성
    },
    mutations: {
      retry: 0,                   // 뮤테이션은 재시도 없음 (사용자가 명시적으로 재시도)
    },
  },
})
```

### 도메인별 staleTime 오버라이드

```typescript
const queryConfig = {
  'home-today':        { staleTime: 1000 * 60 * 5  },  // 5분
  'report-weekly':     { staleTime: 1000 * 60 * 10 },  // 10분
  'chat-history':      { staleTime: 0               },  // 항상 최신 (대화 일관성)
  'characters':        { staleTime: Infinity         },  // 정적 데이터
  'mind-explore-stages': { staleTime: Infinity       },  // 정적 시나리오
}
```

### 에러 처리 전략

| 에러 유형 | 처리 방식 |
|---|---|
| `401 Unauthorized` | Axios 인터셉터 → Refresh Token 재발급 시도 → 실패 시 `logout()` + 로그인 화면 이동 |
| `403 Forbidden` | 인터셉터에서 무시 (앱 내 권한 분리 없음 — USER 단일 롤) |
| `5xx Server Error` | TanStack Query `retry: 1` — 1회 재시도 후 `ErrorState` 컴포넌트 노출 |
| `Network Error` | TanStack Query `retry: 1` — 재시도 후 오프라인 안내 |
| Mutation 실패 | 재시도 없음. 각 페이지에서 `onError` 콜백으로 Toast/Alert 처리 |

---

## 관련 문서

- [페이지 설계 — 온보딩](./04-pages-onboarding.md)
- [페이지 설계 — 체크인](./04-pages-checkin.md)
- [페이지 설계 — 채팅](./04-pages-chat.md)
- [페이지 설계 — 마음 탐색](./04-pages-mind-explore.md)
- [네비게이션 구조](./03-navigation.md) — AuthGuard 분기 (isOnboarded 활용)
- [디자인 토큰](./11-design-tokens.md) — 감정 색상 매핑
- [폴더 구조](./06-folder-structure.md) — `src/stores/`, `src/queries/`
