# 번들 최적화 심화 가이드

## 목차
1. [번들 분석](#번들-분석)
2. [Barrel Export 회피](#barrel-export-회피)
3. [Tree Shaking](#tree-shaking)
4. [코드 스플리팅](#코드-스플리팅)
5. [Hermes 번들 최적화](#hermes-번들-최적화)
6. [Android R8 최적화](#android-r8-최적화)
7. [에셋 최적화](#에셋-최적화)
8. [라이브러리 크기 관리](#라이브러리-크기-관리)

## 번들 분석

### JS 번들 시각화
```bash
# source-map-explorer로 번들 구성 확인
npx expo export --dump-sourcemap
npx source-map-explorer dist/bundles/ios-*.js
```

### 앱 전체 크기 분석
- **iOS**: Xcode > Window > Organizer > App Size
- **Android**: `bundletool dump manifest --bundle=app.aab`
- **EAS**: 빌드 완료 후 EAS 대시보드에서 크기 확인

## Barrel Export 회피

Barrel export(index.ts에서 re-export)는 tree-shaking을 방해하여 불필요한 코드가 번들에 포함됨.

```tsx
// 나쁜 예 — barrel import (전체 모듈 로드됨)
// src/components/index.ts
export { Button } from './Button';
export { Card } from './Card';
export { Modal } from './Modal';
// ...50개 컴포넌트

// 이렇게 import하면 50개 전부 번들에 포함될 수 있음
import { Button } from '@/components';

// 좋은 예 — 직접 import
import { Button } from '@/components/ui/Button';
```

## Tree Shaking

Metro 번들러의 tree-shaking 활용:

```tsx
// 나쁜 예 — 전체 라이브러리 import
import _ from 'lodash';
_.debounce(fn, 300);

// 좋은 예 — 개별 함수 import
import debounce from 'lodash/debounce';
debounce(fn, 300);
```

### 주요 라이브러리 대체
| 무거운 라이브러리 | 가벼운 대안 | 크기 절감 |
|-----------------|------------|---------|
| moment (330KB) | dayjs (7KB) | ~95% |
| lodash (530KB) | lodash-es 개별 import | ~90% |
| uuid (12KB) | expo-crypto randomUUID | 0KB (내장) |

## 코드 스플리팅

Expo Router는 파일 기반 라우팅으로 **자동 코드 스플리팅** 지원:
- 각 라우트 파일은 별도 청크로 분리
- 방문하지 않은 라우트의 코드는 로드되지 않음
- 별도 설정 불필요

### 지연 로딩 (수동)
```tsx
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('@/components/features/HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<ActivityIndicator />}>
      <HeavyChart data={chartData} />
    </Suspense>
  );
}
```

## Hermes 번들 최적화

### Hermes mmap
Hermes는 바이트코드를 mmap으로 로드하여 시작 시간 단축.
**번들 압축을 비활성화**해야 mmap이 동작:

```json
// app.json (Android)
{
  "expo": {
    "android": {
      "jsEngine": "hermes"
    }
  }
}
```

EAS Build에서 Hermes는 기본 활성화.

## Android R8 최적화

R8은 Android의 코드 축소/난독화 도구. EAS Build production에서 자동 활성화.

```
# proguard-rules.pro (커스텀 규칙)
-keep class com.myapp.** { *; }
-keepattributes Signature
```

```json
// app.json
{
  "expo": {
    "android": {
      "proguardRules": "./proguard-rules.pro"
    }
  }
}
```

## 에셋 최적화

### 이미지
- **WebP** 포맷 사용 (PNG/JPEG 대비 25-35% 작음)
- 플랫폼별 해상도: `image.png`, `image@2x.png`, `image@3x.png`
- 큰 이미지는 CDN에서 로드 (번들에 포함하지 않음)

### 폰트
- 사용하는 weight만 포함 (Regular, Bold 등)
- 가변 폰트(variable font) 사용 시 하나의 파일로 여러 weight 커버

### 앱 아이콘/스플래시
```json
// app.json — 적절한 크기 사용
{
  "expo": {
    "icon": "./assets/icon.png",          // 1024x1024
    "splash": {
      "image": "./assets/splash.png",     // 1284x2778
      "resizeMode": "contain"
    }
  }
}
```

## 라이브러리 크기 관리

새 라이브러리 추가 전 크기 확인:
```bash
# bundlephobia에서 크기 확인 (웹)
# https://bundlephobia.com/package/라이브러리명

# 또는 로컬에서
npx cost-of-modules
```

### 크기 기준
| 크기 | 판단 |
|------|------|
| < 10KB | 자유롭게 사용 |
| 10-50KB | 대안 검토 |
| 50-100KB | 정말 필요한지 재고 |
| > 100KB | 직접 구현 또는 가벼운 대안 찾기 |
