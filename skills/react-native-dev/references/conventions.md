# 코딩 컨벤션

## 네이밍

| 대상 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 | PascalCase | `UserProfile.tsx` |
| 훅 | camelCase, use 접두사 | `useAuth.ts` |
| 스토어 | camelCase, Store 접미사 | `authStore.ts` |
| 유틸 함수 | camelCase | `formatDate.ts` |
| 상수 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 타입/인터페이스 | PascalCase | `UserProfile`, `AuthState` |
| 폴더 | kebab-case 또는 camelCase | `user-profile/` |

## 컴포넌트 작성 규칙

### 함수 컴포넌트만 사용
```tsx
// 좋은 예
export function UserCard({ user }: UserCardProps) {
  return <View>...</View>;
}

// 나쁜 예 — class component 금지
class UserCard extends React.Component { }
```

### Props 타입은 인터페이스로 정의
```tsx
interface UserCardProps {
  user: User;
  onPress?: () => void;
}

export function UserCard({ user, onPress }: UserCardProps) { }
```

### 한 파일에 하나의 exported 컴포넌트
내부 헬퍼 컴포넌트는 같은 파일에 있어도 됨 (export하지 않으면).

## import 순서

```tsx
// 1. React / React Native
import { useState, useEffect } from 'react';
import { View, Text, Pressable } from 'react-native';

// 2. 외부 라이브러리
import { router } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';

// 3. 내부 모듈 (@ alias 사용)
import { useAuthStore } from '@/stores/authStore';
import { UserCard } from '@/components/features/UserCard';
import { formatDate } from '@/utils/formatDate';

// 4. 타입 (type-only import)
import type { User } from '@/types/user';
```

## 에러 처리

```tsx
// API 호출 시 try-catch
async function fetchData() {
  try {
    const data = await api.get('/endpoint');
    return data;
  } catch (error) {
    if (error instanceof AxiosError) {
      // HTTP 에러 처리
      console.error(`API Error: ${error.response?.status}`);
    }
    throw error;
  }
}
```

## 경로 alias

`tsconfig.json`에서 `@/` alias 설정:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

## 기타

- `console.log` 디버깅 후 반드시 제거 (또는 `__DEV__` 분기)
- 매직 넘버 금지 → 상수로 추출
- 인라인 스타일 최소화 → NativeWind className 또는 StyleSheet 사용
- `any` 타입 사용 금지 → 적절한 타입 명시
- 불필요한 `useEffect` 지양 → 이벤트 핸들러에서 처리
