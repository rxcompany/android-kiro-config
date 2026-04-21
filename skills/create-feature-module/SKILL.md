---
name: create-feature-module
description: 새 피처 모듈 보일러플레이트 생성. "피처 모듈 생성", "새 모듈", "Fragment 생성" 요청 시 사용.
---

# Feature Module 생성

MVI 아키텍처 기반 Fragment + ViewModel + Screen 보일러플레이트를 생성합니다.

## 생성 파일

```
feature/<feature>/
├── <Feature>Fragment.kt
├── <Feature>ViewModel.kt
├── <Feature>State.kt
└── <Feature>Screen.kt
```

## Fragment 템플릿

```kotlin
@AndroidEntryPoint
class <Feature>Fragment : ComposeFragment(), MavericksView {
  private val viewModel: <Feature>ViewModel by fragmentViewModel()

  @Composable
  override fun Content() {
    PrizmDesignSystem {
      <Feature>Screen(
        viewModel = viewModel,
        navController = findNavController()
      )
    }
  }

  override fun invalidate() = Unit
}
```

## ViewModel 템플릿

```kotlin
class <Feature>ViewModel @AssistedInject constructor(
  @Assisted initialState: <Feature>State,
  // private val useCase: SomeUseCase
) : MavericksViewModel<<Feature>State>(initialState) {

  @AssistedFactory
  interface Factory : AssistedViewModelFactory<<Feature>ViewModel, <Feature>State> {
    override fun create(state: <Feature>State): <Feature>ViewModel
  }

  companion object : MavericksViewModelFactory<<Feature>ViewModel, <Feature>State> by hiltMavericksViewModelFactory()
}
```

## State 템플릿

```kotlin
data class <Feature>State(
  val data: Async<Data> = Uninitialized
) : MavericksState
```

## Screen 템플릿

```kotlin
@Composable
fun <Feature>Screen(
  viewModel: <Feature>ViewModel,
  navController: NavController
) {
  val state by viewModel.collectAsStateWithLifecycle()

  // UI 구현
}
```

## 네이밍 규칙

- Fragment: `<Feature>Fragment`
- ViewModel: `<Feature>ViewModel`
- State: `<Feature>State`
- Screen: `<Feature>Screen`
