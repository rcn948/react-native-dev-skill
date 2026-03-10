# NativeWind v4 스타일링 가이드

## 목차
1. [설치 및 설정](#설치-및-설정)
2. [기본 사용법](#기본-사용법)
3. [다크 모드](#다크-모드)
4. [반응형 디자인](#반응형-디자인)
5. [커스텀 테마](#커스텀-테마)
6. [애니메이션](#애니메이션)
7. [웹 포함 시 대안](#웹-포함-시-대안)

## 설치 및 설정

```bash
npx expo install nativewind tailwindcss
```

### tailwind.config.js
```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './app/**/*.{js,jsx,ts,tsx}',
    './src/**/*.{js,jsx,ts,tsx}',
  ],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### global.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### babel.config.js
```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: [
      ['babel-preset-expo', { jsxImportSource: 'nativewind' }],
      'nativewind/babel',
    ],
  };
};
```

### metro.config.js
```js
const { getDefaultConfig } = require('expo/metro-config');
const { withNativeWind } = require('nativewind/metro');

const config = getDefaultConfig(__dirname);
module.exports = withNativeWind(config, { input: './global.css' });
```

### app/_layout.tsx에서 CSS import
```tsx
import '../global.css';
```

### nativewind-env.d.ts
```ts
/// <reference types="nativewind/types" />
```

## 기본 사용법

```tsx
import { View, Text, Pressable } from 'react-native';

export function Card({ title, description }: CardProps) {
  return (
    <View className="bg-white rounded-2xl p-4 shadow-sm mx-4 mb-3">
      <Text className="text-lg font-bold text-gray-900">{title}</Text>
      <Text className="text-sm text-gray-500 mt-1">{description}</Text>
      <Pressable className="mt-3 bg-blue-500 rounded-lg py-2 px-4 active:bg-blue-600">
        <Text className="text-white text-center font-semibold">자세히 보기</Text>
      </Pressable>
    </View>
  );
}
```

### 조건부 스타일
```tsx
<View className={`p-4 rounded-lg ${isActive ? 'bg-blue-100' : 'bg-gray-100'}`}>
```

### clsx/twMerge 활용 (복잡한 조건부)
```tsx
import { twMerge } from 'tailwind-merge';
import clsx from 'clsx';

function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

<Text className={cn('text-base', isError && 'text-red-500', isDisabled && 'opacity-50')}>
```

## 다크 모드

```tsx
// useColorScheme 사용
import { useColorScheme } from 'nativewind';

function ThemeToggle() {
  const { colorScheme, toggleColorScheme } = useColorScheme();
  return (
    <Pressable onPress={toggleColorScheme}>
      <Text className="text-black dark:text-white">
        현재: {colorScheme}
      </Text>
    </Pressable>
  );
}

// className에서 dark: 접두사
<View className="bg-white dark:bg-gray-900">
  <Text className="text-black dark:text-white">다크 모드 지원</Text>
</View>
```

## 반응형 디자인

NativeWind는 화면 너비 기반 breakpoint 지원:

```tsx
<View className="flex-col md:flex-row">
  <View className="w-full md:w-1/2">
    <Text className="text-base lg:text-lg">반응형 텍스트</Text>
  </View>
</View>
```

| Breakpoint | 최소 너비 |
|------------|-----------|
| sm | 640px |
| md | 768px |
| lg | 1024px |
| xl | 1280px |

## 커스텀 테마

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          900: '#1e3a8a',
        },
        accent: '#f59e0b',
      },
      fontFamily: {
        sans: ['Pretendard'],
        heading: ['Pretendard-Bold'],
      },
      borderRadius: {
        '4xl': '2rem',
      },
    },
  },
};
```

사용:
```tsx
<View className="bg-primary-500 rounded-4xl">
  <Text className="font-heading text-white">커스텀 테마</Text>
</View>
```

## 애니메이션

NativeWind는 `react-native-reanimated`와 함께 사용:

```bash
npx expo install react-native-reanimated
```

```tsx
import Animated, { FadeInDown } from 'react-native-reanimated';

<Animated.View
  entering={FadeInDown.duration(300)}
  className="bg-white rounded-xl p-4"
>
  <Text>애니메이션 카드</Text>
</Animated.View>
```

## 웹 포함 시 대안

웹을 포함하는 경우 NativeWind 대신 `StyleSheet.create()` 사용을 고려:

```tsx
import { StyleSheet, View, Text } from 'react-native';

export function Card() {
  return (
    <View style={styles.card}>
      <Text style={styles.title}>제목</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    backgroundColor: '#fff',
    borderRadius: 16,
    padding: 16,
    marginHorizontal: 16,
    marginBottom: 12,
    // 웹에서도 동작하는 그림자
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3, // Android
  },
  title: {
    fontSize: 18,
    fontWeight: '700',
    color: '#111',
  },
});
```
