# React Compiler 가이드

## 개요

React Compiler는 React 19+에서 사용 가능한 자동 메모이제이션 도구.
수동 `useMemo`, `useCallback`, `React.memo`를 자동으로 처리해준다.

Expo SDK 53+에서 기본 활성화.

## 설정

### Expo에서 활성화
```json
// app.json
{
  "expo": {
    "experiments": {
      "reactCompiler": true
    }
  }
}
```

### babel.config.js (수동 설정 시)
```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      ['babel-plugin-react-compiler', {}],
    ],
  };
};
```

## 무엇이 바뀌나

### Before (수동 메모이제이션)
```tsx
const MemoizedItem = React.memo(function Item({ data, onPress }: Props) {
  const formattedDate = useMemo(() => formatDate(data.createdAt), [data.createdAt]);
  const handlePress = useCallback(() => onPress(data.id), [data.id, onPress]);

  return (
    <Pressable onPress={handlePress}>
      <Text>{formattedDate}</Text>
    </Pressable>
  );
});
```

### After (React Compiler)
```tsx
// memo, useMemo, useCallback 불필요 — 컴파일러가 자동 처리
function Item({ data, onPress }: Props) {
  const formattedDate = formatDate(data.createdAt);
  const handlePress = () => onPress(data.id);

  return (
    <Pressable onPress={handlePress}>
      <Text>{formattedDate}</Text>
    </Pressable>
  );
}
```

## 주의사항

### Reanimated와 함께 사용 시
Reanimated의 shared value는 `.get()` / `.set()` 사용 (`.value` 대신):
```tsx
// React Compiler와 호환
const offset = useSharedValue(0);

// .get() / .set() 사용
const animatedStyle = useAnimatedStyle(() => ({
  transform: [{ translateX: offset.get() }],
}));

// 값 변경
offset.set(100);
```

### 컴파일러가 최적화하지 않는 경우
- 외부 mutable 변수 참조
- `useRef().current` 직접 변경 후 렌더링에 사용
- Rules of React 위반 코드

### 기존 코드와 호환성
- 기존 `useMemo`, `useCallback`, `React.memo`가 있어도 문제없음 (무시됨)
- 점진적 도입 가능 — 파일 상단에 `'use no memo'`로 특정 파일 제외

```tsx
'use no memo'; // 이 파일은 React Compiler 적용 제외

export function LegacyComponent() { }
```

## 디버깅

React DevTools에서 컴파일러 적용 여부 확인 가능.
컴포넌트 이름 옆에 ✨ 표시가 나타나면 컴파일러가 최적화한 것.
