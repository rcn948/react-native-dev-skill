# 플랫폼 처리 가이드

## 목차
1. [Platform API](#platform-api)
2. [플랫폼별 파일](#플랫폼별-파일)
3. [SafeArea 심화](#safearea-심화)
4. [KeyboardAvoidingView](#keyboardavoidingview)
5. [StatusBar](#statusbar)
6. [Android 뒤로가기 (BackHandler)](#android-backhandler)

## Platform API

### Platform.select
```tsx
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  card: {
    padding: 16,
    borderRadius: 12,
    backgroundColor: '#fff',
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 8,
      },
      android: {
        elevation: 4,
      },
    }),
  },
  text: {
    fontFamily: Platform.select({
      ios: 'Helvetica Neue',
      android: 'Roboto',
    }),
  },
});
```

### Platform.OS
```tsx
import { Platform } from 'react-native';

const isIOS = Platform.OS === 'ios';
const isAndroid = Platform.OS === 'android';

// 조건부 렌더링
{isIOS && <IOSOnlyComponent />}

// 값 분기
const padding = Platform.OS === 'ios' ? 20 : 16;
```

## 플랫폼별 파일

```
components/
├── Button.tsx           # 공통 로직 / 기본
├── Button.ios.tsx       # iOS 전용
└── Button.android.tsx   # Android 전용
```

```tsx
// import 시 자동으로 플랫폼에 맞는 파일 선택
import Button from './components/Button';
```

## SafeArea 심화

### useSafeAreaInsets (세밀한 제어)
```tsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

function CustomHeader() {
  const insets = useSafeAreaInsets();

  return (
    <View style={[styles.header, { paddingTop: insets.top }]}>
      <Text>헤더</Text>
    </View>
  );
}
```

### contentInsetAdjustmentBehavior (ScrollView)
ScrollView 내부에서는 SafeAreaView 대신 이 속성을 사용하면 더 자연스러운 SafeArea 처리 가능:
```tsx
<ScrollView contentInsetAdjustmentBehavior="automatic">
  {/* SafeAreaView 래핑 불필요 */}
  <Content />
</ScrollView>
```

## KeyboardAvoidingView

키보드가 올라올 때 입력 필드가 가려지지 않도록 처리. **iOS와 Android에서 behavior가 다르다.**

```tsx
import { KeyboardAvoidingView, Platform, ScrollView, TextInput } from 'react-native';

function FormScreen() {
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      style={{ flex: 1 }}
      keyboardVerticalOffset={Platform.select({ ios: 88, android: 0 })}
    >
      <ScrollView>
        <TextInput placeholder="이름" />
        <TextInput placeholder="이메일" />
        <TextInput placeholder="메시지" multiline />
      </ScrollView>
    </KeyboardAvoidingView>
  );
}
```

| 플랫폼 | behavior | keyboardVerticalOffset |
|--------|----------|----------------------|
| iOS | `'padding'` | 헤더 높이 (보통 88) |
| Android | `'height'` | 0 |

## StatusBar

```tsx
import { StatusBar, Platform } from 'react-native';

function Screen() {
  return (
    <>
      <StatusBar
        barStyle="dark-content"  // 'light-content' for dark backgrounds
        backgroundColor={Platform.OS === 'android' ? '#fff' : undefined}
        translucent={Platform.OS === 'android'}
      />
      <Content />
    </>
  );
}
```

Expo Router에서는 `expo-status-bar` 사용:
```tsx
import { StatusBar } from 'expo-status-bar';

<StatusBar style="auto" />  // light/dark 자동 감지
```

## Android BackHandler

Android 하드웨어 뒤로가기 버튼 커스텀 처리:

```tsx
import { useEffect, useCallback } from 'react';
import { BackHandler, Platform } from 'react-native';

export function useBackHandler(handler: () => boolean) {
  useEffect(() => {
    if (Platform.OS !== 'android') return;

    const subscription = BackHandler.addEventListener(
      'hardwareBackPress',
      handler
    );

    return () => subscription.remove();
  }, [handler]);
}

// 사용 예 — 저장하지 않은 변경사항 경고
function EditScreen() {
  const [hasChanges, setHasChanges] = useState(false);

  useBackHandler(
    useCallback(() => {
      if (hasChanges) {
        Alert.alert(
          '변경사항 저장',
          '저장하지 않은 변경사항이 있습니다. 나가시겠습니까?',
          [
            { text: '계속 편집', style: 'cancel' },
            { text: '나가기', style: 'destructive', onPress: () => router.back() },
          ]
        );
        return true; // 기본 뒤로가기 방지
      }
      return false; // 기본 뒤로가기 허용
    }, [hasChanges])
  );

  return <View>...</View>;
}
```

## 빠른 참조

| API | 용도 |
|-----|------|
| `Platform.OS` | 플랫폼 확인 ('ios' / 'android') |
| `Platform.select()` | 플랫폼별 값 |
| `Platform.Version` | OS 버전 |
| `.ios.tsx` / `.android.tsx` | 플랫폼별 파일 |
| `SafeAreaView` / `useSafeAreaInsets` | 노치/홈바 영역 |
| `KeyboardAvoidingView` | 키보드 처리 |
| `StatusBar` | 상태바 스타일 |
| `BackHandler` | Android 뒤로가기 |
