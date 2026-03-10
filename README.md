# React Native Dev Skill

React Native (Expo) 앱 개발을 위한 [Claude Code](https://claude.com/claude-code) skill 플러그인.

## 기본 스택

- **Expo** (Managed Workflow, SDK 53+)
- **Expo Router** (파일 기반 라우팅)
- **Zustand** (상태 관리)
- **NativeWind v5** (Tailwind CSS v4 기반 스타일링)
- **MMKV / AsyncStorage / SecureStore** (로컬 저장)
- **React Hook Form + Zod** (폼/검증)
- **TanStack Query** (서버 상태)

## 설치

```bash
claude plugin install rcn948/react-native-dev-skill
```

## 구조

```
skills/react-native-dev/
├── SKILL.md                         # 메인 skill 정의
└── references/
    ├── navigation.md                # Expo Router 네비게이션
    ├── state-management.md          # Zustand, MMKV, 영속화
    ├── styling.md                   # NativeWind v4/v5
    ├── platform-handling.md         # Keyboard, BackHandler, StatusBar
    ├── build-deploy.md              # EAS Build, 스토어 배포
    ├── testing.md                   # Jest, React Native Testing Library
    ├── performance.md               # 리스트 최적화, 리렌더링 방지
    ├── bundle-optimization.md       # 번들 분석, tree-shaking, 코드 스플리팅
    ├── react-compiler.md            # React Compiler (자동 메모이제이션)
    ├── native-modules.md            # Expo Modules API, 네이티브 연동
    └── conventions.md               # 코딩 컨벤션, 네이밍 규칙
```

## 주요 기능

- **프로젝트 생성**: Expo 기반 새 프로젝트 셋업
- **네비게이션**: 탭, 스택, 모달, 인증 플로우, 딥링크
- **상태 관리**: Zustand 스토어, MMKV 동기 스토리지, persist 미들웨어
- **스타일링**: NativeWind v5 (Tailwind v4), 다크 모드, 반응형
- **플랫폼 대응**: KeyboardAvoidingView, Android BackHandler, SafeArea
- **성능 최적화**: FlashList, FlatList 튜닝, Infinite Scroll, 번들 최적화
- **React Compiler**: 자동 메모이제이션, Reanimated 호환
- **빌드/배포**: EAS Build, OTA 업데이트, App Store/Google Play 제출
- **SafeArea 필수**: ScreenWrapper 패턴, edges 명시적 지정
- **OTA 업데이트 필수**: expo-updates, useOTAUpdates 훅

## 라이선스

MIT
