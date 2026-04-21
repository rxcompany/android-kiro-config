---
name: create-feature-branch
description: JIRA 이슈 기반 feature 브랜치 생성. "브랜치 생성", "MO-XXXX 브랜치", "이슈 브랜치" 요청 시 사용.
---

# Feature Branch 생성

JIRA 이슈 기반으로 feature 브랜치를 생성하고 JIRA 상태를 변경합니다.

## 인증 정보

`.kiro/skills/credentials.md`의 JIRA 섹션 참조

## 워크플로우

1. **JIRA 이슈 조회**: JIRA REST API(`curl`)로 이슈 정보 가져오기
2. **브랜치 생성**: `git checkout -b` (현재 브랜치 기준, 로컬만 생성, 원격 푸시하지 않음)
3. **JIRA 상태 변경**: JIRA REST API(`curl`)로 "진행 중" 상태로 변경

## JIRA 전환 ID

| ID | 상태 |
|----|------|
| 11 | 해야 할 일 |
| 21 | 진행 중 |
| 31 | 완료 |
| 41 | 리뷰 중 |

## 브랜치 명명 규칙

- 형식: `feature/{ISSUE-KEY}-{description}`
- Epic 이슈: `project/{ISSUE-KEY}-{description}` 사용
- description은 JIRA 이슈 제목(summary)에서 추출
- `[AOS]`, `[iOS]` 등 플랫폼 접두사 제거
- 영문 소문자, 하이픈으로 구분

## 명령어

```bash
# 브랜치 생성 (로컬만)
git checkout -b feature/MO-XXXX-description
```

## 예시

```
입력: "MO-5047 브랜치 생성해"

1. curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" "$JIRA_URL/rest/api/3/issue/MO-5047"
   → summary: "[AOS] 다중 아이템 스냅 캐러셀 구현"

2. git checkout -b feature/MO-5047-multi-item-snap-carousel

3. curl -s -X POST -u "$JIRA_EMAIL:$JIRA_API_TOKEN" -H "Content-Type: application/json" \
     "$JIRA_URL/rest/api/3/issue/MO-5047/transitions" -d '{"transition":{"id":"21"}}'
```

## 자동 실행

모든 단계는 사용자 확인 없이 자동으로 실행합니다.
