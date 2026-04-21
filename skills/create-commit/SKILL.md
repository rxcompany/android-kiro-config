---
name: create-commit
description: JIRA 이슈 기반 커밋 생성. "커밋", "커밋 생성", "변경사항 커밋" 요청 시 사용.
---

# Commit 생성

JIRA 이슈 제목을 기반으로 컨벤션에 맞는 커밋을 생성합니다.

## 인증 정보

`.kiro/skills/credentials.md`의 JIRA 섹션 참조

## 커밋 메시지 형식

```
[ISSUE-KEY] <subject>
<body>
```

**중요**: 제목과 본문 사이에 빈줄을 넣지 않습니다.

## 워크플로우

1. **JIRA 이슈 조회**: JIRA REST API(`curl`)로 이슈 정보 가져오기
2. **제목 생성**: JIRA summary를 그대로 사용 (플랫폼 접두사 제거)
3. **커밋 메시지 확인**: 작성한 커밋 메시지를 사용자에게 보여주고 승인을 받는다
4. **커밋 실행**: 승인 후 `git commit`

## Subject 규칙

- JIRA 이슈 키를 대괄호로 감싸서 시작: `[MO-1234]`
- 이슈 키 뒤 공백 한 칸 후 JIRA summary 사용
- **JIRA 티켓 제목에 플랫폼 말머리(예: `[AOS]`, `[iOS]`, `[Server]`)가 포함된 경우 반드시 제거** (예: `[AOS] 홈 피드 API 변경` → `홈 피드 API 변경`)
- **하나의 JIRA 이슈 브랜치에서 모든 커밋의 제목은 `[이슈키] JIRA 티켓 제목`으로 통일**
- 커밋별 세부 작업 내용은 본문에 작성
- 한글로 작성
- 마침표 없이 작성

## Body 규칙 (선택)

- 상세 설명이 필요한 경우 작성
- 피드백 반영, 리팩토링 내용 등을 bullet point로 작성
- 들여쓰기: 1단계 공백 1개 + `-`, 2단계 공백 3개 + `-`

### Body 작성 요령
- **맥락 중심 서술**: 무엇을 했는지(what)뿐 아니라 왜 필요한지(why)를 함께 작성
  - ✗ `variant별 height() 함수 추가`
  - ✓ `variant별 height() 함수 추가: Large(60/80dp), Mini(40dp) 등 variant마다 다른 높이 스펙 대응`
- **새로 추가된 항목은 자명한 속성을 나열하지 않음**: 신규 항목의 기본 구성요소는 생략하고 핵심 특성만 기술
  - ✗ `Mini variant 추가: mini 타이포그래피, 고정 높이 40dp, isBold 파라미터 추가`
  - ✓ `Mini variant 추가: mini 타이포그래피, 고정 높이 40dp` (isBold는 Mini의 기본 구성이므로 생략)
- **파라미터 추가보다 역할을 서술**: 파라미터명이 아닌 그 파라미터가 가능하게 하는 기능을 작성
  - ✗ `Small variant에 isBold 파라미터 추가`
  - ✓ `Small variant에 타이틀 bold 여부 조정 기능 추가`

## 예시

### 기본 커밋
```
[MO-5047] 다중 아이템 스냅 캐러셀 구현
```

### 본문 포함 커밋
```
[MO-5089] 라이브 아웃링크 컴포넌트 디자인 변경
 - LiveOutLinkLogo 디자인 스펙 변경
   - 배경색을 반투명 흰색에서 불투명 흰색으로 변경
   - 프로필 이미지 우측 하단에 AD 배지 추가
```

### 피드백 반영
```
[MO-4743] ScheduleShortcut Compose 전환
 - (피드백 반영)
   - 라이프 사이클 관련 사이드 이펙트 간소화
   - remember key 추가
```

## 특수 케이스

### 배포 커밋
- 형식: `[ISSUE-KEY] v{versionName}({versionCode}) 배포`
- 예: `[MO-5113] v1.75.0(292) 배포`
- 본문 없이 제목만 작성
