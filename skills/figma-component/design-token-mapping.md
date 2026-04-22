# Figma ↔ Code Design Token Mapping

피그마 변수명과 코드 토큰 간의 매핑 테이블.
피그마 변수는 `Color/{카테고리}/{이름} ({시맨틱 역할})` 형태로 명명됨.

## Color — Foundation (FoundationColor)

| Figma Variable | Compose (PrizmDesignSystem.foundationColor.*) | XML Resource |
|---|---|---|
| `Color/Neutral/white` | `.neutralWhite` | `@color/neutral_white` |
| `Color/Neutral/gray3` | `.neutralGray3` | `@color/neutral_gray3` |
| `Color/Neutral/gray8` | `.neutralGray8` | `@color/neutral_gray8` |
| `Color/Neutral/gray20` | `.neutralGray20` | `@color/neutral_gray20` |
| `Color/Neutral/gray50` | `.neutralGray50` | `@color/neutral_gray50` |
| `Color/Neutral/gray70` | `.neutralGray70` | `@color/neutral_gray70` |
| `Color/Neutral/black` | `.neutralBlack` | `@color/neutral_black` |
| `Color/Neutral/white_variant_1` | `.neutralWhiteVariant1` | `@color/neutral_white_variant_1` |
| `Color/Neutral/white_variant_2` | `.neutralWhiteVariant2` | `@color/neutral_white_variant_2` |
| `Color/Neutral/gray_bg` | `.neutralGrayBg` | `@color/neutral_gray_bg` |
| `Color/Neutral/gray8_filled` | `.neutralGray8Filled` | `@color/neutral_gray8_filled` |
| `Color/Neutral/white20` | `.neutralWhite20` | `@color/neutral_white_20` |
| `Color/Neutral/white70` | `.neutralWhite70` | `@color/neutral_white_70` |
| `Color/Neutral/white50` | `.neutralWhite50` | `@color/neutral_white_50` |
| `Color/Neutral/fixed/white_light` | `.neutralFixedWhiteLight` | `@color/neutral_fixed_white_light` |
| `Color/Neutral/fixed/gray8_light` | `.neutralFixedGray8Light` | `@color/neutral_fixed_gray8_light` |
| `Color/Neutral/fixed/gray20_light` | `.neutralFixedGray20Light` | `@color/neutral_fixed_gray20_light` |
| `Color/Neutral/fixed/gray20_dark` | `.neutralFixedGray20Dark` | `@color/neutral_fixed_gray20_dark` |
| `Color/Neutral/fixed/gray50_light` | `.neutralFixedGray50Light` | `@color/neutral_fixed_gray50_light` |
| `Color/Neutral/fixed/gray50_dark` | `.neutralFixedGray50Dark` | `@color/neutral_fixed_gray50_dark` |
| `Color/Neutral/fixed/gray70_light` | `.neutralFixedGray70Light` | `@color/neutral_fixed_gray70_light` |
| `Color/Neutral/fixed/gray70_dark` | `.neutralFixedGray70Dark` | `@color/neutral_fixed_gray70_dark` |
| `Color/Neutral/fixed/black_light` | `.neutralFixedBlackLight` | `@color/neutral_fixed_black_light` |
| `Color/Neutral/fixed/white_variant_2_dark` | `.neutralFixedWhiteVariant2Dark` | `@color/neutral_fixed_white_variant_2_dark` |
| `Color/Etc/red` | `.etcRed` | `@color/etc_red` |
| `Color/Etc/green` | `.etcGreen` | `@color/etc_green` |
| `Color/Etc/neon_green` | `.etcNeonGreen` | `@color/etc_neon_green` |

## Color — Semantic Token (ColorToken)

피그마에서 괄호 안의 시맨틱 역할이 코드 토큰명에 대응됨.
예: `Color/Text/black (text_primary)` → `colorToken.textPrimary`

| Figma Variable | Compose (PrizmDesignSystem.colorToken.*) | XML Resource |
|---|---|---|
| `Color/Brand/black (tint)` | `.brandTint` | `@color/token_brand_tint` |
| `Color/Brand/gray3 (tint3)` | `.brandTint3` | `@color/token_brand_tint3` |
| `Color/Background/white_variant_1 (surface)` | `.backgroundSurface` | `@color/token_background_surface` |
| `Color/Background/white_variant_2 (surface_high)` | `.backgroundSurfaceHigh` | `@color/token_background_surface_high` |
| `Color/Background/gray_bg (bg)` | `.backgroundBg` | `@color/token_background_bg` |
| `Color/Background/layout/gray_bg (section)` | `.backgroundLayoutSection` | `@color/token_background_layout_section` |
| `Color/Background/layout/gray8 (line)` | `.backgroundLayoutLine` | `@color/token_background_layout_line` |
| `Color/States/gray8_filled (disabled_bg)` | `.stateDisabledBg` | `@color/token_state_disabled_bg` |
| `Color/States/gray8_filled (seleted_bg)` | `.stateSelectedBg` | `@color/token_state_selected_bg` |
| `Color/States/white70 (disabled_media)` | `.stateDisabledMedia` | `@color/token_state_disabled_media` |
| `Color/States/gray8 (pressed_media)` | `.statePressedMedia` | `@color/token_state_pressed_media` |
| `Color/States/gray20_r (pressed_primary)` | `.statePressedPrimary` | `@color/token_state_pressed_primary` |
| `Color/States/gray3 (pressed_cell)` | `.statePressedCell` | `@color/token_state_pressed_cell` |
| `Color/Text/black (text_primary)` | `.textPrimary` | `@color/token_text_primary` |
| `Color/Text/gray70 (text_secondary)` | `.textSecondary` | `@color/token_text_secondary` |
| `Color/Text/gray50 (text_tertiary)` | `.textTertiary` | `@color/token_text_tertiary` |
| `Color/Text/tint (text_link)` | `.textLink` | `@color/token_text_link` |
| `Color/Text/gray50 (text_helper)` | `.textHelper` | `@color/token_text_helper` |
| `Color/Text/gray20 (text_placeholder)` | `.textPlaceholder` | `@color/token_text_placeholder` |
| `Color/Text/gray20 (text_disabled)` | `.textDisabled` | `@color/token_text_disabled` |
| `Color/Semantic/red (error)` | `.semanticError` | `@color/token_semantic_error` |
| `Color/Semantic/red (sale)` | `.semanticSale` | `@color/token_semantic_sale` |
| `Color/Semantic/red (live)` | `.semanticLive` | `@color/token_semantic_live` |
| `Color/Semantic/red (noti)` | `.semanticNoti` | `@color/token_semantic_noti` |
| `Color/Semantic/red (like)` | `.semanticLike` | `@color/token_semantic_like` |
| `Color/Semantic/red (secondary)` | `.semanticRedSecondary` | `@color/token_semantic_red_secondary` |
| `Color/Semantic/red (disabled)` | `.semanticRedDisabled` | `@color/token_semantic_red_disabled` |
| *(코드 전용)* | `.iconDisabled` | `@color/token_icon_disabled` |

## Typography

| Figma Style | Compose (PrizmDesignSystem.typography.*) | XML Style | Size |
|---|---|---|---|
| Micro / Regular | `.micro` | `@style/Micro` | 10dp |
| Micro / Bold | `.microBold` | `@style/Micro.Bold` | 10dp |
| Mini / Regular | `.mini` | `@style/Mini` | 12dp |
| Mini / Bold | `.miniBold` | `@style/Mini.Bold` | 12dp |
| Small / Regular | `.small` | `@style/Small` | 14dp |
| Small / Bold | `.smallBold` | `@style/Small.Bold` | 14dp |
| Medium / Regular | `.medium` | `@style/Medium` | 15dp |
| Medium / Bold | `.mediumBold` | `@style/Medium.Bold` | 15dp |
| Large / Regular | `.large` | `@style/Large` | 18dp |
| Large / Bold | `.largeBold` | `@style/Large.Bold` | 18dp |
| Title2 / Regular | `.title2` | `@style/Title2` | 20dp |
| Title2 / Bold | `.title2Bold` | `@style/Title2.Bold` | 20dp |
| Title / Regular | `.title` | `@style/Title` | 24dp |
| Title / Bold | `.titleBold` | `@style/Title.Bold` | 24dp |
| Headline2 / Regular | `.headline2` | `@style/Headline2` | 28dp |
| Headline2 / Bold | `.headline2Bold` | `@style/Headline2.Bold` | 28dp |
| Headline / Regular | `.headline` | `@style/Headline` | 32dp |
| Headline / Bold | `.headlineBold` | `@style/Headline.Bold` | 32dp |

StrikeThrough 변형: `.microStrikeThrough`, `.miniStrikeThrough`, `.mediumStrikeThrough` (Compose 전용)

## Shapes

| Figma Variable | Compose (PrizmDesignSystem.shapes.*) |
|---|---|
| `Radius/radius_6` (또는 6px corner) | `.radius6` |
| `Radius/radius_12` (또는 12px corner) | `.radius12` |
| 2px corner | `.radius2` |
| 4px corner | `.radius4` |
| 8px corner | `.radius8` |
| Full / Capsule | `.radiusFull` |

## Spacing

| Figma Variable | Code |
|---|---|
| `Spacing/spacing_8` | `8.dp` |
| `Spacing/spacing_16` | `16.dp` |

> Spacing은 별도 토큰 없이 `Dp` 리터럴로 직접 사용.

## 변환 규칙 요약

1. 피그마 괄호 안 시맨틱 역할 → `colorToken.*` (camelCase)
2. 피그마 `Color/Neutral/*` → `foundationColor.neutral*` (camelCase)
3. 피그마 `Color/Etc/*` → `foundationColor.etc*` (camelCase)
4. 피그마 `Color/Neutral/fixed/*` → `foundationColor.neutralFixed*` (camelCase)
5. 피그마 font size → Typography 이름 매핑: 10dp=micro, 12dp=mini, 14dp=small, 15dp=medium, 18dp=large, 20dp=title2, 24dp=title, 28dp=headline2, 32dp=headline
6. Bold weight → `*Bold` 접미사
7. Corner radius → `shapes.radius{N}` 또는 `shapes.radiusFull`
