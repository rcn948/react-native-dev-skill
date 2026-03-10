# Zustand 상태 관리 가이드

## 목차
1. [기본 스토어](#기본-스토어)
2. [비동기 액션](#비동기-액션)
3. [영속화 (Persist)](#영속화)
4. [슬라이스 패턴](#슬라이스-패턴)
5. [셀렉터 최적화](#셀렉터-최적화)

## 기본 스토어

```tsx
// src/stores/counterStore.ts
import { create } from 'zustand';

interface CounterState {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

export const useCounterStore = create<CounterState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));
```

컴포넌트에서 사용:
```tsx
function Counter() {
  const count = useCounterStore((s) => s.count);
  const increment = useCounterStore((s) => s.increment);
  return <Button title={`Count: ${count}`} onPress={increment} />;
}
```

## 비동기 액션

```tsx
// src/stores/todoStore.ts
import { create } from 'zustand';
import { todoApi } from '@/services/todoApi';

interface TodoState {
  todos: Todo[];
  loading: boolean;
  error: string | null;
  fetchTodos: () => Promise<void>;
  addTodo: (title: string) => Promise<void>;
}

export const useTodoStore = create<TodoState>((set, get) => ({
  todos: [],
  loading: false,
  error: null,

  fetchTodos: async () => {
    set({ loading: true, error: null });
    try {
      const todos = await todoApi.getAll();
      set({ todos, loading: false });
    } catch (e) {
      set({ error: (e as Error).message, loading: false });
    }
  },

  addTodo: async (title) => {
    const newTodo = await todoApi.create({ title, completed: false });
    set((state) => ({ todos: [...state.todos, newTodo] }));
  },
}));
```

## 영속화

AsyncStorage를 사용한 상태 영속화:

```tsx
// src/stores/settingsStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface SettingsState {
  theme: 'light' | 'dark';
  language: string;
  setTheme: (theme: 'light' | 'dark') => void;
  setLanguage: (lang: string) => void;
}

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      theme: 'light',
      language: 'ko',
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    {
      name: 'settings-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

### 민감 데이터 영속화 (SecureStore)

```tsx
import * as SecureStore from 'expo-secure-store';
import { StateStorage } from 'zustand/middleware';

const secureStorage: StateStorage = {
  getItem: async (name) => await SecureStore.getItemAsync(name),
  setItem: async (name, value) => await SecureStore.setItemAsync(name, value),
  removeItem: async (name) => await SecureStore.deleteItemAsync(name),
};

// persist의 storage 옵션에 사용
storage: createJSONStorage(() => secureStorage),
```

## 슬라이스 패턴

큰 앱에서 스토어를 분리할 때:

```tsx
// src/stores/slices/userSlice.ts
import { StateCreator } from 'zustand';

export interface UserSlice {
  user: User | null;
  setUser: (user: User | null) => void;
}

export const createUserSlice: StateCreator<UserSlice> = (set) => ({
  user: null,
  setUser: (user) => set({ user }),
});

// src/stores/slices/cartSlice.ts
export interface CartSlice {
  items: CartItem[];
  addItem: (item: CartItem) => void;
}

export const createCartSlice: StateCreator<CartSlice> = (set) => ({
  items: [],
  addItem: (item) => set((s) => ({ items: [...s.items, item] })),
});

// src/stores/appStore.ts — 합치기
import { create } from 'zustand';

export const useAppStore = create<UserSlice & CartSlice>()((...a) => ({
  ...createUserSlice(...a),
  ...createCartSlice(...a),
}));
```

## 셀렉터 최적화

리렌더링을 줄이려면 필요한 상태만 구독:

```tsx
// 나쁜 예 — 스토어 전체 구독, 모든 변경에 리렌더
const store = useTodoStore();

// 좋은 예 — 필요한 값만 구독
const todos = useTodoStore((s) => s.todos);
const loading = useTodoStore((s) => s.loading);

// 여러 값을 한번에 — shallow 비교
import { useShallow } from 'zustand/shallow';

const { todos, loading } = useTodoStore(
  useShallow((s) => ({ todos: s.todos, loading: s.loading }))
);
```

## MMKV (고속 동기 스토리지)

AsyncStorage보다 훨씬 빠른 동기 스토리지. 자주 접근하는 데이터에 적합.

```bash
npx expo install react-native-mmkv
```

### 기본 사용
```tsx
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

// 동기 — await 불필요
storage.set('user.name', 'John');
const name = storage.getString('user.name');

storage.set('user.age', 25);
const age = storage.getNumber('user.age');

storage.set('user.premium', true);
const isPremium = storage.getBoolean('user.premium');

storage.delete('user.name');
storage.clearAll();

// JSON
storage.set('user', JSON.stringify(user));
const user = JSON.parse(storage.getString('user') || '{}');
```

### useMMKV 훅
```tsx
import { useMMKVString, useMMKVNumber, useMMKVBoolean } from 'react-native-mmkv';

function Settings() {
  const [theme, setTheme] = useMMKVString('theme');
  const [fontSize, setFontSize] = useMMKVNumber('fontSize');
  const [notifications, setNotifications] = useMMKVBoolean('notifications');

  return (
    <>
      <Switch value={theme === 'dark'} onValueChange={(dark) => setTheme(dark ? 'dark' : 'light')} />
      <Slider value={fontSize} onValueChange={setFontSize} />
      <Switch value={notifications} onValueChange={setNotifications} />
    </>
  );
}
```

### Zustand + MMKV Persist
```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV();

const mmkvStorage = {
  getItem: (name: string) => storage.getString(name) ?? null,
  setItem: (name: string, value: string) => storage.set(name, value),
  removeItem: (name: string) => storage.delete(name),
};

export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      theme: 'light',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'settings-storage',
      storage: createJSONStorage(() => mmkvStorage),
    }
  )
);
```

### 스토리지 선택 가이드

| 스토리지 | 속도 | 동기 | 용도 |
|----------|------|------|------|
| AsyncStorage | 느림 | 비동기 | 간단한 앱, 소량 데이터 |
| MMKV | 빠름 | 동기 | 대량 데이터, 빈번한 접근 |
| SecureStore | 보통 | 비동기 | 민감 데이터 (토큰, 비밀번호) |
