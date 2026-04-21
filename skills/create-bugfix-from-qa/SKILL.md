---
name: create-bugfix-from-qa
description: QA 티켓 기반 버그 수정 티켓 생성 및 브랜치 생성. "QA 버그", "PQ-XXXX 수정", "QA 티켓 작업" 요청 시 사용.
---

# QA 버그 수정 티켓 생성

QA 티켓(PQ-XXXX)을 기반으로 MO 보드에 버그 수정 티켓을 생성하고, 브랜치까지 만듭니다.

## 인증 정보

`.kiro/skills/credentials.md`의 JIRA 섹션 참조

## 워크플로우

1. **QA 티켓 조회**: 입력받은 PQ 티켓의 summary, epic 정보 조회
2. **MO 버그 티켓 생성**: QA 티켓 summary를 사용하여 MO 보드에 버그 티켓 생성
   - 업무 유형: 버그 (10004)
   - 담당자: 나 (`/rest/api/3/myself`)
   - 레이블: AOS
   - 스프린트: 현재 활성 스프린트
   - 에픽: QA 티켓에서 찾을 수 있으면 설정, 없으면 비워둠
3. **이슈 링크 생성**: MO 티켓 ↔ PQ 티켓 (relates to)
4. **상태 전환**: MO 티켓, PQ 티켓 모두 "진행 중"으로 변경
5. **티켓 링크 출력**: 생성된 MO 티켓의 JIRA 링크 출력
6. **베이스 브랜치 입력 요청**: 최신 release 브랜치를 기본값으로 제안
7. **feature 브랜치 생성**: `feature/{MO-XXXX}-{description}` 형식으로 로컬 브랜치 생성

## 티켓 생성 상세

### 제목 규칙
- QA 티켓의 summary를 그대로 사용
- `[AOS]` 말머리 추가: `[AOS] {PQ 티켓 summary}`

### 에픽 탐색
- QA 티켓의 parent 또는 epic link 필드 확인
- QA 에픽이 있으면 해당 에픽에 연결된 MO 에픽을 이슈 링크에서 탐색
- 찾지 못하면 에픽 비워둠

## API 호출

### QA 티켓 조회
```bash
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_URL/rest/api/3/issue/PQ-XXXX?fields=summary,parent,customfield_10014"
```

### 버그 티켓 생성
```bash
curl -s -X POST -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": {"key": "MO"},
      "summary": "[AOS] QA 티켓 제목",
      "issuetype": {"id": "10004"},
      "assignee": {"accountId": "<myself_accountId>"},
      "labels": ["AOS"],
      "customfield_10020": <sprintId>,
      "parent": {"key": "MO-XXXX"}
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

### 상태 전환 (진행 중)
```bash
# MO 티켓 (transition id: 21)
curl -s -X POST -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_URL/rest/api/3/issue/MO-XXXX/transitions" \
  -d '{"transition": {"id": "21"}}'

# PQ 티켓 (transition id: 61)
curl -s -X POST -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_URL/rest/api/3/issue/PQ-XXXX/transitions" \
  -d '{"transition": {"id": "61"}}'
```

### 최신 release 브랜치 조회
```bash
git ls-remote --heads origin 'release/*' | sed 's|.*refs/heads/||' | sort -t. -k1,1n -k2,2n -k3,3n | tail -1
```

### 브랜치 생성
```bash
git checkout -b feature/{MO-XXXX}-{description} origin/{base-branch} --no-track
```

## 출력 형식

```
✅ 버그 티켓 생성 완료
티켓: MO-XXXX
링크: https://rxc.atlassian.net/browse/MO-XXXX
연결: PQ-XXXX ↔ MO-XXXX (relates to)
상태: 둘 다 "진행 중"으로 변경

베이스 브랜치를 선택해주세요 (기본값: release/X.XX.X):
```

## 자동 실행

티켓 생성, 이슈 링크, 상태 전환은 사용자 확인 없이 자동 실행합니다.
베이스 브랜치만 사용자에게 입력받습니다.
