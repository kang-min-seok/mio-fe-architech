# 11. Subscription (구독)

> 버전: v1.1.1 | **MVP 제외 — 이 섹션 전체가 post-MVP 대상이다.**
> 

> 결제/구독 기능은 MVP 출시 이후 별도 개발 단계에서 구현한다. MVP에서는 모든 사용자가 동일한 권한으로 서비스를 이용한다.
> 

## MVP 기간 중 정책

- 모든 사용자 동일 권한 (캐릭터 전체 개방, 세션 횟수 제한 없음)
- `is_premium` 플래그는 DB에 존재하나 항상 `false`

## post-MVP 구현 예정 항목

```jsx
POST /v1/subscriptions/verify       // Apple / Google 영수증 검증
GET  /v1/subscriptions/status       // 구독 상태 조회
POST /v1/subscriptions/webhooks/apple   // Apple S2S 알림
POST /v1/subscriptions/webhooks/google  // Google RTDN
```

## post-MVP 프리미엄 기능 계획

| 기능 | 무료 | 프리미엄 |
| --- | --- | --- |
| 월간 리포트 | ❌ | ✅ |
| 프리미엄 캐릭터 (momo, chichi) | ❌ | ✅ |
| 세션 횟수 | 제한 예정 | 무제한 |