---
name: update-analytics-event
description: HTML 이벤트 스펙 문서 기반 Analytics Event 클래스 수정. "이벤트 로그 작업", "Analytics 이벤트 수정" 요청 시 사용.
---

# Update Analytics Event

HTML 테이블 형식의 이벤트 스펙 문서를 기반으로 `library/analytics` 모듈의 Event 클래스를 수정합니다.

## HTML 테이블 필드 의미

| 필드 | 설명 |
|------|------|
| Event Name | 이벤트 클래스의 `name` 필드 값 |
| Property Name | 이벤트 파라미터 (snake_case → camelCase 변환) |
| Data Type | `string` → `String`, `numeric` → `Int`, `list` → `List<String>` |
| 실시간적재(APP) | `PZ` 있으면 `PrizmAcceptable` 구현, 없으면 `DefaultAcceptable`만 |
| App | `app(webview)`면 웹뷰 이벤트 → 네이티브 코드 수정 불필요 |

## Status 필드 의미

| Status | 의미 | 작업 |
|--------|------|------|
| `delete` | 삭제 | property 제거 |
| `add` | 추가 | property 추가 |
| `modify` | 수정 | 기존 property 확인/수정 |

## Property Name 규칙

- `?` 붙은 property → nullable (`String?`, `Int?`)
- snake_case → camelCase 변환 (예: `goods_id` → `goodsId`)

## 패키지 구조

```
library/analytics/src/main/java/com/rxc/prizm/library/analytics/events/
├── BrandsListEvent.kt    # brands_list.* 이벤트
├── GoodsListEvent.kt     # goods_list.* 이벤트
├── DiscoverEvent.kt      # discover.* 이벤트
└── ...
```

## 코드 템플릿

### Event 클래스 (PrizmAcceptable 포함)
```kotlin
data class GoodsListTabSectionGoodsThumbnailEvent(
  val goodsId: String,
  val goodsName: String,
  val goodsIndex: Int,
  val sectionIndex: Int,
  val showroomId: String? = null,
  val discoverCategoryId: String? = null,  // 추가된 property
) : Event,
  DefaultAcceptable,
  PrizmAcceptable {  // 실시간적재(APP) = PZ
  override val name: String = "goods_list.tab_section_goods_thumbnail"
  override val parameters: Map<String, Any?> = mapOf(
    "goods_id" to goodsId,
    "goods_name" to goodsName,
    "goods_index" to goodsIndex,
    "section_index" to sectionIndex,
    "showroom_id" to showroomId,
    "discover_category_id" to discoverCategoryId,
  )
}
```

### Event 클래스 (DefaultAcceptable만)
```kotlin
data class GoodsListImpressionGoodsEvent(
  val goodsId: String,
  val goodsName: String,
  val goodsIndex: Int,
) : Event,
  DefaultAcceptable {  // 실시간적재(APP) = 비어있음
  override val name: String = "goods_list.impression_goods"
  override val parameters: Map<String, Any?> = mapOf(
    "goods_id" to goodsId,
    "goods_name" to goodsName,
    "goods_index" to goodsIndex,
  )
}
```

## 작업 순서

1. HTML 파일 읽기
2. Event Name으로 기존 클래스 찾기 (`grep` 사용)
3. 배경색 기반으로 property 추가/수정/삭제
4. `app(webview)` 이벤트는 스킵 (웹뷰에서 처리)

## 주의사항

- 새 property 추가 시 기본값 `= null` 설정 (기존 호출 코드 호환성)
- parameters map의 key는 snake_case 유지
- 필수 property는 기본값 없이, nullable property는 `= null`로 선언
