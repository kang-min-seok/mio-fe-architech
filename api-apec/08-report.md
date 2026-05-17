# 08. Report (리포트)

> 버전: v1.1.1 | 정책: MIO-Report-001~008
> 

## 도메인 설명

감정 기록과 대화 데이터를 기반로 생성되는 주간·월간 분석 리포트. 리포트는 **사전 생성** 방식 (API 호출 시 즉시 생성 안 함).

## Base Path

```
GET /v1/reports/weekly
GET /v1/reports/weekly/{week_start}
GET /v1/reports/monthly         (프리미엄 전용)
GET /v1/reports/monthly/{year_month}
```

## 리포트 생성 주기

| 종류 | 생성 시점 | 기준 데이터 |
| --- | --- | --- |
| 주간 리포트 | 매주 **월요일 03:00** | 전주 월~일 |
| 월간 리포트 | 매월 **1일 04:00** | 전월 1일~말일 |

체크인 횟수가 3회 미만이면 `is_partial: true`로 일부 섹션 없음 (MIO-Report-004)

## GET /v1/reports/weekly

최신 주간 리포트 조회

Response 200:

```json
{
  "data": {
    "report_id": "550e8400-...",
    "week_start": "2026-05-04",
    "week_end": "2026-05-10",
    "is_partial": false,
    "checkin_count": 14,
    "avg_emotion_score": 2.8,
    "emotion_scores": {
      "2026-05-04": 3.0,
      "2026-05-05": 2.5
    },
    "distortion_distribution": {
      "catastrophizing": 5,
      "all_or_nothing": 3
    },
    "behavior_summary": {
      "total_todos": 9,
      "completed_todos": 6,
      "completion_rate": 0.67,
      "most_effective_action": "3분 호흡 명상"
    },
    "narrative": "이번 주 효찬님은 화요일과 수요일에 힘든 순간이 있었지만...",
    "coaching_direction": "파국화 패턴이 자주 보였어요.",
    "generated_at": "2026-05-11T03:00:00Z"
  }
}
```

| 필드 | 1차 개발 | 설명 |
| --- | --- | --- |
| `narrative` | null | 2차: GPT-4o 생성 캐릭터 내러티브 |
| `coaching_direction` | null | 2차: GPT-4o 생성 코칭 방향 |

## GET /v1/reports/monthly (프리미엄 전용)

Errors: `402 PAYMENT_REQUIRED`, `404 NOT_FOUND`

## 감정 점수 해석 가이드

| 구간 | 해석 | UI 색상 권장 |
| --- | --- | --- |
| 4.0 ~ 5.0 | 긍정적 | 초록 계열 |
| 3.0 ~ 3.9 | 보통 | 노란/회색 |
| 2.0 ~ 2.9 | 다소 부정 | 주황 |
| 1.0 ~ 1.9 | 부정 | 빨강/진한 주황 |