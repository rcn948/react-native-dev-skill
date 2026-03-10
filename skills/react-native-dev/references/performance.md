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

### FlatList 상세 최적화

```tsx
import { FlatList, ListRenderItem } from 'react-native';
import { memo, useCallback } from 'react';

// 1. 리스트 아이템은 반드시 memo
const ListItem = memo(function ListItem({ item, onPress }: { item: Item; onPress: (id: string) => void }) {
  return (
    <Pressable onPress={() => onPress(item.id)} style={styles.item}>
      <Text>{item.title}</Text>
    </Pressable>
  );
});

function OptimizedList({ data }: { data: Item[] }) {
  // 2. 콜백 메모이제이션
  const handlePress = useCallback((id: string) => {
    router.push(`/details/${id}`);
  }, []);

  // 3. renderItem 메모이제이션
  const renderItem: ListRenderItem<Item> = useCallback(
    ({ item }) => <ListItem item={item} onPress={handlePress} />,
    [handlePress]
  );

  const keyExtractor = useCallback((item: Item) => item.id, []);

  // 4. 고정 높이일 때 getItemLayout으로 측정 건너뛰기
  const getItemLayout = useCallback(
    (_: any, index: number) => ({
      length: ITEM_HEIGHT,
      offset: ITEM_HEIGHT * index,
      index,
    }),
    []
  );

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      getItemLayout={getItemLayout}
      removeClippedSubviews        // 화면 밖 아이템 언마운트
      maxToRenderPerBatch={10}     // 배치당 렌더 수
      windowSize={5}               // 렌더 윈도우 배수
      initialNumToRender={10}      // 초기 렌더 수
      updateCellsBatchingPeriod={50} // 배치 업데이트 주기(ms)
    />
  );
}

const ITEM_HEIGHT = 72;
```

### 성능 Props 레퍼런스

| Prop | 용도 |
|------|------|
| `removeClippedSubviews` | 화면 밖 아이템 언마운트 |
| `maxToRenderPerBatch` | 배치당 렌더할 아이템 수 |
| `windowSize` | 렌더 윈도우 배수 (뷰포트 기준) |
| `initialNumToRender` | 초기 렌더 아이템 수 |
| `getItemLayout` | 고정 높이 시 측정 스킵 |
| `updateCellsBatchingPeriod` | 배치 업데이트 주기 |

### SectionList

```tsx
import { SectionList } from 'react-native';

<SectionList
  sections={sections}
  renderItem={renderItem}
  renderSectionHeader={({ section }) => (
    <View style={styles.sectionHeader}>
      <Text style={styles.sectionTitle}>{section.title}</Text>
    </View>
  )}
  keyExtractor={keyExtractor}
  stickySectionHeadersEnabled
/>
```

### Pull to Refresh

```tsx
<FlatList
  data={data}
  renderItem={renderItem}
  refreshControl={
    <RefreshControl
      refreshing={refreshing}
      onRefresh={handleRefresh}
      tintColor="#007AFF"
    />
  }
/>
```

### Infinite Scroll

```tsx
function InfiniteList() {
  const [data, setData] = useState<Item[]>([]);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const loadMore = useCallback(async () => {
    if (loading || !hasMore) return;
    setLoading(true);
    const newItems = await fetchMoreItems(data.length);
    if (newItems.length === 0) {
      setHasMore(false);
    } else {
      setData(prev => [...prev, ...newItems]);
    }
    setLoading(false);
  }, [data.length, loading, hasMore]);

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={loading ? <ActivityIndicator /> : null}
    />
  );
}
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

### 번들 분석
```bash
npx expo export --dump-sourcemap
npx source-map-explorer dist/bundles/*.js
```

### 최적화 체크리스트
- **barrel import 회피**: `import { X } from '@/components'` 대신 `import { X } from '@/components/X'` — barrel export는 tree-shaking 방해
- **tree-shaking**: 필요한 것만 import. `import _ from 'lodash'` 대신 `import debounce from 'lodash/debounce'`
- **무거운 라이브러리 대체**: moment → dayjs, lodash → lodash-es 또는 직접 구현
- **이미지 최적화**: WebP 포맷, 적절한 해상도 (1x/2x/3x)
- **코드 스플리팅**: Expo Router는 자동으로 라우트별 코드 스플리팅
- **console.log 제거**: `babel-plugin-transform-remove-console` 사용
  ```js
  // babel.config.js (production만)
  plugins: [['transform-remove-console', { exclude: ['error', 'warn'] }]]
  ```

### Android R8 코드 축소
EAS Build production 프로필에서 자동 활성화. `app.json`에서 ProGuard 규칙 추가 가능:
```json
{
  "expo": {
    "android": {
      "proguardRules": "./proguard-rules.pro"
    }
  }
}
```

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
