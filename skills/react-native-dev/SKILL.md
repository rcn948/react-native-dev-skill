---
name: react-native-dev
description: React Native 앱 개발 skill. React Native, Expo, 모바일 앱 개발, iOS/Android 앱 만들기, 네이티브 앱 빌드 관련 작업에 사용한다. 사용자가 "앱 만들어줘", "React Native 프로젝트", "Expo 앱", "모바일 앱", "iOS 앱", "Android 앱" 등을 언급하면 반드시 이 skill을 사용할 것. 새 프로젝트 생성, 컴포넌트 개발, 네비게이션 설정, 상태 관리, 스타일링, 빌드/배포 모두 포함.
---

# React Native 앱 개발 Skill

## 기본 스택

| 영역 | 선택 | 비고 |
|------|------|------|
| 프레임워크 | Expo (Managed Workflow) | `npx create-expo-app@latest` |
| 라우팅 | Expo Router (파일 기반) | `app/` 디렉토리 |
| 상태 관리 | Zustand | 경량, 보일러플레이트 최소 |
| 스타일링 | NativeWind v5 (Tailwind v4) | Babel 불필요, CSS-first |
| 폼/검증 | React Hook Form + Zod | |
| HTTP | Axios 또는 fetch + TanStack Query | |
| 로컬 저장 | expo-secure-store (민감), MMKV (일반/고속), AsyncStorage (간단) | |
| 아이콘 | @expo/vector-icons | |

## 버전 정책

항상 최신 안정 버전을 사용한다. 특정 버전을 하드코딩하지 않는다.

- **Expo SDK**: 53 이상 최신 버전
- **React Native / React**: Expo SDK에 맞는 최신 버전 자동 적용
- 프로젝트 생성: `npx create-expo-app@latest`
- 패키지 버전 맞추기: `npx expo install --fix`
- Expo SDK 업그레이드: `npx expo install expo@latest`

## 플랫폼 타겟 구분

프로젝트 시작 시 사용자에게 **플랫폼 타겟**을 확인한다:

### 네이티브 전용 (iOS + Android)
- NativeWind 사용
- `react-native-screens`, `react-native-gesture-handler` 등 네이티브 모듈 자유 사용
- 플랫폼별 코드: `Platform.select()`, `.ios.tsx` / `.android.tsx`

### 웹 포함 (iOS + Android + Web)
- NativeWind 대신 `StyleSheet.create()` + 반응형 유틸 고려 (웹 호환성)
- 웹 미지원 네이티브 모듈 회피 또는 `Platform.OS === 'web'` 분기
- `react-native-web` 호환성 확인 필수
- Expo Router의 웹 라우팅 활용

## 핵심 원칙

### 1. Expo Managed Workflow 우선
bare workflow는 네이티브 모듈 직접 수정이 필요한 경우에만 사용한다.
`expo prebuild`로 네이티브 코드 접근 가능 (CNG - Continuous Native Generation).

### 2. 파일 기반 라우팅 (Expo Router)
`app/` 디렉토리 구조가 곧 라우트. 네비게이션 구조는 references/navigation.md 참고.

### 3. 최신 버전 유지
구버전 API나 deprecated 패턴 사용 금지. New Architecture 기본.

### 4. TypeScript 필수
모든 컴포넌트, 훅, 유틸은 `.tsx` / `.ts`로 작성한다.

### 5. 컴포넌트 구조
```
src/
├── components/     # 재사용 UI 컴포넌트
│   ├── ui/         # 기본 UI (Button, Input, Card 등)
│   └── features/   # 기능별 컴포넌트
├── hooks/          # 커스텀 훅
├── stores/         # Zustand 스토어
├── services/       # API 호출, 외부 서비스
├── utils/          # 유틸리티 함수
├── constants/      # 상수, 테마 값
└── types/          # 공유 타입 정의
app/
├── (tabs)/         # 탭 네비게이션 그룹
├── (auth)/         # 인증 관련 라우트 그룹
├── _layout.tsx     # 루트 레이아웃
└── +not-found.tsx  # 404 페이지
```

### 6. SafeArea 필수
`react-native-safe-area-context`를 반드시 사용한다. RN 내장 SafeAreaView는 deprecated.

```tsx
import { SafeAreaView } from 'react-native-safe-area-context';

// edges를 명시적으로 지정 — 불필요한 영역 패딩 방지
<SafeAreaView edges={['top', 'bottom']}>
  {children}
</SafeAreaView>
```

**ScreenWrapper 패턴** — 모든 화면에서 재사용:
```tsx
// src/components/ui/ScreenWrapper.tsx
import { SafeAreaView, type Edge } from 'react-native-safe-area-context';
import { View } from 'react-native';

interface ScreenWrapperProps {
  children: React.ReactNode;
  edges?: Edge[];
}

export function ScreenWrapper({ children, edges = ['top'] }: ScreenWrapperProps) {
  return (
    <SafeAreaView edges={edges} style={{ flex: 1 }}>
      <View style={{ flex: 1 }}>
        {children}
      </View>
    </SafeAreaView>
  );
}
```

### 7. OTA 업데이트 필수
`expo-updates`를 설치하고 앱 시작 시 업데이트를 체크한다.

```tsx
// src/hooks/useOTAUpdates.ts
import * as Updates from 'expo-updates';
import { useEffect } from 'react';
import { Alert } from 'react-native';

export function useOTAUpdates() {
  useEffect(() => {
    if (__DEV__) return; // 개발 모드에서는 스킵

    async function checkUpdates() {
      try {
        const update = await Updates.checkForUpdateAsync();
        if (update.isAvailable) {
          await Updates.fetchUpdateAsync();
          Alert.alert(
            '업데이트 완료',
            '새 버전이 적용됩니다.',
            [{ text: '확인', onPress: () => Updates.reloadAsync() }]
          );
        }
      } catch (e) {
        console.log('OTA update check failed:', e);
      }
    }
    checkUpdates();
  }, []);
}
```

루트 레이아웃에서 호출:
```tsx
// app/_layout.tsx
export default function RootLayout() {
  useOTAUpdates();
  // ...
}
```

### 8. 플랫폼 대응
- `KeyboardAvoidingView`: iOS는 `behavior="padding"`, Android는 `behavior="height"`
- `BackHandler`: Android 뒤로가기 커스텀 처리 (저장 확인 등)
- `StatusBar`: 플랫폼별 스타일 분기
- 상세 패턴은 references/platform-handling.md 참고

### 9. React Compiler (Expo SDK 53+)
수동 `useMemo`, `useCallback`, `React.memo` 대신 컴파일러가 자동 메모이제이션.
Reanimated 사용 시 `.get()`/`.set()` 패턴 필요. references/react-compiler.md 참고.

## Reference 파일

필요 시 아래 파일을 읽어 상세 가이드를 확인한다:

| 파일 | 내용 | 언제 읽나 |
|------|------|-----------|
| references/navigation.md | Expo Router 네비게이션 패턴 | 라우팅/네비게이션 설정 시 |
| references/state-management.md | Zustand 패턴, 영속화, MMKV | 상태 관리 구현 시 |
| references/styling.md | NativeWind v4/v5 설정, 패턴 | 스타일링 작업 시 |
| references/platform-handling.md | 플랫폼별 처리, Keyboard, BackHandler | 플랫폼 분기/키보드/뒤로가기 처리 시 |
| references/build-deploy.md | 빌드, 서명, 스토어 배포 | 빌드/배포 시 |
| references/testing.md | 테스트 전략, 도구 | 테스트 작성 시 |
| references/performance.md | 리스트 최적화, 리렌더링 방지 | 성능 개선 시 |
| references/bundle-optimization.md | 번들 분석, tree-shaking, 코드 스플리팅 | 앱 크기 최적화 시 |
| references/react-compiler.md | React Compiler 설정, 주의사항 | React 19+ 자동 메모이제이션 |
| references/native-modules.md | 네이티브 모듈 연동 | 네이티브 코드 필요 시 |
| references/conventions.md | 코딩 컨벤션, 네이밍 | 항상 (코드 작성 시) |
