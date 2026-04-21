---
name: create-usecase
description: Clean Architecture UseCase 생성. "유스케이스 생성", "UseCase 작성" 요청 시 사용.
---

# UseCase 생성

Clean Architecture 패턴의 UseCase를 생성합니다.

## 패키지 구조

```
domain/usecases/<feature>/
└── <Action><Entity>UseCase.kt
```

## 네이밍 규칙

- 형식: `<Action><Entity>UseCase`
- 예: `GetGoodsListUseCase`, `SearchKeywordUseCase`, `DeleteBookmarkUseCase`

## UseCase 템플릿

```kotlin
class Get<Entity>UseCase @Inject constructor(
  private val repository: <Feature>Repository
) {
  suspend operator fun invoke(param: Param): Result<Entity> =
    runCatching { repository.get<Entity>(param) }
}
```

## 패턴별 예시

### 단순 조회
```kotlin
class GetGoodsDetailUseCase @Inject constructor(
  private val goodsRepository: GoodsRepository
) {
  suspend operator fun invoke(goodsId: Long): Result<GoodsDetail> =
    runCatching { goodsRepository.getGoodsDetail(goodsId) }
}
```

### 파라미터 객체 사용
```kotlin
class GetGoodsListUseCase @Inject constructor(
  private val goodsRepository: GoodsRepository
) {
  suspend operator fun invoke(params: Params): Result<List<Goods>> =
    runCatching { goodsRepository.getGoodsList(params.categoryId, params.sort) }

  data class Params(
    val categoryId: Long,
    val sort: GoodsSort
  )
}
```

### Flow 반환
```kotlin
class ObserveUserUseCase @Inject constructor(
  private val userRepository: UserRepository
) {
  operator fun invoke(): Flow<User> = userRepository.observeUser()
}
```

## ViewModel에서 사용

```kotlin
class MyViewModel @AssistedInject constructor(
  @Assisted initialState: MyState,
  private val getGoodsDetail: GetGoodsDetailUseCase
) : MavericksViewModel<MyState>(initialState) {

  fun loadGoods(goodsId: Long) {
    suspend { getGoodsDetail(goodsId).getOrThrow() }
      .execute { copy(goods = it) }
  }
}
```
