# 푸시 알림 설계 (FCM)

> [← 인덱스로 돌아가기](./index.md)

---

## 개요

| 항목 | 내용 |
|---|---|
| 플랫폼 | Firebase Cloud Messaging (FCM) |
| Expo 연동 | `expo-notifications` |
| 발신 주체 | 서버 → FCM → 디바이스 |
| 알림 방향 | **서버/AI 발신 전용** (사용자 → 서버 요청 알림 없음) |

---

## 알림 타입 정의

### 1. `CHARACTER_MESSAGE` — 캐릭터 선제 메세지

AI 캐릭터가 먼저 사용자에게 말을 거는 알림. 일정 시간 앱을 열지 않거나 서버가 개입 타이밍을 판단할 때 발송.

| 필드 | 예시 값 |
|---|---|
| `type` | `"CHARACTER_MESSAGE"` |
| `title` | `"솔이가 보고 싶대요 👋"` |
| `body` | `"오늘 하루 어땠어요? 잠깐 얘기해요."` |
| `data.characterId` | `"sol"` |
| `data.previewMessage` | `"오늘 하루 어땠어요?"` |
| 탭 시 이동 | `/chat` (해당 캐릭터 채팅방) |

---

### 2. `REPORT_READY` — 리포트 확인 제안

주간/월간 리포트가 생성됐을 때 확인을 유도하는 알림.

| 필드 | 예시 값 |
|---|---|
| `type` | `"REPORT_READY"` |
| `title` | `"이번 주 감정 리포트가 도착했어요 📊"` |
| `body` | `"7일간의 감정 흐름을 확인해보세요."` |
| `data.reportPeriod` | `"weekly"` \| `"monthly"` |
| `data.reportDate` | `"2026-05-07"` |
| 탭 시 이동 | `/report?period=weekly` |

---

### 3. `TODO_SUGGESTION` — 투두 제안

캐릭터가 오늘의 행동을 제안하는 알림. 체크인 미완료, 리포트 트리거 패턴 기반으로 발송.

| 필드 | 예시 값 |
|---|---|
| `type` | `"TODO_SUGGESTION"` |
| `title` | `"오늘의 제안이 왔어요 ✨"` |
| `body` | `"산책 10분, 해볼 수 있을 것 같아요?"` |
| `data.todoText` | `"산책 10분"` |
| `data.characterId` | `"sol"` |
| 탭 시 이동 | `/home` (홈 투두 섹션 스크롤) |

---

## 구현 흐름

### FCM 토큰 등록

```
앱 최초 실행 또는 로그인 완료
  └→ expo-notifications.getExpoPushTokenAsync() 또는
     getDevicePushTokenAsync() (raw FCM token)
       └→ POST /api/notifications/token { token, platform }
            └→ 서버가 FCM 토큰 저장 및 사용자와 연결
```

토큰 갱신 이벤트(`addPushTokenListener`)를 구독해 변경 시 서버에 재전송.

---

### 수신 처리 (앱 상태별)

| 앱 상태 | 처리 방식 |
|---|---|
| **Foreground** | `addNotificationReceivedListener` → In-app 토스트 or 배너 표시 |
| **Background** | OS가 알림 트레이에 표시, 탭 시 앱 실행 |
| **Killed** | OS가 알림 트레이에 표시, 탭 시 앱 콜드스타트 |

Foreground 수신 시 `CHARACTER_MESSAGE`는 채팅 화면이 열려 있으면 알림 억제, 그 외 화면이면 캐릭터 말풍선 배너로 표시.

---

### 딥링크 처리

`addNotificationResponseReceivedListener`에서 `notification.request.content.data.type`을 보고 라우팅.

```ts
// src/notifications/handleNotificationTap.ts
export function handleNotificationTap(response: NotificationResponse) {
  const { type, reportPeriod } = response.notification.request.content.data;

  switch (type) {
    case 'CHARACTER_MESSAGE':
      router.push('/chat');
      break;
    case 'REPORT_READY':
      router.push(`/report?period=${reportPeriod ?? 'weekly'}`);
      break;
    case 'TODO_SUGGESTION':
      router.push('/home');
      break;
  }
}
```

콜드스타트 시에는 `getLastNotificationResponseAsync()`로 초기 응답을 처리.

---

## 알림 권한 요청 타이밍

| 시점 | 위치 | 이유 |
|---|---|---|
| 온보딩 4단계 (캐릭터 선택) 완료 직후 | `app/(auth)/onboarding/step4-character.tsx` | 캐릭터와의 연결감이 형성된 직후가 수락률 최적 |
| 거부 시 | 마이페이지 → 설정에서 재유도 배너 표시 | |

---

## 폴더 구조 추가

```
src/
└── notifications/
    ├── fcm.ts                  # 토큰 등록/갱신 로직
    ├── handleNotificationTap.ts # 딥링크 라우팅
    └── notificationTypes.ts    # 알림 타입 상수 및 payload 타입 정의
```

---

## 관련 문서

- [기술 스택](./02-tech-stack.md) — `expo-notifications` 라이브러리
- [폴더 구조](./06-folder-structure.md) — `src/notifications/` 위치
- [페이지 설계 — 홈](./04-pages-home.md) — TODO 섹션
- [페이지 설계 — 채팅](./04-pages-chat.md) — 캐릭터 채팅 진입
- [페이지 설계 — 리포트](./04-pages-report.md) — 리포트 화면 진입
