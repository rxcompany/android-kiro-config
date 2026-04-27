---
name: figma-component
description: 피그마 디자인가이드 기반 Compose 컴포넌트 생성. "피그마 컴포넌트", "디자인 구현", "피그마 → 코드" 요청 시 사용.
---

# Figma → Compose Component

피그마 디자인가이드에서 Compose 컴포넌트를 생성합니다.

## 토큰 매핑

디자인 토큰 변환 시 반드시 [design-token-mapping.md](./design-token-mapping.md)를 참조합니다.

## 규칙

### 색상
- 피그마 변수의 괄호 안 시맨틱 역할 → `PrizmDesignSystem.colorToken.*`
- 피그마 `Color/Neutral/*` → `PrizmDesignSystem.foundationColor.*`
- 하드코딩 `Color(0xFF...)` 금지 — 반드시 디자인 시스템 토큰 사용
- 피그마에서 변수 바인딩 없이 직접 hex 값만 있는 경우, 매핑 테이블에서 동일 hex 값의 토큰을 찾아 사용

### 타이포그래피
- 피그마 font size로 Typography 매핑: 10dp=micro, 12dp=mini, 14dp=small, 15dp=medium, 18dp=large, 20dp=title2, 24dp=title, 28dp=headline2, 32dp=headline
- Bold weight → `*Bold` 접미사

### Shape
- 피그마 corner radius → `PrizmDesignSystem.shapes.radius{N}` 또는 `.radiusFull`

### 이미지 / 비디오
- 이미지 표시: `PrizmImage` 사용. 파라미터는 domain `Image` 또는 `Media` 타입 (String URL 금지)
- 비디오 표시: `PrizmVideo` 사용. 파라미터는 domain `Media` 타입
- 사용 패턴은 [compose-migration SKILL.md](../compose-migration/SKILL.md)의 Before/After 패턴 참조

```kotlin
// 이미지
@Composable
fun MyComponent(image: Image) {
  PrizmImage(
    image = PrizmImage.Content(image = image, size = Image.Size.LARGE_1080),
    modifier = Modifier.fillMaxWidth(),
  )
}

// 비디오
@Composable
fun MyComponent(media: Media) {
  PrizmVideo(
    media = media,
    modifier = Modifier.fillMaxWidth(),
    isResume = true,
    size = Image.Size.LARGE_1080,
  )
}
```

### 컴포넌트 구조
- Stateless composable — 데이터는 파라미터, 인터랙션은 `on*` 람다 콜백
- `modifier: Modifier = Modifier`를 첫 번째 기본 파라미터로 (필수 파라미터 뒤)
- `PrizmDesignSystem` 테마로 감싸는 `@Preview` 포함 (Light + Dark)
- 리스트 파라미터는 `ImmutableList` 사용

### 기존 컴포넌트 재사용
- 디자인 라이브러리(`library.design.compose`)에 이미 존재하는 컴포넌트는 재사용
- GeneralButton, BubbleButton, SquircleButton → Compose 네이티브 버전 사용
- DataDisplayTitle, DataDisplayCardGoods 등 기존 PDS 컴포넌트 확인 후 활용
- TopBar, NavigationTabBar 등 네비게이션 컴포넌트 재사용

## 워크플로

### 1. 디자인가이드 링크 수집
- 사용자에게 피그마 디자인가이드 URL을 요청
- URL에서 nodeId 추출 (예: `node-id=1-2` → `1:2`)

### 2. 피그마 디자인 분석
- 피그마 MCP 도구를 활용하여 디자인 구조, 변수, 스크린샷 등을 파악
- 컴포넌트 계층 구조, 레이아웃, 간격, 색상, 타이포그래피 파악

### 3. 토큰 매핑
- 피그마 변수 → 코드 토큰 변환 (design-token-mapping.md 참조)
- 매핑 테이블에 없는 토큰 발견 시 사용자에게 확인

### 4. 기존 컴포넌트 탐색
- `library/design/src/main/java/com/rxc/prizm/library/design/compose/` 하위에서 재사용 가능한 컴포넌트 검색
- 동일하거나 유사한 컴포넌트가 있으면 재사용 또는 확장

### 5. 구현 계획 제안 → 사용자 승인
아래 항목을 결정하여 사용자에게 제시하고 승인을 받은 후 구현에 착수한다.

- **배치 모듈/패키지**: PDS 공통 컴포넌트 → `library/design/compose/{카테고리}/`, 피처 전용 → 해당 피처 모듈
- **파일명/클래스명**: 피그마 컴포넌트명 기반 PascalCase
- **파라미터 설계**: 필수 데이터, 옵션, 콜백 람다 목록
- **재사용할 기존 컴포넌트** 목록
- **구현 방향 요약**: 레이아웃 구조, 상태 관리 방식 등

### 6. 컴포넌트 구현
- Compose 코드 작성
- 디자인 토큰만 사용 (하드코딩 금지)
- Preview 함수 포함

### 7. 검증
- 피그마 스크린샷과 Preview 비교
- 다크모드 대응 확인 (토큰 사용 시 자동 대응)

## 예시

피그마에서 `Color/Text/black (text_primary)` + `15dp Bold` 텍스트를 발견한 경우:

```kotlin
Text(
  text = title,
  color = PrizmDesignSystem.colorToken.textPrimary,
  style = PrizmDesignSystem.typography.mediumBold,
)
```

피그마에서 `Color/Background/white_variant_1 (surface)` 배경 + `radius_12` 모서리를 발견한 경우:

```kotlin
Box(
  modifier = Modifier
    .background(
      color = PrizmDesignSystem.colorToken.backgroundSurface,
      shape = PrizmDesignSystem.shapes.radius12,
    )
)
```
