---
name: compose-migration
description: View에서 Compose로 마이그레이션. "Compose 전환", "마이그레이션", "Epoxy 제거" 요청 시 사용.
---

# Compose Migration

View/Epoxy 컴포넌트를 Compose로 전환합니다.

## 원칙

- 기존 View와 동일한 외관/동작 유지 (스펙 누락 금지)
- Stateless composable 패턴 + state hoisting
- 기존 디자인 시스템 컴포넌트 활용
- 라이프사이클 동작 보존
- 마이그레이션 중 View/Compose 공존 허용

## 컴포넌트 구조

- `PrizmDesignSystem` 테마로 감싸기
- `modifier: Modifier = Modifier`를 첫 번째 기본 파라미터로
- 데이터는 파라미터로, 인터랙션은 람다 콜백(`on` prefix)으로
- `@Preview` 포함
- PrizmImage 사용 시 `Image` 또는 `Media` (domain) 파라미터 사용 (String URL 금지)
- PrizmVideo 사용 시 `Media` (domain) 파라미터 사용
- GeneralButton, BubbleButton, SquircleButton은 Compose 네이티브 버전 사용

## 상태 관리

- `remember` - recomposition 생존 상태
- `derivedStateOf` - 계산된 상태
- `LaunchedEffect` - 사이드 이펙트
- `LifecycleStartEffect`, `LifecycleResumeEffect` - 라이프사이클 이벤트
- `collectAsStateWithLifecycle()` - ViewModel 상태 수집
- LazyColumn/LazyRow `key` 파라미터 필수

## 의존성

- `androidx.compose.material3` for Material components
- `library.design.compose` 디자인 시스템 컴포넌트
- `kotlinx.collections.immutable` for list parameters
- Coil for image loading

## 마이그레이션 체크리스트

- [ ] XML → Composable 변환
- [ ] ViewBinding → Compose state
- [ ] Epoxy → LazyColumn/LazyRow (key 파라미터 필수)
- [ ] 애니메이션 마이그레이션
- [ ] @Preview 추가
- [ ] 라이프사이클 동작 테스트
- [ ] 외관 일치 확인

## Before/After 패턴

### 이미지 (PrizmImage)
```kotlin
// Before
viewBinding.image.load(image, Image.Size.LARGE_1080)

// After - domain Image
@Composable
fun MyComponent(image: Image) {
  PrizmImage(
    image = PrizmImage.Content(image = image, size = Image.Size.LARGE_1080),
    modifier = Modifier.fillMaxWidth()
  )
}

// After - domain Media
@Composable
fun MyComponent(media: Media) {
  PrizmImage(
    image = PrizmImage.Content(image = media.image, size = Image.Size.LARGE_1080),
    modifier = Modifier.fillMaxWidth()
  )
}

// 내부 전용 - 로컬 파일
PrizmImage(image = PrizmImage.LocalInform(file = file))

// 내부 전용 - 원격 URL
PrizmImage(image = PrizmImage.RemoteInform(path = imageUrl))
```

### 비디오 (PrizmVideo)
```kotlin
// Before
viewBinding.video.player = player

// After
@Composable
fun MyComponent(media: Media) {
  PrizmVideo(
    media = media,
    modifier = Modifier.fillMaxWidth(),
    isResume = true,
    isRepeat = false,
    size = Image.Size.LARGE_1080
  )
}

// 커스텀 플레이어
val player by rememberExoPlayer()
PrizmVideo(
  media = media,
  player = player,
  modifier = Modifier.fillMaxWidth(),
  isResume = true,
  isThumbnailVisible = true,
  size = Image.Size.LARGE_1080
)
```

### 클릭
```kotlin
// Before
@CallbackProp fun setOnClick(listener: OnClickListener?)

// After
modifier = Modifier.clickable { onClick() }
```

### Visibility
```kotlin
// Before
viewBinding.profile.isVisible = isVisible

// After
if (isVisible) { Profile(...) }
```

### 색상
```kotlin
// Before
context.getColor(R.color.primary)

// After
val color = LocalFoundationColor.current.primary
val tokenColor = LocalColorToken.current.primary
```

### Padding
```kotlin
// Before - @ModelProp padding parameter
@ModelProp fun setPadding(padding: Rect?) { updatePadding(padding ?: Rect()) }

// After - Modifier.padding (padding 파라미터 추가하지 않음)
MyComponent(modifier = Modifier.padding(16.dp))
```

### GeneralButton
```kotlin
// Before
GeneralButton(context).apply {
  text = "Button Text"
  setColorVariant(ColorVariant.Primary)
  setShapeVariant(ShapeVariant.Large)
  setOnClick { /* handle click */ }
}

// After
GeneralButton(
  text = "Button Text",
  onClick = { /* handle click */ },
  colorVariant = ColorVariant.Primary,
  size = GeneralButtonSize.Large,
)
```

### BubbleButton / SquircleButton
```kotlin
// Before
BubbleButton(context).apply {
  text = "Label"
  setColorVariant(ColorVariant.Primary)
}

// After
BubbleButton(
  text = "Label",
  onClick = {},
  colorVariant = ColorVariant.Primary,
  size = BubbleButtonSize.Small,
)
```

### 커스텀 ripple이 필요한 클릭 영역
```kotlin
// Surface 대신 Box + clip + drawBehind + clickable 패턴 사용
// ripple indication을 커스텀해야 할 때 필요
Box(
  modifier = Modifier
    .clip(shape)
    .drawBehind { drawRect(backgroundColor) }
    .clickable(
      interactionSource = rememberInteractionSource(),
      indication = rememberCellRipple(), // 또는 rememberPrimaryRipple()
      onClick = onClick,
    )
)
```

### 디자인 시스템 ripple/interaction 헬퍼
```kotlin
// ripple
rememberPrimaryRipple()  // Primary variant (white20)
rememberCellRipple()     // Secondary/Tertiary variant (black3)
rememberMediaRipple()    // Media variant

// interaction
rememberInteractionSource()
Modifier.pressedScale(interactionSource)  // 0.96 스케일 애니메이션
```

### 아이콘 파라미터는 Painter 타입으로
```kotlin
// drawable과 Lottie를 동일 파라미터로 처리 가능
// 컴포넌트 정의
@Composable
fun MyButton(icon: Painter? = null)

// 호출부 - 일반 drawable
MyButton(icon = painterResource(R.drawable.ic_bell))

// 호출부 - Lottie
val composition by rememberLottieComposition(LottieCompositionSpec.RawRes(R.raw.ic_bell_filled))
val animatable = rememberLottieAnimatable()
MyButton(icon = rememberLottiePainter(composition, animatable.progress))
```

## 공존 패턴

- XML에 Compose 삽입: `ComposeView`
- Compose에 View 삽입: `AndroidView`
