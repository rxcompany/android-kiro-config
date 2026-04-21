---
name: api-integration
description: Swagger 기반 API 연동. "API 연동", "Service 작성", "Repository 작성" 요청 시 사용.
---

# API Integration

Swagger 문서 기반으로 Service, Repository, Entity를 생성합니다.

## Swagger 문서 조회

```bash
curl -s "https://api-dev.prizm.co.kr/v3/api-docs/Commerce%20API"
```

## 패키지 구조

| 타입 | 경로 |
|------|------|
| Service | `library/network/services/` |
| Response DTO | `library/network/response/<feature>/` |
| Request Body | `library/network/body/<feature>/` |
| Repository Interface | `domain/interfaces/` |
| Repository Impl | `library/network/domain/` |
| Domain Entity | `domain/entities/<feature>/` |

## 네이밍 규칙

- **Service**: `<Feature>Service` (예: `SearchService`)
- **Response**: `<Name>Resp` (예: `SearchKeywordRecentlyResp`)
- **Request Body**: `<Name>Body` (예: `SearchKeywordBody`)
- **Repository**: `<Feature>Repository` / `<Feature>RepositoryImpl`

## 코드 템플릿

### Response DTO
```kotlin
@JsonClass(generateAdapter = true)
data class ExampleResp(
  @Json(name = "id")
  val id: String,
  @Json(name = "name")
  val name: String
) {
  companion object {
    fun ExampleResp.asEntity(): Example = Example(id, name)
  }
}
```

### Service
```kotlin
interface ExampleService {
  @GET("v1/example/{id}")
  suspend fun getExample(@Path("id") id: Long): ExampleResp

  @POST("v1/example")
  suspend fun createExample(@Body body: ExampleBody): ExampleResp
}
```

## 도메인 엔티티 규칙

- 도메인 엔티티는 API 스펙 값을 직접 가지지 않음
- API 스펙 매핑은 Repository 구현체에서 처리

```kotlin
// ✓ 도메인 enum
enum class DiscoverGoodsType { POPULAR, RECENTLY }

// ✓ Repository에서 매핑
override suspend fun getDiscoverGoods(type: DiscoverGoodsType) {
  val typeValue = when (type) {
    DiscoverGoodsType.POPULAR -> "popular"
    DiscoverGoodsType.RECENTLY -> "recently"
  }
  return service.getDiscoverGoods(typeValue)
}
```
