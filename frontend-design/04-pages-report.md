# 페이지 설계 — 리포트

> [← 인덱스로 돌아가기](./index.md)

**네비게이션:** `/(main)/report/index`

![리포트 화면](../screen_report.png)

---

## 구성 섹션

| 섹션 | 기능 |
|---|---|
| 기간 탭 | 주간 / 월간 / 3개월 / 전체 |
| 감정 변화 추이 | 꺾은선 그래프 (x: 요일/날짜, y: 감정 강도) |
| 주요 트리거 TOP 3 | 바 차트 형태, 퍼센트 + 트리거명 |
| 캐릭터 주간 이야기 | 캐릭터 아이콘 + 날짜 + AI 생성 주간 요약 텍스트 |

---

## 주요 컴포넌트

```
ReportScreen
  ├── ReportHeader
  ├── PeriodTabBar             # 주간/월간/3개월/전체
  ├── EmotionTrendChart        # victory-native LineChart
  │   └── EmptyChartState      # 데이터 없을 때
  ├── TriggerTopList
  │   └── TriggerItem          # 순위 번호 + 트리거명 + 퍼센트 바
  └── CharacterWeeklyStory
       ├── CharacterAvatar
       ├── StoryDateRange
       └── StoryText
```

---

## 상태 관리 (Zustand)

Zustand 의존 없음. 기간 탭 선택(`period`)은 `useState` 로컬 상태 처리, 데이터는 TanStack Query.

---

## 데이터 의존성 (TanStack Query)

```typescript
useQuery(['report', period, 'trend'])        // 기간별 감정 강도 추이 (꺾은선 그래프 데이터)
useQuery(['report', period, 'triggers'])     // 기간별 트리거 TOP 3 + 퍼센트
useQuery(['report', 'story', 'weekly'])      // 캐릭터 주간 이야기 (AI 생성 텍스트)
```

---

## 관련 문서

- [기술 스택](./02-tech-stack.md) — 차트 라이브러리 (`victory-native`)
- [상태 관리 설계](./07-state-management.md) — TanStack Query 캐시 전략
