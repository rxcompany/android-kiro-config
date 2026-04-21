---
name: debug-crashlytics
description: Firebase Crashlytics 이슈 분석. "크래시 분석", "Crashlytics", "오류 분석" 요청 시 사용.
---

# Crashlytics 디버깅

Firebase Crashlytics 이슈를 분석하고 원인을 파악합니다.

## 워크플로우

1. **이슈 조회**: `crashlytics_get_issue`로 이슈 정보 가져오기
2. **이벤트 조회**: `crashlytics_list_events`로 샘플 크래시 가져오기
3. **스택트레이스 분석**: 코드베이스에서 관련 코드 찾기
4. **원인 분석 및 수정 제안**

## MCP 도구

```
crashlytics_get_issue(appId, issueId)
crashlytics_list_events(appId, filter: { issueId })
crashlytics_batch_get_events(appId, names)
```

## 분석 순서

### 1. 이슈 개요 파악
- 영향받은 사용자 수
- 발생 빈도
- 영향받은 버전

### 2. 스택트레이스 분석
- 크래시 발생 위치 (파일, 라인)
- 호출 스택 추적
- 예외 타입 및 메시지

### 3. 코드 분석
- 해당 코드 파일 조회
- 관련 컨텍스트 파악
- null safety, lifecycle 이슈 확인

### 4. 수정 제안
- 근본 원인 설명
- 수정 코드 제안
- 테스트 방법 안내

## 일반적인 크래시 원인

| 타입 | 원인 | 해결 |
|------|------|------|
| NPE | null 체크 누락 | `?.`, `?:`, `requireNotNull` |
| IllegalStateException | 잘못된 lifecycle | `isSafe()`, `viewLifecycleOwner` |
| IndexOutOfBounds | 리스트 범위 초과 | `getOrNull`, 범위 체크 |
| ConcurrentModification | 동시 수정 | `toList()`, 동기화 |

## 예시

```
입력: "이 크래시 분석해줘" + issueId

1. crashlytics_get_issue(appId, issueId)
2. crashlytics_list_events(appId, { issueId })
3. 스택트레이스에서 파일/라인 추출
4. 코드베이스에서 해당 코드 조회
5. 원인 분석 및 수정안 제시
```
