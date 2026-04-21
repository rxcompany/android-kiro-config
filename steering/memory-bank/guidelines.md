# Development Guidelines

## Code Quality Standards

### Package Structure
- Follow reverse domain naming: `com.rxc.prizm.<module>.<feature>`
- Group by feature/functionality, not by type
- Keep domain entities in `domain.entities` package
- Place interfaces in `domain.interfaces` package

### Naming Conventions
- **Classes**: PascalCase (e.g., `ServiceWebViewFragment`, `LiveViewModel`)
- **Functions**: camelCase (e.g., `updateLive`, `connectChat`)
- **Properties**: camelCase (e.g., `liveId`, `isVisible`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `BOTTOM_NAVIGATION_HEIGHT_DP`)
- **Resources**: snake_case (e.g., `ic_send_filled`, `fragment_service_webview`)

### Code Organization
- Nested and inner classes are positioned immediately above the companion object.
- Companion objects at the end of class definitions
- Override public functions are positioned first among public functions.
- Group related functions together
- Private functions after public functions (including override functions)
- Extension functions in separate files with `Ext` suffix

## Architecture Patterns

The codebase implements a **Model-View-Intent (MVI) architecture** using the Mavericks framework, combined with **Jetpack Compose** for declarative UI and **Clean Architecture** principles for separation of concerns. This pattern enforces unidirectional data flow and immutable state management across three distinct layers.

### Component Responsibilities

**ViewModel Layer (e.g., GoodsListViewModel)**
- Extends `MavericksViewModel<StateClass extends MavericksState>` to manage immutable application state
- Orchestrates business logic through domain use cases (e.g., `GetGoodsListFromSectionUseCase`, `GetSearchedGoodsListUseCase`)
- Processes user intents via public functions (e.g., `setCurrentSort()`, `fetchGoodsList()`, `fetchLoadMore()`)
- Updates state immutably using `setState { copy(...) }` pattern
- Handles asynchronous operations with `suspend` functions and `.execute()` reducer pattern
- Manages pagination, filtering, and sorting state transformations
- Uses `@AssistedInject` for dependency injection with initial state parameter

```kotlin
class GoodsListViewModel @AssistedInject constructor(
  @Assisted goodsListState: GoodsListState,
  private val getGoodsList: GetGoodsListFromSectionUseCase
) : MavericksViewModel<GoodsListState>(goodsListState) {
  
  fun setCurrentSort(goodsSort: GoodsSort) = withState { state ->
    setState { copy(currentSortIndex = state.sortList.indexOf(goodsSort)) }
    withState { fetchGoodsList() }
  }
}
```

**Fragment Layer (e.g., GoodsListFragment)**
- Extends `ComposeFragment` and implements `MavericksView` for lifecycle-aware state observation
- Acts as the Android framework integration point and navigation host
- Obtains ViewModel instance via `fragmentViewModel()` delegate
- Manages Compose lifecycle with `@Composable Content()` function
- Handles fragment-specific concerns (transitions, bottom navigation reselection)
- Delegates all UI rendering to the Screen composable
- Implements `invalidate()` for Mavericks state subscription (typically empty when using Compose)

```kotlin
@AndroidEntryPoint
class GoodsListFragment : ComposeFragment(), MavericksView {
  private val viewModel: GoodsListViewModel by fragmentViewModel()
  
  @Composable
  override fun Content() {
    PrizmDesignSystem {
      GoodsListScreen(
        goodsListViewModel = viewModel,
        navController = findNavController()
      )
    }
  }
  
  override fun invalidate() = Unit
}
```

**Screen Layer (e.g., GoodsListScreen)**
- Pure Composable function responsible for declarative UI rendering
- Observes ViewModel state via `collectAsStateWithLifecycle()` for lifecycle-aware collection
- Renders UI based on current state without maintaining local business state
- Dispatches user intents back to ViewModel (e.g., `goodsListViewModel.fetchGoodsList()`)
- Handles UI-specific concerns (scroll state, animations, layout composition)
- Uses `LaunchedEffect` for side effects tied to composition lifecycle
- Implements responsive UI patterns (loading, error, empty states)

```kotlin
@Composable
fun GoodsListScreen(
  goodsListViewModel: GoodsListViewModel,
  navController: NavController
) {
  val feeds by goodsListViewModel.collectAsStateWithLifecycle(GoodsListState::goodsFeedList)
  val initialRequest by goodsListViewModel.collectAsStateWithLifecycle(GoodsListState::initialRequest)
  
  when {
    initialRequest is Fail -> Error(onClickButton = { goodsListViewModel.fetchGoodsList() })
    else -> LazyVerticalGrid { items(feeds) { /* render item */ } }
  }
}
```

### Data Flow Pattern

**Unidirectional Data Flow:**
1. **User Interaction** → Screen captures user events (clicks, scrolls, selections)
2. **Intent Dispatch** → Screen invokes ViewModel functions to express user intent
3. **State Update** → ViewModel processes intent, executes business logic, and updates state immutably
4. **State Emission** → Mavericks emits new state through reactive streams
5. **UI Recomposition** → Screen observes state changes and recomposes affected UI elements

**State Observation:**
- Screen uses `collectAsStateWithLifecycle()` to observe specific state properties
- Mavericks automatically handles lifecycle-aware collection and cancellation
- State updates trigger selective recomposition only for affected composables
- `withState { }` provides synchronous state access within ViewModel for decision-making

**Async Operations:**
- ViewModel wraps suspend functions with `.execute { }` for automatic state management
- Async results are typed as `Async<T>` (Uninitialized, Loading, Success, Fail)
- Reducer functions transform async results into new state immutably
- Screen renders different UI based on async state (loading indicators, error views)

This architecture ensures clear separation of concerns: Fragment handles Android lifecycle, ViewModel manages business logic and state, and Screen provides pure declarative UI rendering with unidirectional data flow.

### Dependency Injection with Hilt
- Use `@AndroidEntryPoint` for Android components
- Use `@Inject` for constructor injection
- Use `@AssistedInject` for ViewModels with initial state
- Factory interfaces with `@AssistedFactory`
- Use `@EntryPoint` or `*ScreenFactory` pattern for dependency injection in Composables

```kotlin
@AndroidEntryPoint
class ServiceWebViewFragment : Fragment() {
  @Inject lateinit var userStore: UserStore
  @Inject lateinit var eventTracker: EventTracker
  
  private val viewModel: ServiceWebViewModel by fragmentViewModel()
}
```

**Dependency Injection in Composables:**

```kotlin
// Use EntryPoint for accessing dependencies in Composables
@EntryPoint
@InstallIn(SingletonComponent::class)
interface MyScreenEntryPoint {
  fun eventTracker(): EventTracker
  fun userStore(): UserStore
}

@Composable
fun MyScreen() {
  val context = LocalContext.current
  val entryPoint = remember {
    EntryPointAccessors.fromApplication(
      context.applicationContext,
      MyScreenEntryPoint::class.java
    )
  }
  val eventTracker = remember { entryPoint.eventTracker() }
  val userStore = remember { entryPoint.userStore() }
  
  // Use dependencies
}

// Or use ScreenFactory pattern
class MyScreenFactory @Inject constructor(
  private val eventTracker: EventTracker,
  private val userStore: UserStore
) {
  @Composable
  fun Content(
    viewModel: MyViewModel,
    navController: NavController
  ) {
    MyScreen(
      viewModel = viewModel,
      navController = navController,
      eventTracker = eventTracker,
      userStore = userStore
    )
  }
}

@Composable
fun MyScreen(
  viewModel: MyViewModel,
  navController: NavController,
  eventTracker: EventTracker,
  userStore: UserStore
) {
  // Use injected dependencies
}
```

## Kotlin Best Practices

### Type Declaration
- Type can be omitted only in these cases:
  - Variables (var, val) with Primitive types or immediate class constructor assignment
  - Single Expression Functions returning Primitive types or immediate class constructor calls

```kotlin
// ✓ Type omission allowed
val count = 5                    // Primitive
val name = "John"                // Primitive
val user = User("John", 25)      // Class constructor
fun createUser() = User("John")  // Single expression with constructor

// ✗ Type must be explicit
val result: String = processData()     // Function call
val items: List<String> = getItems()   // Generic type
fun getData(): ApiResponse = repository.fetch()  // Complex return type
```

### Single Expression Function
- Single line: Expression appears immediately after "="
- Multiple lines: Expression appears after line break following "="

```kotlin
// ✓ Correct - Single line
fun createUser() = User("John", 25)
fun getName() = user.name

// ✓ Correct - Multiple lines
fun calculateTotal() =
    items.sumOf { it.price }

fun processData() =
    data.filter { it.isValid }
        .map { it.transform() }

// ✗ Incorrect
fun calculateTotal() = items.sumOf { it.price } // Single line but too long, needs line break
fun createUser() =
    User("John", 25) // Single line but unnecessary line break
```

### Code Formatting (ktlint)
- **Code Style**: Android Studio
- **Max Line Length**: 200 characters
- **Indent Size**: 2 spaces
- **End of Line**: LF (Unix style)
- **Charset**: UTF-8
- **Trailing Whitespace**: Trimmed
- **Final Newline**: Required

**Disabled Rules:**
- Import ordering (custom import layout used)
- Trailing comma on call/declaration sites
- Filename conventions
- Spacing between declarations with annotations
- Function signature formatting
- Function naming conventions

**Custom Rules:**
- Class signature: Force multiline when parameter count ≥ 2
- Function signature: Force multiline when parameter count ≥ 3 (ktlint enforced)

### Function Parameter & Argument Wrapping
- 파라미터/인자가 3개 이상이면 줄바꿈하여 1줄에 1개씩 작성한다.
- 함수 **정의**(파라미터)는 ktlint `function-signature` 룰로 자동 적용된다.
- 함수 **호출**(인자)은 ktlint로 강제되지 않으므로 코드 리뷰에서 확인한다.

```kotlin
// ✓ Correct - 함수 정의 (ktlint enforced)
fun createOrder(
  userId: String,
  goodsId: Long,
  quantity: Int,
) { ... }

// ✓ Correct - 함수 호출 (NOT enforced by ktlint, review required)
createOrder(
  userId = "user123",
  goodsId = 456L,
  quantity = 2,
)

// ✗ Incorrect
fun createOrder(userId: String, goodsId: Long, quantity: Int) { ... }
createOrder(userId = "user123", goodsId = 456L, quantity = 2)
```

### Guard Clause Formatting
- `?: return`, `?: continue`, `?: break` 등 가드문 뒤에는 빈줄을 추가한다.

```kotlin
// ✓ Correct
val userId = userInfo.userId ?: return

AppsFlyerLib.getInstance()
  .setCustomerUserId(userId)

// ✗ Incorrect
val userId = userInfo.userId ?: return
AppsFlyerLib.getInstance()
  .setCustomerUserId(userId)
```

### Chaining Call Wrapping
- 함수 체이닝 호출은 줄바꿈하여 작성한다.

```kotlin
// ✓ Correct
recyclerView.children
  .forEach { child -> ... }

activeIds.toList()
  .forEach { ExoPlayerPool.release(it) }

Handler(Looper.getMainLooper())
  .postDelayed({ prepare() }, 500)

// ✗ Incorrect
recyclerView.children.forEach { child -> ... }
activeIds.toList().forEach { ExoPlayerPool.release(it) }
Handler(Looper.getMainLooper()).postDelayed({ prepare() }, 500)
```

### Import Guidelines
- Always add import statements and use short type names
- Never use Fully Qualified Names directly in code
- Example: Import `HomeLiveBanner` instead of using `com.rxc.prizm.domain.entities.HomeLiveBanner`
- Use aliases when type names conflict with file names or other imports
- Example: `import android.graphics.Rect as AndroidRect`

```kotlin
// ✓ Correct - use imports
import com.rxc.prizm.domain.entities.HomeLiveBanner
import android.graphics.Rect as AndroidRect

val banner: HomeLiveBanner = getBanner()
val bounds: AndroidRect = getBounds()

// ✗ Incorrect - don't use fully qualified names
val banner: com.rxc.prizm.domain.entities.HomeLiveBanner = getBanner()
```

### Immutable Collections
- Use `kotlinx.collections.immutable` for Compose state
- Convert with `.toImmutableList()` before passing to Composables
- Use `persistentListOf()` for static lists

```kotlin
val menus by viewModel.collectAsStateWithLifecycle { it.topBarMenus.toImmutableList() }
```

## UI Development

### Compose Patterns
- Wrap Compose UI with `PrizmDesignSystem` theme
- Composable components must accept `modifier: Modifier = Modifier` as first default parameter (after required parameters)
- Use `LaunchedEffect` for side effects
- Use `LifecycleStartEffect`, `LifecycleResumeEffect` for handling lifecycle events,
- Use `remember` for state that survives recomposition
- Use `derivedStateOf` for computed state
- Specify `key` parameter in LazyColumn/LazyRow items for performance optimization

```kotlin
@Composable
fun MyComposable() {
  var impression by remember { mutableStateOf(false) }
  
  if (impression) {
    LaunchedEffect(Unit) {
      eventTracker.log(event)
    }
  }
  
  // Handle lifecycle events
  LifecycleResumeEffect(lifecycleOwner) {
    // add ON_RESUME effect here

    onPauseOrDispose {
      // add clean up for work kicked off in the ON_RESUME effect here
    }
  }
  
  Column(
    modifier = Modifier
      .padding(top = 12.dp)
      .onLayoutRectChanged { bounds ->
        impression =  bounds.fractionVisibleInWindow() > 0f
      }
  ) {
    // Content
  }
}

// LazyList with key parameter
LazyColumn {
  items(items, key = { it.id }) { item ->
    ItemCard(item)
  }
}
```

### Design System Tokens
- 색상은 하드코딩(`Color(0xFF...)`)하지 않고 디자인 시스템 토큰을 사용한다.
- Feature 모듈에서는 `PrizmDesignSystem` 객체를 통해 접근한다.
- Design 라이브러리 내부 컴포넌트에서는 `Local*` CompositionLocal을 직접 사용한다.

```kotlin
// ✓ Feature 모듈에서 사용
@Composable
fun MyScreen() {
  Text(
    text = "Hello",
    color = PrizmDesignSystem.foundationColor.neutralBlack,
    style = PrizmDesignSystem.typography.body1,
  )
  Box(
    modifier = Modifier.background(PrizmDesignSystem.colorToken.backgroundSurface)
  )
}

// ✓ Design 라이브러리 내부에서 사용
@Composable
internal fun DesignComponent() {
  val color = LocalFoundationColor.current
  val token = LocalColorToken.current
  Text(
    text = "Hello",
    color = color.neutralBlack,
  )
}

// ✗ 하드코딩 금지 (디버그/개발 전용 UI 제외)
Text(color = Color(0xFF000000))
Box(modifier = Modifier.background(Color.White))
```

사용 가능한 토큰:
- `PrizmDesignSystem.foundationColor` — 기본 색상 (neutralWhite, neutralBlack, neutralGray*, etcRed 등)
- `PrizmDesignSystem.colorToken` — 시맨틱 토큰 (backgroundSurface, brandTint, statePressed* 등)
- `PrizmDesignSystem.typography` — 텍스트 스타일
- `PrizmDesignSystem.shapes` — Shape 토큰

### Composable Function Structure
- **Screen Composable**: ViewModel을 직접 파라미터로 받아 상태를 수집하고 UI를 렌더링한다.
- **Component Composable**: 순수 데이터를 파라미터로 받는 Stateless 컴포넌트. ViewModel 의존 없음.
- Screen에서 Component를 호출할 때 ViewModel 상태를 풀어서 전달한다.

```kotlin
// Screen: ViewModel을 받아 상태 수집
@Composable
fun GoodsListScreen(
  goodsListViewModel: GoodsListViewModel,
  navController: NavController,
) {
  val feeds by goodsListViewModel.collectAsStateWithLifecycle(GoodsListState::goodsFeedList)

  GoodsGrid(
    feeds = feeds,
    onClickItem = { navController.navigate(...) },
  )
}

// Component: 순수 데이터만 받음
@Composable
fun GoodsGrid(
  feeds: List<GoodsFeed>,
  onClickItem: (GoodsFeed) -> Unit,
  modifier: Modifier = Modifier,
) {
  LazyVerticalGrid(modifier = modifier) {
    items(feeds, key = { it.id }) { feed ->
      GoodsCard(feed, onClick = { onClickItem(feed) })
    }
  }
}
```

### ComposeView in ViewBinding Fragments
- ViewBinding Fragment 안에서 ComposeView를 사용할 때 `ViewCompositionStrategy`를 설정한다.
- Fragment의 View 라이프사이클에 맞춰 `DisposeOnLifecycleDestroyed(viewLifecycleOwner)`를 사용한다.

```kotlin
// ✓ ViewBinding Fragment에서 ComposeView 사용
viewBinding.composeView.setViewCompositionStrategy(
  ViewCompositionStrategy.DisposeOnLifecycleDestroyed(viewLifecycleOwner)
)
viewBinding.composeView.setContent {
  PrizmDesignSystem {
    MyComposable()
  }
}
```

### Mavericks + Compose State Collection
- Mavericks ViewModel 상태 수집: `com.airbnb.mvrx.compose.collectAsStateWithLifecycle`
- 일반 Flow 수집: `androidx.lifecycle.compose.collectAsStateWithLifecycle`
- 두 import가 공존할 수 있으므로 혼동 주의

```kotlin
// Mavericks ViewModel 상태 수집 (State의 특정 프로퍼티)
import com.airbnb.mvrx.compose.collectAsStateWithLifecycle

val feeds by viewModel.collectAsStateWithLifecycle(GoodsListState::goodsFeedList)
val live by liveViewModel.collectAsStateWithLifecycle(LiveState::live)

// 일반 StateFlow 수집
import androidx.lifecycle.compose.collectAsStateWithLifecycle

val debugInfo by debugCollector.stateFlow.collectAsStateWithLifecycle()
```

### Side Effects
- `LaunchedEffect(Unit)` — Composition 진입 시 1회 실행 (일회성 초기화, impression 로깅)
- `LaunchedEffect(key)` — key 변경 시마다 재실행 (상태 변화에 반응)
- `DisposableEffect` — 정리(cleanup)가 필요한 리소스 (lifecycle observer 등록/해제)
- `snapshotFlow { }` — Compose 상태를 Flow로 변환하여 `LaunchedEffect` 내에서 collect

```kotlin
// 일회성 실행
LaunchedEffect(Unit) {
  viewModel.fetchInitialData()
}

// 상태 변화에 반응
LaunchedEffect(selectedTab) {
  viewModel.loadTab(selectedTab)
}

// 정리가 필요한 리소스
DisposableEffect(lifecycleOwner) {
  val observer = LifecycleEventObserver { _, event -> ... }
  lifecycleOwner.lifecycle.addObserver(observer)
  onDispose { lifecycleOwner.lifecycle.removeObserver(observer) }
}

// Compose 상태 → Flow 변환
LaunchedEffect(pagerState) {
  snapshotFlow { pagerState.currentPage }
    .collect { page -> viewModel.onPageChanged(page) }
}
```

### Compose Preview
- Preview 함수 네이밍: `Preview` + 컴포넌트명 + 상태 (e.g., `PreviewBubbleItemLight`, `PreviewBubbleItemDark`)
- 반드시 `PrizmDesignSystem { }` 으로 감싼다.
- 다크모드 Preview: `@Preview(uiMode = Configuration.UI_MODE_NIGHT_YES)` 추가
- `Image` 도메인 모델이 필요하면 `PreviewImage` 사용 (`com.rxc.prizm.library.design.compose.common.Common.kt`)

```kotlin
@Preview
@Composable
fun PreviewMyComponentLight() {
  PrizmDesignSystem {
    MyComponent(title = "Sample", isActive = true)
  }
}

@Preview(uiMode = Configuration.UI_MODE_NIGHT_YES)
@Composable
fun PreviewMyComponentDark() {
  PrizmDesignSystem {
    MyComponent(title = "Sample", isActive = true)
  }
}
```

### Custom Views
- Extend appropriate base class (ConstraintLayout, etc.)
- Use `@JvmOverloads` for constructors
- Inflate layout in init block
- Expose properties with getters/setters
- Use backing fields when needed

```kotlin
class ChatMessageInputLayout @JvmOverloads constructor(
  context: Context,
  attrs: AttributeSet? = null,
  defStyleAttr: Int = 0
) : ConstraintLayout(context, attrs, defStyleAttr) {
  
  private val binding = LayoutChatMessageInputBinding.inflate(
    LayoutInflater.from(context), this
  )
  
  var messageText: String
    get() = binding.inputText.text?.toString() ?: ""
    set(value) { binding.inputText.setText(value) }
}
```

## Analytics & Tracking

### Event Tracking
- Use `EventTracker` interface for logging
- Create event classes in `library.analytics.events`
- Log impressions with `LaunchedEffect` in Compose
- Track user interactions immediately on click

```kotlin
eventTracker.log(
  DiscoverTabSectionGoodsThumbnail(
    goodsId = goods.id.toString(),
    goodsName = goods.name,
    goodsIndex = index + 1,
    sectionId = section.id.toString()
  )
)
```

### Impression Tracking
- Use `onLayoutRectChanged` or `onVisibilityChanged` modifier
- `onLayoutRectChanged` provides `RelativeLayoutBounds` when layout position changes
- `onVisibilityChanged` provides boolean visibility state
- Use `remember` to track impression state
- Log once per impression with `LaunchedEffect`

## Navigation

### Fragment Navigation
- Use Navigation Component with `findNavController()`
- Pass arguments with `.asMavericksArgs()`
- Use `navOptions` for navigation behavior
- Handle navigation results with `getNavigationResult`

```kotlin
findNavController().navigate(
  R.id.action_global_goodsFragment,
  ServiceWebType.Goods(code).asMavericksArgs(),
  navOptions { launchSingleTop = true }
)
```

### Deep Links
- Handle deep links in `shouldOverrideUrlLoading`
- Use `launch(uri)` extension for deep link navigation
- Check scheme with `uri.scheme == Constants.PRIZM_DEEPLINK_SCHEME`

## Error Handling
- Check `isSafe()` before UI operations in callbacks
- Use `runCatching` for exception-prone operations
- Handle `Async` states: `Uninitialized`, `Loading`, `Success`, `Fail`

```kotlin
if (isSafe()) {
  viewBinding.errorView.isVisible = true
}

runCatching {
  submitData()
}.onFailure { error ->
  eventTracker.issueTrackingLog("Error: $error")
}
```
