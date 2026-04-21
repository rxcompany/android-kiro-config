# Technology Stack

> 버전은 `gradle/libs.versions.toml`이 SSoT.

## Core Libraries
- **DI**: Hilt
- **Architecture**: Mavericks (MVI), AndroidX Lifecycle, Navigation, Room
- **UI**: Jetpack Compose (Material3), Epoxy, Lottie
- **Network**: Retrofit, OkHttp, Moshi
- **Media**: Media3 (ExoPlayer), CameraX, Coil
- **Analytics**: Firebase (Crashlytics, Analytics, FCM), Mixpanel, Branch
- **3rd Party**: Kakao SDK, Facebook SDK, AWS IVS Chat, Google ML Kit
- **Utilities**: Coroutines, Joda-Time, Timber, JSoup
- **Testing**: Kotest, MockK

## Development Commands
```bash
./gradlew assembleInhouseDebug        # Debug build
./gradlew assembleProductRelease       # Production build
./gradlew testInhouseDebugUnitTest    # Unit tests
ktlint --format <file_paths>          # Formatting
./gradlew assembleInhouseDebug -PcomposeCompilerReports=true  # Compose reports
./gradlew assembleInhouseDebug -PcomposeCompilerMetrics=true   # Compose metrics
```

## Build Variants

### Flavor (`target` dimension)
- **inhouse**: 내부 테스트 (applicationIdSuffix `.inhouse`)
- **product**: 프로덕션

### Build Type
- **debug**: 디버깅 활성화, 난독화/리소스 축소 비활성화, Crashlytics 수집 비활성화
- **release**: 난독화/리소스 축소 활성화, Crashlytics 수집 및 매핑 업로드 활성화

### 조합
| Variant | 용도 |
|---------|------|
| inhouseDebug | 개발 시 사용 (기본 개발 빌드) |
| inhouseRelease | QA 빌드 |
| productRelease | RC / 프로덕션 (스토어 배포) |
| productDebug | 일반적으로 미사용. 릴리즈 빌드에서 디버깅이 필요한 경우에만 개발자가 사용 |

`BuildConfig`에서 `inhouse`/`product` (flavor)와 `debug`/`release` (buildType)를 구분할 수 있는 플래그가 제공됨.
