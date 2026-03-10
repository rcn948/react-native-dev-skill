# 성능 최적화 가이드

## 목차
1. [리스트 최적화](#리스트-최적화)
2. [리렌더링 방지](#리렌더링-방지)
3. [이미지 최적화](#이미지-최적화)
4. [번들 크기](#번들-크기)
5. [네이티브 성능](#네이티브-성능)

## 리스트 최적화

### FlashList 사용 (FlatList 대체)
```bash
npx expo install @shopify/flash-list
```

```tsx
import { FlashList } from '@shopify/flash-list';

<FlashList
  data={items}
  renderItem={({ item }) => <ItemCard item={item} />}
  estimatedItemSize={80}  // 필수 — 예상 아이템 높이
  keyExtractor={(item) => item.id}
/>
```

### FlatList 최적화 (FlashList 사용 불가 시)
```tsx
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={(item) => item.id}
  getItemLayout={(_, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
  maxToRenderPerBatch={10}
  windowSize={5}
  removeClippedSubviews={true}
  initialNumToRender={10}
/>
```

### renderItem 메모이제이션
```tsx
const renderItem = useCallback(({ item }: { item: Todo }) => (
  <TodoItem todo={item} onToggle={handleToggle} />
), [handleToggle]);
```

## 리렌더링 방지

### React.memo
```tsx
export const UserCard = React.memo(function UserCard({ user }: Props) {
  return <View>...</View>;
});
```

### useMemo / useCallback
```tsx
// 비용이 큰 계산
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// 자식에게 전달하는 콜백
const handlePress = useCallback((id: string) => {
  router.push(`/details/${id}`);
}, []);
```

### Zustand 셀렉터
```tsx
// 나쁜 예 — 모든 스토어 변경에 리렌더
const store = useAppStore();

// 좋은 예 — count 변경 시에만 리렌더
const count = useAppStore((s) => s.count);
```

## 이미지 최적화

### expo-image 사용 (Image 대체)
```bash
npx expo install expo-image
```

```tsx
import { Image } from 'expo-image';

<Image
  source={{ uri: imageUrl }}
  style={{ width: 200, height: 200 }}
  contentFit="cover"
  placeholder={{ blurhash: 'LKO2?U%2Tw=w]~RBVZRi};RPxuwH' }}
  transition={200}
  cachePolicy="memory-disk"
/>
```

`expo-image`는 `react-native`의 `Image`보다 성능과 캐싱이 우수.

## 번들 크기

```bash
# 번들 분석
npx expo export --dump-sourcemap
npx source-map-explorer dist/bundles/*.js
```

- 필요한 것만 import (tree-shaking)
- 무거운 라이브러리는 대안 검토 (moment → dayjs, lodash → lodash-es 또는 직접 구현)
- 이미지/폰트 에셋 최적화

## 네이티브 성능

### Hermes 엔진
Expo 최신 버전은 Hermes 기본. 별도 설정 불필요.

### react-native-reanimated (UI 스레드 애니메이션)
```tsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';

function AnimatedBox() {
  const offset = useSharedValue(0);
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: withSpring(offset.value) }],
  }));

  return (
    <Pressable onPress={() => { offset.value = offset.value === 0 ? 100 : 0; }}>
      <Animated.View style={[styles.box, animatedStyle]} />
    </Pressable>
  );
}
```

Animated API 대신 Reanimated 사용 → JS 스레드 블로킹 방지.
