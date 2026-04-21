---
name: list-sprint-tasks
description: 현재 스프린트의 내 할일 목록 조회. "할일 목록", "내 이슈", "스프린트 이슈", "지라 할일" 요청 시 사용.
---

# 스프린트 할일 목록 조회

MO 보드의 현재 활성 스프린트에서 담당자가 나인 미완료 이슈를 조회합니다.

## 워크플로우

1. **활성 스프린트 조회**: MO 보드(boardId=1)의 활성 스프린트 ID 가져오기
2. **이슈 조회**: 해당 스프린트에서 담당자가 나이고 완료가 아닌 이슈 조회
3. **결과 표시**: 상태별로 그룹핑하여 테이블로 표시

## API

```bash
# 인증 정보
# .kiro/skills/credentials.md의 JIRA 섹션 참조

# 1. 활성 스프린트 조회
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_URL/rest/agile/1.0/board/1/sprint?state=active"

# 2. 스프린트 이슈 조회
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_URL/rest/agile/1.0/board/1/sprint/{sprintId}/issue" -G \
  --data-urlencode "jql=assignee=currentUser() AND status != Done ORDER BY status ASC, created DESC" \
  --data-urlencode "fields=summary,status,issuetype,priority" \
  --data-urlencode "maxResults=50"
```

## 출력 형식

스프린트 이름과 기간을 표시하고, 상태별로 그룹핑하여 테이블로 출력합니다.

```
Sprint #123 (2/13 ~ 2/27) 기준

### 진행 중
| 이슈 | 유형 | 제목 |
|------|------|------|
| MO-XXXX | 스토리 | 제목 |

### 해야 할 일
| 이슈 | 유형 | 제목 |
|------|------|------|
| MO-XXXX | 스토리 | 제목 |
```

## 자동 실행

모든 단계는 사용자 확인 없이 자동으로 실행합니다.
