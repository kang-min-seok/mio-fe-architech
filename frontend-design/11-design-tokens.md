# 디자인 토큰

> [← 인덱스로 돌아가기](./index.md)
>
> 작성일: 2026-05-10  
> 테마: 다크 기본 + 라이트 지원  
> 구현 파일: `src/constants/colors.ts`, `tailwind.config.js`

---

## 색상 팔레트

### 베이스 팔레트 (다크 테마)

| 토큰 | 값 | 용도 |
|---|---|---|
| `background` | `#0A0914` | 앱 배경 (딥 퍼플-블랙) |
| `surface` | `#13112A` | 카드·시트 기본 배경 |
| `surfaceElevated` | `#1C1A3A` | 팝업·모달·툴팁 배경 |
| `primary` | `#7C5CFC` | 주요 CTA 버튼, 선택 강조 |
| `primaryLight` | `#9B7FFE` | 호버·포커스 상태 |
| `glass` | `rgba(255,255,255,0.08)` | GlassCard 배경 |
| `glassBorder` | `rgba(255,255,255,0.12)` | GlassCard 테두리 |
| `textPrimary` | `#FFFFFF` | 본문·제목 |
| `textSecondary` | `#B0ABCE` | 보조 텍스트 |
| `textMuted` | `#6B6491` | 비활성·플레이스홀더 |
| `border` | `#2D2B4E` | 구분선·인풋 테두리 |
| `success` | `#4ADE80` | 완료·저장 성공 |
| `warning` | `#FBBF24` | 주의 |
| `error` | `#F87171` | 에러·위험 액션 (로그아웃·탈퇴 버튼 등) |

### 베이스 팔레트 (라이트 테마)

| 토큰 | 값 | 용도 |
|---|---|---|
| `background` | `#F5F3FF` | 앱 배경 (연한 라벤더) |
| `surface` | `#FFFFFF` | 카드·시트 기본 배경 |
| `surfaceElevated` | `#EDE9FE` | 팝업·모달 배경 |
| `primary` | `#7C5CFC` | 동일 (브랜드 컬러 불변) |
| `primaryLight` | `#9B7FFE` | 동일 |
| `glass` | `rgba(0,0,0,0.04)` | GlassCard 배경 |
| `glassBorder` | `rgba(0,0,0,0.08)` | GlassCard 테두리 |
| `textPrimary` | `#1C1A3A` | 본문·제목 |
| `textSecondary` | `#4B4671` | 보조 텍스트 |
| `textMuted` | `#9B96C1` | 비활성·플레이스홀더 |
| `border` | `#DDD8F8` | 구분선·인풋 테두리 |
| `success` | `#16A34A` | 완료·저장 성공 |
| `warning` | `#D97706` | 주의 |
| `error` | `#DC2626` | 에러·위험 액션 |

---

## 감정 색상 토큰

체크인 이모지 선택 강조, 리포트 차트 데이터 포인트 색상에 사용한다.  
다크/라이트 테마 모두 동일한 감정 색상을 사용한다.

| 토큰 | EmotionType | 값 |
|---|---|---|
| `emotion.anxiety` | `anxiety` | `#F59E0B` |
| `emotion.sadness` | `sadness` | `#60A5FA` |
| `emotion.anger` | `anger` | `#F87171` |
| `emotion.shame` | `shame` | `#C084FC` |
| `emotion.stress` | `stress` | `#FB923C` |
| `emotion.neutral` | `neutral` | `#94A3B8` |
| `emotion.positive` | `positive` | `#34D399` |

---

## 타이포그래피

| 토큰 | 크기 | 굵기 | 줄간격 | 용도 |
|---|---|---|---|---|
| `heading1` | 28sp | 700 | 1.2 | 온보딩 제목, 결과 화면 헤드라인 |
| `heading2` | 22sp | 700 | 1.3 | 페이지 제목 |
| `heading3` | 18sp | 600 | 1.3 | 섹션 제목, 카드 헤딩 |
| `body1` | 16sp | 400 | 1.5 | 본문 텍스트 |
| `body2` | 14sp | 400 | 1.5 | 보조 텍스트, 라벨 |
| `caption` | 12sp | 400 | 1.4 | 날짜, 타임스탬프, 힌트 |
| `button` | 16sp | 600 | 1.0 | 버튼 레이블 |

폰트 패밀리: 시스템 기본 (`System` — iOS San Francisco, Android Roboto)  
커스텀 폰트 적용 시 `assets/fonts/`에 추가 후 `expo-font`로 로드.

---

## 간격(Spacing) 시스템

4px 베이스 그리드 기준.

| 토큰 | 값 | 용도 |
|---|---|---|
| `space.1` | 4px | 아이콘·라벨 인접 간격 |
| `space.2` | 8px | 컴포넌트 내부 소간격 |
| `space.3` | 12px | 버튼 수직 패딩 |
| `space.4` | 16px | 화면 수평 패딩(기본), 카드 내부 패딩 |
| `space.5` | 20px | 섹션 상단 패딩 |
| `space.6` | 24px | 카드 간 간격 |
| `space.8` | 32px | 섹션 간 간격 |
| `space.10` | 40px | 페이지 상단 여백 |

---

## 모서리 반경(Border Radius)

| 토큰 | 값 | 용도 |
|---|---|---|
| `radius.sm` | 8px | 버튼, 소형 카드 |
| `radius.md` | 12px | 일반 카드, 인풋 |
| `radius.lg` | 16px | GlassCard, 모달 시트 |
| `radius.xl` | 24px | 바텀 시트, 결과 카드 |
| `radius.full` | 9999px | 태그·배지·아바타 |

---

## NativeWind tailwind.config.js 설정

```javascript
// tailwind.config.js
const colors = require('./src/constants/colors')

module.exports = {
  content: ['./app/**/*.{tsx,ts}', './src/**/*.{tsx,ts}'],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {
      colors: {
        background: colors.background,
        surface: colors.surface,
        'surface-elevated': colors.surfaceElevated,
        primary: colors.primary,
        'primary-light': colors.primaryLight,
        glass: colors.glass,
        'text-primary': colors.textPrimary,
        'text-secondary': colors.textSecondary,
        'text-muted': colors.textMuted,
        border: colors.border,
        success: colors.success,
        warning: colors.warning,
        error: colors.error,
        emotion: {
          anxiety: '#F59E0B',
          sadness: '#60A5FA',
          anger: '#F87171',
          shame: '#C084FC',
          stress: '#FB923C',
          neutral: '#94A3B8',
          positive: '#34D399',
        },
      },
      spacing: {
        1: '4px',
        2: '8px',
        3: '12px',
        4: '16px',
        5: '20px',
        6: '24px',
        8: '32px',
        10: '40px',
      },
      borderRadius: {
        sm: '8px',
        md: '12px',
        lg: '16px',
        xl: '24px',
      },
    },
  },
}
```

---

## src/constants/colors.ts 구조

```typescript
import { useColorScheme } from 'react-native'

const dark = {
  background: '#0A0914',
  surface: '#13112A',
  surfaceElevated: '#1C1A3A',
  primary: '#7C5CFC',
  primaryLight: '#9B7FFE',
  glass: 'rgba(255,255,255,0.08)',
  glassBorder: 'rgba(255,255,255,0.12)',
  textPrimary: '#FFFFFF',
  textSecondary: '#B0ABCE',
  textMuted: '#6B6491',
  border: '#2D2B4E',
  success: '#4ADE80',
  warning: '#FBBF24',
  error: '#F87171',
}

const light = {
  background: '#F5F3FF',
  surface: '#FFFFFF',
  surfaceElevated: '#EDE9FE',
  primary: '#7C5CFC',
  primaryLight: '#9B7FFE',
  glass: 'rgba(0,0,0,0.04)',
  glassBorder: 'rgba(0,0,0,0.08)',
  textPrimary: '#1C1A3A',
  textSecondary: '#4B4671',
  textMuted: '#9B96C1',
  border: '#DDD8F8',
  success: '#16A34A',
  warning: '#D97706',
  error: '#DC2626',
}

export const emotionColors = {
  anxiety: '#F59E0B',
  sadness: '#60A5FA',
  anger: '#F87171',
  shame: '#C084FC',
  stress: '#FB923C',
  neutral: '#94A3B8',
  positive: '#34D399',
}

// 훅으로 현재 테마 색상 반환
export function useColors() {
  const scheme = useColorScheme()
  return scheme === 'dark' ? dark : light
}

// tailwind.config.js 참조용 정적 내보내기 (다크 기본값)
module.exports = dark
```

---

## 관련 문서

- [기술 스택](./02-tech-stack.md) — NativeWind, 테마 방향
- [공통 컴포넌트](./05-components.md) — GlassCard, ErrorState, Button Props
- [페이지 설계 — 체크인](./04-pages-checkin.md) — 감정 이모지 매핑
- [폴더 구조](./06-folder-structure.md) — `src/constants/colors.ts`, `tailwind.config.js`
