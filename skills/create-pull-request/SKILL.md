---
name: create-pull-request
description: GitHub Pull Request 생성. "PR 생성", "PR 만들어", "풀리퀘스트" 요청 시 사용.
---

# Pull Request 생성

GitHub CLI(`gh`)를 사용하여 JIRA 이슈 기반 PR을 생성합니다.

## 필수 정보 수집 (순차적)

1. **타겟 브랜치 선택**: JIRA REST API로 티켓의 상위 에픽을 조회하여 에픽 브랜치가 있으면 `develop`과 함께 제안, 없으면 `develop` 제안. 사용자에게 선택 요청
2. **마일스톤 선택**: `gh api` 조회 후 사용자에게 선택 요청
3. **라벨 선택**: `gh label list` 조회 후 사용자에게 선택 요청

**중요**: 각 항목을 순차적으로 질문합니다. 동시에 질문하지 않습니다.

## 워크플로우

1. 마일스톤 조회: `gh api repos/:owner/:repo/milestones --jq '.[] | "\(.title) (\(.open_issues) open)"'`
2. 번호와 함께 마일스톤 목록 표시
3. 사용자 선택 대기
4. 라벨 조회: `gh label list`
5. 번호와 함께 라벨 목록 표시
6. 사용자 선택 대기
7. JIRA REST API(`curl`)로 이슈 정보 조회
8. `gh pr create` 실행

## GitHub CLI 명령어

```bash
# 마일스톤 조회
gh api repos/:owner/:repo/milestones --jq '.[] | "\(.title)"'

# 라벨 조회
gh label list

# 현재 GitHub 사용자 조회
gh api user --jq '.login'

# PR 생성
gh pr create \
  --title "[MO-XXXX] 제목" \
  --body "본문" \
  --base develop \
  --assignee "$(gh api user --jq '.login')" \
  --milestone "Sprint XX" \
  --label "label1,label2" \
  --draft
```

## PR 제목

- JIRA 이슈 summary 그대로 사용
- 형식: `[ISSUE-KEY] <JIRA summary>`

## PR 본문 형식

```markdown
## 배경
- MO-XXXX
- MO-YYYY (연결된 이슈가 있는 경우)
- 노션 문서, 슬랙 쓰레드, 피그마 디자인 가이드 등 참조 링크
- 필요 시 부연 설명 (PR 복잡도에 맞게 판단, 자명한 경우 생략)

## 작업 내용
- 코드 변경의 핵심만 기술, 자명한 항목은 생략
- 코드 요소(클래스명, 파일명, 설정값 등)는 backtick(`)으로 감싸기

## 참고
- 리뷰어가 알아야 할 추가 정보
```

## 기본 설정

- **Assignee**: 현재 GitHub 로그인 사용자 (`gh api user --jq '.login'`)
- **Draft**: true (기본값, 명시적 요청 시 false)
- **Base**: 사용자에게 질문 (제안: `develop` 또는 JIRA 상위 에픽 티켓의 브랜치)

## 예시

```bash
gh pr create \
  --title "[MO-5047] 다중 아이템 스냅 캐러셀 구현" \
  --body "## 배경
- MO-5047

## 작업 내용
- 다중 아이템 스냅 캐러셀 구현" \
  --base develop \
  --assignee "$(gh api user --jq '.login')" \
  --milestone "Sprint 25" \
  --label "feature,android" \
  --draft
```
