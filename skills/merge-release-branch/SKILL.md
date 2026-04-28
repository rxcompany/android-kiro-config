---
name: merge-release
description: 릴리즈 머지 및 Slack 알림. "릴리즈 머지", "release 머지", "develop/master 머지", "릴리즈 브랜치 머지" 요청 시 사용.
---

# Release 브랜치 머지

release 브랜치를 develop, master에 머지하는 PR을 생성하고 dev_mobile 채널에 PR 링크를 전달합니다.

## Release 브랜치 탐색

1. 리모트에서 `release/*` 브랜치 조회
2. 1개면 바로 진행
3. 복수개면 사용자에게 선택 요청
4. 0개면 사용자에게 알림 후 종료

## 워크플로우

### 1. PR 생성

1. release 브랜치 최신화
2. release → develop PR 생성
3. release → master PR 생성

### 2. Slack 알림

1. 두 PR 링크를 dev_mobile 채널에 전달

## 명령어

```bash
# 리모트 release 브랜치 조회
git fetch --prune
git branch -r | grep 'origin/release/' | sed 's|origin/||' | sed 's/^[[:space:]]*//'

# release 브랜치 최신화
git checkout release/{versionName} && git pull

# 현재 GitHub 사용자 조회
gh api user --jq '.login'

# develop 머지 PR 생성
gh pr create \
  --title "release/{versionName} → develop" \
  --base develop \
  --head release/{versionName} \
  --assignee "$(gh api user --jq '.login')" \
  --label "merge"

# master 머지 PR 생성
gh pr create \
  --title "release/{versionName} → master" \
  --base master \
  --head release/{versionName} \
  --assignee "$(gh api user --jq '.login')" \
  --label "merge"
```

## Slack 메시지

dev_mobile 채널(C020ZQLBRFG)에 Slack MCP의 `conversations_add_message` 도구로 전송합니다.

```
{develop PR 링크}
{master PR 링크}
```

## 예시

```
입력: "릴리즈 브랜치 머지해"

1. git fetch --prune
2. git branch -r | grep 'origin/release/'
   → origin/release/1.75.0

3. release 브랜치 1개 발견 → release/1.75.0으로 진행

4. git checkout release/1.75.0 && git pull

5. gh pr create --title "release/1.75.0 → develop" --base develop --head release/1.75.0 --assignee "$(gh api user --jq '.login')" --label "merge"
   → PR #1234 생성

6. gh pr create --title "release/1.75.0 → master" --base master --head release/1.75.0 --assignee "$(gh api user --jq '.login')" --label "merge"
   → PR #1235 생성

7. dev_mobile 채널(C020ZQLBRFG)에 Slack 메시지 전송:
   https://github.com/ARC-MARKET/prizm-android/pull/1234
   https://github.com/ARC-MARKET/prizm-android/pull/1235
```

## 주의사항

- 각 PR은 사용자가 직접 머지 (자동 머지하지 않음)
- develop, master PR 간 순서 의존성 없음
- Slack 메시지 전송에는 Slack MCP 서버의 `conversations_add_message` 도구 필요 (미설정 시 Slack 알림 스킵)
