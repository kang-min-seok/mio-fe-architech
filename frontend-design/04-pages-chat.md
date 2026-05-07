# 페이지 설계 — 채팅

> [← 인덱스로 돌아가기](./index.md)

**네비게이션:** `/(main)/chat/index` + `/(main)/chat/restructure`

![채팅 메인](../screen_chat_main.png)
![생각 재구성](../screen_chat_restructure.png)

---

## 채팅 메인 기능

| 기능 | 설명 |
|---|---|
| AI 캐릭터 대화 | 온보딩에서 선택한 캐릭터와 실시간 대화 (스트리밍) |
| 인지 왜곡 감지 배너 | AI가 감지 시 "지금 생각 재구성을 해볼까요?" 인라인 카드 노출 |
| 감정 강도 감지 | 대화 중 "지금 느끼는 감정 강도" 슬라이더 인라인 표시 |
| 생각 재구성 제안 수락 | 미루기 / 건너뛰기 버튼 → 수락 시 restructure 화면 진입 |

---

## 채팅 메시지 타입

```typescript
type MessageType =
  | 'text'                    // 일반 텍스트 버블
  | 'ai-text'                 // AI 응답 버블
  | 'restructure-prompt'      // 생각 재구성 제안 카드
  | 'emotion-intensity'       // 감정 강도 슬라이더 카드
  | 'typing'                  // 타이핑 인디케이터
```

---

## 생각 재구성 화면 구성

```
RestructureScreen
  ├── RestructureHeader        # ← 뒤로가기
  ├── OriginalThoughtCard      # "내가 가진 생각" (사용자 말 인용)
  ├── CharacterQuestionCard    # "루미의 질문"
  ├── AlternativeThoughtCard   # "대안 생각" (AI 생성, 그린 배경)
  ├── EmotionSlider            # "다시 느끼는 감정은?" 0~100
  └── SaveButton               # 변화 저장하기 / 저장 완료
```

---

## 주요 컴포넌트

```
ChatScreen
  ├── ChatHeader               # 캐릭터명 + 온라인 dot + 클라우드 아이콘
  ├── ChatMessageList          # FlatList 역방향
  │   ├── TextBubble           # 사용자 / AI 버블
  │   ├── RestructurePromptCard
  │   └── EmotionIntensityCard # 슬라이더 인라인 카드
  ├── TypingIndicator          # 말풍선 애니메이션 3점
  └── ChatInput
       ├── TextInput
       └── SendButton

RestructureScreen
  ├── OriginalThoughtCard
  ├── CharacterQuestionCard
  ├── AlternativeThoughtCard
  ├── IntensitySlider
  └── SaveButton
```

---

## 상태 관리 (Zustand)

#### `chat/index.tsx`

| Store | 필드 / 액션 | R/W | 용도 |
|---|---|:---:|---|
| chatStore | `messages` | R | `ChatMessageList` 렌더링 |
| chatStore | `isTyping` | R | `TypingIndicator` 표시 여부 |
| chatStore | `streamingMessageId` | R | 스트리밍 진행 중인 버블 식별 (커서 애니메이션) |
| chatStore | `restructurePrompt` | R | `!= null` 이면 `RestructurePromptCard` 인라인 카드 노출 |
| chatStore | `addMessage()` | W | 사용자 메시지 전송 시 즉시 낙관적 추가 |
| chatStore | `appendStreamChunk()` | W | SSE 청크 수신마다 해당 AI 메시지 버블에 append |
| chatStore | `setTyping()` | W | 메시지 전송 후 `true`, AI 첫 청크 수신 시 `false` |
| chatStore | `setRestructurePrompt()` | W | AI 응답에 인지 왜곡 감지 플래그가 포함될 때 |

#### `chat/restructure.tsx`

| Store | 필드 / 액션 | R/W | 용도 |
|---|---|:---:|---|
| chatStore | `restructurePrompt.originalThought` | R | `OriginalThoughtCard` 사용자 발화 인용 |
| chatStore | `restructurePrompt.aiQuestion` | R | `CharacterQuestionCard` AI CBT 질문 |
| chatStore | `restructurePrompt.alternativeThought` | R | `AlternativeThoughtCard` AI 대안 생각 |
| chatStore | `setRestructurePrompt(null)` | W | 저장 완료(`useMutation` 성공) 후 카드 닫기 |

---

## 데이터 의존성 (TanStack Query)

```typescript
useQuery(['chat', 'character'])              // 현재 선택 캐릭터 정보 (이름, 아바타)
useInfiniteQuery(['chat', 'messages'])       // 대화 히스토리 (역방향 페이지네이션)
useMutation(['chat', 'send'])                // 메시지 전송 (SSE 스트리밍 응답)
useMutation(['chat', 'restructure', 'save']) // 생각 재구성 결과 저장
```

---

## 관련 문서

- [상태 관리 설계](./07-state-management.md) — chatStore 상세 인터페이스
- [다음 설계 단계](./08-next-steps.md) — 스트리밍 방식 확정 (SSE vs WebSocket)
