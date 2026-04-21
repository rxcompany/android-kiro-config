---
name: create-jira-issue
description: JIRA 이슈 생성. "이슈 생성", "지라 생성", "티켓 생성" 요청 시 사용.
---

# JIRA 이슈 생성

MO 프로젝트에 JIRA 이슈를 생성합니다.

## 인증 정보

`.kiro/skills/credentials.md`의 JIRA 섹션에서 URL, Email, API Token을 읽어 사용합니다.

## 기본값 (항상 적용)

- **제목**: `[AOS] ` 말머리 + 사용자 입력 제목
- **레이블**: `AOS`
- **담당자**: 현재 인증된 사용자 (`/rest/api/3/myself`로 accountId 조회)
- **스프린트**: 현재 활성 스프린트 (Board ID: 1). 사용자가 "백로그"를 명시하면 스프린트 미지정

## 옵셔널 항목 (사용자에게 확인)

아래 항목은 사용자가 명시하지 않은 경우 **되물어서 확인**합니다. 사용자가 "없음", "스킵" 등으로 답하면 생략합니다.

1. **업무 유형**: 작업(10002), 스토리(10001), 버그(10004) 중 선택 (기본: 스토리)
2. **상위 이슈**: Epic 또는 상위 이슈 키 (예: MO-5081)
3. **수정 버전(Fix Version)**: 릴리즈 버전 (예: 1.77.0)
4. **연결된 업무 항목**: 이슈 키와 링크 타입
   - relates to (기본), blocks, is blocked by, duplicates, clones
   - 지정하지 않으면 항상 relates to로 연결
5. **스토리 포인트**: 숫자 값 (필드: `customfield_10026`)

## 본문 (description) 작성 규칙

티켓 본문에는 다음 내용을 포함한다:
- **참조 링크**: 관련 Notion 문서, Slack 쓰레드, Figma 디자인 가이드 링크
- **실행/구현 계획 요약**: 해당 티켓의 작업 내용을 요약하여 기술

## 워크플로우

1. 사용자에게 옵셔널 항목 확인
2. 현재 인증된 사용자의 accountId 조회 (`/rest/api/3/myself`)
3. 현재 활성 스프린트 ID 조회
4. JIRA REST API로 이슈 생성
5. 연결된 업무 항목이 있으면 이슈 링크 생성

## API 호출

### 내 계정 조회
```bash
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_URL/rest/api/3/myself" | jq -r '.accountId'
```

### 스프린트 조회
```bash
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_URL/rest/agile/1.0/board/1/sprint?state=active"
```

### 이슈 생성
```bash
curl -s -X POST -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": {"key": "MO"},
      "summary": "[AOS] 제목",
      "issuetype": {"id": "10002"},
      "assignee": {"accountId": "<myself_accountId>"},
      "labels": ["AOS"],
      "customfield_10020": <sprintId>,
      "parent": {"key": "MO-XXXX"},
      "fixVersions": [{"name": "1.77.0"}]
    }
  }'
```

### 이슈 링크 생성
```bash
curl -s -X POST -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_URL/rest/api/3/issueLink" \
  -d '{
    "type": {"name": "Relates"},
    "inwardIssue": {"key": "MO-XXXX"},
    "outwardIssue": {"key": "PQ-XXXX"}
  }'
```

## 브랜치 생성 시 상태 전환

이슈 생성 후 연결된 브랜치를 함께 생성하는 경우, JIRA 이슈 상태를 "진행 중"으로 전환합니다.

```bash
curl -s -X POST -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_URL/rest/api/3/issue/MO-XXXX/transitions" \
  -d '{"transition": {"id": "21"}}'
```
