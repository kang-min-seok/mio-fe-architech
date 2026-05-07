# 페이지 설계 — 마이페이지

> [← 인덱스로 돌아가기](./index.md)

**네비게이션:** `/(main)/my/index`

---

## 구성 항목

| 항목 | 기능 |
|---|---|
| 프로필 | 닉네임, 현재 파트너 캐릭터 |
| 파트너 변경 | 캐릭터 선택 화면 재진입 |
| 알림 설정 | 체크인 리마인더, 채팅 알림 토글 |
| 계정 설정 | 로그아웃, 회원탈퇴 |
| 앱 정보 | 버전, 개인정보처리방침, 이용약관 |

---

## 주요 컴포넌트

```
MyScreen
  ├── ProfileCard              # 닉네임 + 캐릭터 아바타
  ├── SettingSection
  │   ├── PartnerChangeRow     # → partner.tsx 이동
  │   ├── NotificationRow      # 알림 설정 토글
  │   └── AccountSection       # 로그아웃, 탈퇴
  └── AppInfoSection
```

---

## 상태 관리 (Zustand)

#### `my/index.tsx`

| Store | 필드 / 액션 | R/W | 용도 |
|---|---|:---:|---|
| authStore | `nickname` | R | `ProfileCard` 닉네임 표시 |
| authStore | `logout()` | W | 로그아웃 버튼 탭 → `useMutation` 성공 콜백에서 호출 |

#### `my/partner.tsx`

Zustand 직접 의존 없음. 파트너 변경은 `useMutation(['my', 'partner', 'update'])` 처리.

#### `my/settings.tsx`

Zustand 의존 없음. 알림 설정은 TanStack Query.

---

## 데이터 의존성 (TanStack Query)

```typescript
useQuery(['my', 'profile'])                  // 유저 닉네임 + 현재 파트너 캐릭터
useQuery(['my', 'settings'])                 // 알림 설정 (체크인 리마인더, 채팅 알림 토글 상태)
useMutation(['my', 'partner', 'update'])     // 파트너 변경 → profile 캐시 무효화
useMutation(['my', 'account', 'logout'])     // 로그아웃 → 전체 쿼리 캐시 초기화
```

---

## 관련 문서

- [상태 관리 설계](./07-state-management.md) — authStore 상세 인터페이스
