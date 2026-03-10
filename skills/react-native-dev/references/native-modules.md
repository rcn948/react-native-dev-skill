# 네이티브 모듈 연동 가이드

## Expo 모듈 우선

Expo SDK에서 제공하는 모듈을 우선 사용한다. 직접 네이티브 코드 작성은 최후의 수단.

### 자주 사용하는 Expo 모듈

| 기능 | 패키지 | 설치 |
|------|--------|------|
| 카메라 | expo-camera | `npx expo install expo-camera` |
| 이미지 선택 | expo-image-picker | `npx expo install expo-image-picker` |
| 위치 | expo-location | `npx expo install expo-location` |
| 알림 | expo-notifications | `npx expo install expo-notifications` |
| 파일 시스템 | expo-file-system | `npx expo install expo-file-system` |
| 생체 인증 | expo-local-authentication | `npx expo install expo-local-authentication` |
| 링크/브라우저 | expo-linking, expo-web-browser | `npx expo install expo-linking expo-web-browser` |
| 비디오 | expo-av | `npx expo install expo-av` |
| 지도 | react-native-maps | `npx expo install react-native-maps` |
| WebView | react-native-webview | `npx expo install react-native-webview` |

`npx expo install`을 사용하면 Expo SDK 버전에 맞는 호환 버전이 자동 설치된다.

## Expo Config Plugin

서드파티 네이티브 모듈을 Managed Workflow에서 사용:

```json
// app.json
{
  "expo": {
    "plugins": [
      "expo-camera",
      ["expo-location", { "locationAlwaysAndWhenInUsePermission": "위치 정보를 사용합니다." }],
      ["expo-notifications", { "icon": "./assets/notification-icon.png" }]
    ]
  }
}
```

Config Plugin이 있는 라이브러리는 `expo prebuild` 없이도 EAS Build에서 자동 네이티브 설정.

## Expo Prebuild (CNG)

네이티브 코드 직접 수정이 필요한 경우:

```bash
# 네이티브 프로젝트 생성
npx expo prebuild

# 특정 플랫폼만
npx expo prebuild --platform ios
npx expo prebuild --platform android
```

생성된 `ios/`, `android/` 폴더를 직접 수정 가능. 단, CNG 패턴에서는 이 폴더를 `.gitignore`에 추가하고 빌드 시 재생성하는 것을 권장.

## 커스텀 네이티브 모듈 (Expo Modules API)

Expo Modules API로 Swift/Kotlin 네이티브 모듈 작성:

```bash
npx create-expo-module my-native-module
```

```swift
// ios/MyNativeModule.swift
import ExpoModulesCore

public class MyNativeModule: Module {
  public func definition() -> ModuleDefinition {
    Name("MyNativeModule")

    Function("hello") { (name: String) -> String in
      return "Hello, \(name)!"
    }
  }
}
```

```kotlin
// android/src/main/java/expo/modules/mynativemodule/MyNativeModule.kt
package expo.modules.mynativemodule

import expo.modules.kotlin.modules.Module
import expo.modules.kotlin.modules.ModuleDefinition

class MyNativeModule : Module() {
  override fun definition() = ModuleDefinition {
    Name("MyNativeModule")

    Function("hello") { name: String ->
      "Hello, $name!"
    }
  }
}
```

사용:
```tsx
import { requireNativeModule } from 'expo-modules-core';
const MyModule = requireNativeModule('MyNativeModule');
const result = MyModule.hello('World');
```
