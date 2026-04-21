---
name: create-viewmodel-test
description: ViewModel 테스트 작성. "ViewModel 테스트", "테스트 작성" 요청 시 사용.
---

# ViewModel Test 생성

Kotest + MockK 기반 ViewModel 테스트를 작성합니다.

## 테스트 파일 위치

```
<module>/src/test/java/com/rxc/prizm/<feature>/
└── <Feature>ViewModelTest.kt
```

## 테스트 템플릿

```kotlin
class <Feature>ViewModelTest : BehaviorSpec({
  val useCase = mockk<<UseCase>>()
  
  lateinit var viewModel: <Feature>ViewModel

  beforeTest {
    viewModel = <Feature>ViewModel(
      initialState = <Feature>State(),
      useCase = useCase
    )
  }

  Given("초기 상태") {
    When("데이터 로드 요청") {
      coEvery { useCase() } returns Result.success(mockData)
      
      viewModel.loadData()

      Then("데이터가 로드됨") {
        viewModel.awaitState().data shouldBe Success(mockData)
      }
    }
  }
})
```

## MockK 패턴

### suspend 함수 모킹
```kotlin
coEvery { repository.getData() } returns data
coVerify { repository.getData() }
```

### Flow 모킹
```kotlin
every { repository.observeData() } returns flowOf(data)
```

### 예외 모킹
```kotlin
coEvery { useCase() } returns Result.failure(Exception("error"))
```

## Mavericks 상태 검증

```kotlin
// 상태 대기 후 검증
viewModel.awaitState().data shouldBe Success(expected)

// 특정 상태까지 대기
viewModel.awaitState { it.data is Success }
```

## 의존성

```kotlin
// build.gradle
testImplementation(libs.kotest.runner.junit5)
testImplementation(libs.kotest.assertions.core)
testImplementation(libs.mockk)
testImplementation(libs.kotlinx.coroutines.test)
```
