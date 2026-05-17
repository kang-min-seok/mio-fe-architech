# 03. Character (캐릭터)

> 버전: v1.1.1 | 정책: MIO-Character-001~007
> 

## 캐릭터 목록

| character_id | 이름 | 동물 | MVP 접근 |
| --- | --- | --- | --- |
| `mio` | 미오 | 펭귄 | ✅ 전체 개방 |
| `bau` | 바우 | 강아지 | ✅ 전체 개방 |
| `rumi` | 루미 | 부엉이 | ✅ 전체 개방 |
| `momo` | 모모 | 곰 | ✅ 전체 개방 |
| `chichi` | 치치 | 고양이 | ✅ 전체 개방 |

## Base Path

```
GET  /v1/characters
POST /v1/user/character
GET  /v1/user/character
```

## 주요 정책

- **캐릭터 변경 시 맥락 초기화** (MIO-Character-007): 새 캐릭터와의 대화는 새로 시작. 사용자에게 사전 안내 필수.
- **프리미엄 접근**: MVP에서는 모든 캐릭터 제한 없이 선택 가능. post-MVP에서 프리미엄 캐릭터 구분 예정.
- **선제 메시지**: 매일 09:00 / 12:00 / 22:00 가 성 타임 발송 (MIO-Character-005)

## GET /v1/characters

Response 200:

```json
{
  "data": {
    "characters": [
      {
        "character_id": "mio",
        "name": "미오",
        "animal": "penguin",
        "description": "따뜻하고 섬세하게 감정을 들어줘요.",
        "personality_tags": ["warm", "empathetic", "gentㅑle"],
        "thumbnail_url": "https://cdn.mio.app/characters/mio/thumbnail.png"
      }
      // bau, rumi, momo, chichi ...
    ]
  }
}
```

## POST /v1/user/character

Request Body:

```json
{ "character_id": "mio" }
```

Response 200:

```json
{
  "data": {
    "character_id": "mio",
    "changed": true,
    "greeting_message": "안녕! 나 미오야 🐧 오늘 어떤 하루를 보냈어?"
  }
}
```

`changed: false`이면 이전과 동일 캐릭터.

Errors: `404 NOT_FOUND`

## GET /v1/user/character

현재 선택된 캐릭터 조회. 온보딩 완료 후 미선택이면 `character_id: null`

## 에러코드

| 코드 | HTTP | 설명 |
| --- | --- | --- |
| `NOT_FOUND` | 404 | 유효하지 않은 character_id |
| `ONBOARDING_REQUIRED` | 403 | 온보딩 미완료 |