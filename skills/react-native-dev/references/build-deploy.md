# 빌드 및 배포 가이드

## 목차
1. [EAS Build 설정](#eas-build-설정)
2. [빌드 프로필](#빌드-프로필)
3. [iOS 배포](#ios-배포)
4. [Android 배포](#android-배포)
5. [OTA 업데이트](#ota-업데이트)
6. [Android 16KB 페이지 크기](#android-16kb-페이지-크기)
7. [환경변수](#환경변수)

## EAS Build 설정

```bash
npm install -g eas-cli
eas login
eas build:configure
```

### eas.json
```json
{
  "cli": {
    "version": ">= 5.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": {
        "simulator": true
      }
    },
    "preview": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "autoIncrement": true
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your@email.com",
        "ascAppId": "1234567890",
        "appleTeamId": "TEAM_ID"
      },
      "android": {
        "serviceAccountKeyPath": "./google-services.json",
        "track": "internal"
      }
    }
  }
}
```

## 빌드 프로필

| 프로필 | 용도 | 명령어 |
|--------|------|--------|
| development | 개발용 빌드 (dev client) | `eas build --profile development` |
| preview | 내부 테스트 배포 | `eas build --profile preview` |
| production | 스토어 출시 | `eas build --profile production` |

```bash
# iOS 빌드
eas build --platform ios --profile production

# Android 빌드
eas build --platform android --profile production

# 양쪽 동시
eas build --platform all --profile production
```

## iOS 배포

### App Store 제출
```bash
eas submit --platform ios --profile production
```

### TestFlight 배포
```bash
eas submit --platform ios --profile production
# App Store Connect에서 TestFlight 그룹에 추가
```

### 필수 설정 (app.json)
```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.company.appname",
      "buildNumber": "1",
      "supportsTablet": true,
      "infoPlist": {
        "NSCameraUsageDescription": "사진 촬영을 위해 카메라 접근이 필요합니다.",
        "NSPhotoLibraryUsageDescription": "사진 선택을 위해 앨범 접근이 필요합니다."
      }
    }
  }
}
```

## Android 배포

### Google Play 제출
```bash
eas submit --platform android --profile production
```

### 필수 설정 (app.json)
```json
{
  "expo": {
    "android": {
      "package": "com.company.appname",
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "permissions": [
        "CAMERA",
        "READ_EXTERNAL_STORAGE"
      ]
    }
  }
}
```

## OTA 업데이트

EAS Update를 사용한 코드 푸시:

```bash
# 업데이트 배포
eas update --branch production --message "버그 수정"

# 특정 브랜치에 업데이트
eas update --branch preview --message "새 기능 테스트"
```

### app.json 설정
```json
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/your-project-id"
    },
    "runtimeVersion": {
      "policy": "appVersion"
    }
  }
}
```

## Android 16KB 페이지 크기

Android 15+에서 16KB 페이지 크기 지원이 요구됨.

### 확인 사항
- Expo SDK 최신 버전은 기본 지원
- 커스텀 네이티브 모듈 사용 시 NDK 빌드에서 16KB 정렬 확인
- `expo prebuild` 후 `android/gradle.properties`에서:
```properties
# 16KB 페이지 크기 지원
android.enablePageSizeAlignment=true
```

### 테스트
```bash
# 16KB 페이지 크기 에뮬레이터에서 테스트
# Android Studio > AVD Manager > 16KB page size 옵션 활성화
```

## 환경변수

### expo-constants 사용
```json
// app.json
{
  "expo": {
    "extra": {
      "apiUrl": "https://api.example.com"
    }
  }
}
```

```tsx
import Constants from 'expo-constants';
const API_URL = Constants.expoConfig?.extra?.apiUrl;
```

### .env 파일 사용 (expo-env 또는 dotenv)
```bash
npx expo install expo-env
```

```
# .env
EXPO_PUBLIC_API_URL=https://api.example.com
```

```tsx
// 접두사 EXPO_PUBLIC_ 필수
const apiUrl = process.env.EXPO_PUBLIC_API_URL;
```

`.gitignore`에 `.env` 추가 필수. 민감한 키는 EAS Secrets 사용:
```bash
eas secret:create --name API_SECRET --value "..."
```
