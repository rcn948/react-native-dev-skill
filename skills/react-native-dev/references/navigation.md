# Expo Router 네비게이션 가이드

## 목차
1. [기본 구조](#기본-구조)
2. [레이아웃 패턴](#레이아웃-패턴)
3. [탭 네비게이션](#탭-네비게이션)
4. [스택 네비게이션](#스택-네비게이션)
5. [인증 플로우](#인증-플로우)
6. [딥링크](#딥링크)
7. [모달](#모달)

## 기본 구조

Expo Router는 파일 시스템 기반 라우팅. `app/` 디렉토리의 파일 구조가 곧 URL 구조.

```
app/
├── _layout.tsx          # 루트 레이아웃
├── index.tsx            # / (홈)
├── +not-found.tsx       # 404
├── (tabs)/              # 탭 그룹
│   ├── _layout.tsx      # 탭 레이아웃
│   ├── index.tsx        # 첫 번째 탭
│   ├── explore.tsx      # 두 번째 탭
│   └── profile.tsx      # 세 번째 탭
├── (auth)/              # 인증 그룹
│   ├── _layout.tsx
│   ├── login.tsx
│   └── register.tsx
└── [id].tsx             # 동적 라우트 /123
```

## 레이아웃 패턴

### 루트 레이아�트
```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';
import { SafeAreaProvider } from 'react-native-safe-area-context';

export default function RootLayout() {
  return (
    <SafeAreaProvider>
      <Stack screenOptions={{ headerShown: false }}>
        <Stack.Screen name="(tabs)" />
        <Stack.Screen name="(auth)" />
        <Stack.Screen name="+not-found" />
      </Stack>
    </SafeAreaProvider>
  );
}
```

### 그룹 라우트 `(group)`
괄호로 감싼 폴더는 URL에 포함되지 않음. 레이아웃 공유용.
- `(tabs)/index.tsx` → URL은 `/`
- `(auth)/login.tsx` → URL은 `/login`

## 탭 네비게이션

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        headerShown: false,
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: '홈',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="explore"
        options={{
          title: '탐색',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="search" size={size} color={color} />
          ),
        }}
      />
    </Tabs>
  );
}
```

## 스택 네비게이션

```tsx
// 네비게이션 이동
import { router } from 'expo-router';

// 이동
router.push('/details/123');

// 대체 (뒤로가기 불가)
router.replace('/home');

// 뒤로
router.back();

// 파라미터 전달
router.push({ pathname: '/details/[id]', params: { id: '123' } });
```

### 동적 라우트
```tsx
// app/details/[id].tsx
import { useLocalSearchParams } from 'expo-router';

export default function DetailsScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  return <Text>상세: {id}</Text>;
}
```

## 인증 플로우

조건부 라우팅으로 인증/비인증 화면 분리:

```tsx
// app/_layout.tsx
import { useAuthStore } from '@/stores/authStore';
import { Redirect, Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Screen name="(tabs)" />
      <Stack.Screen name="(auth)" />
    </Stack>
  );
}
```

```tsx
// app/(tabs)/_layout.tsx
import { Redirect } from 'expo-router';
import { useAuthStore } from '@/stores/authStore';

export default function TabLayout() {
  const isAuthenticated = useAuthStore((s) => s.isAuthenticated);

  if (!isAuthenticated) {
    return <Redirect href="/(auth)/login" />;
  }

  return <Tabs>{/* ... */}</Tabs>;
}
```

## 딥링크

Expo Router는 자동으로 딥링크 지원. `app.json`에서 scheme 설정:

```json
{
  "expo": {
    "scheme": "myapp",
    "web": {
      "bundler": "metro"
    }
  }
}
```

딥링크 URL: `myapp://details/123` → `app/details/[id].tsx`

## 모달

```tsx
// app/_layout.tsx
<Stack>
  <Stack.Screen name="(tabs)" />
  <Stack.Screen
    name="modal"
    options={{ presentation: 'modal' }}
  />
</Stack>

// app/modal.tsx
export default function ModalScreen() {
  return (
    <View>
      <Text>모달 화면</Text>
    </View>
  );
}

// 모달 열기
router.push('/modal');
```

## 타입 안전 라우팅

```tsx
// typed routes 활성화: app.json
{
  "expo": {
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

이렇게 하면 `router.push()` 등에서 자동완성과 타입 검사가 적용된다.
