# 테스트 가이드

## 테스트 스택

| 도구 | 용도 |
|------|------|
| Jest | 테스트 러너 (Expo 기본 포함) |
| React Native Testing Library | 컴포넌트 테스트 |
| MSW (Mock Service Worker) | API 모킹 |

## 설치

```bash
npx expo install jest-expo @testing-library/react-native @testing-library/jest-native
```

### jest.config.js
```js
module.exports = {
  preset: 'jest-expo',
  setupFilesAfterSetup: ['@testing-library/jest-native/extend-expect'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@sentry/react-native|native-base|react-native-svg)',
  ],
};
```

## 컴포넌트 테스트

```tsx
// src/components/ui/__tests__/Button.test.tsx
import { render, fireEvent, screen } from '@testing-library/react-native';
import { Button } from '../Button';

describe('Button', () => {
  it('renders title correctly', () => {
    render(<Button title="클릭" onPress={() => {}} />);
    expect(screen.getByText('클릭')).toBeTruthy();
  });

  it('calls onPress when pressed', () => {
    const onPress = jest.fn();
    render(<Button title="클릭" onPress={onPress} />);
    fireEvent.press(screen.getByText('클릭'));
    expect(onPress).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    const onPress = jest.fn();
    render(<Button title="클릭" onPress={onPress} disabled />);
    fireEvent.press(screen.getByText('클릭'));
    expect(onPress).not.toHaveBeenCalled();
  });
});
```

## 훅 테스트

```tsx
import { renderHook, act } from '@testing-library/react-native';
import { useCounterStore } from '@/stores/counterStore';

describe('useCounterStore', () => {
  beforeEach(() => {
    // 스토어 초기화
    useCounterStore.setState({ count: 0 });
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounterStore());
    act(() => result.current.increment());
    expect(result.current.count).toBe(1);
  });
});
```

## API 모킹 (MSW)

```tsx
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

const server = setupServer(
  http.get('/api/todos', () => {
    return HttpResponse.json([
      { id: 1, title: '할 일 1', completed: false },
    ]);
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## 테스트 실행

```bash
# 전체 테스트
npx jest

# 특정 파일
npx jest Button.test.tsx

# 워치 모드
npx jest --watch

# 커버리지
npx jest --coverage
```
