---
name: deploy-release
description: "릴리즈 배포", "배포 브랜치 생성", "versionCode 업데이트 커밋, PR 생성까지 수행", "버전 업데이트 PR", "배포 브랜치", "릴리즈 배포" 요청 시 사용.
---

# Deployment Branch 생성

release 브랜치로부터 배포용 feature 브랜치를 생성하고, versionName/versionCode를 업데이트합니다.

## 필수 정보 수집

1. **Release 브랜치**: "어느 release 브랜치로부터 생성하시겠습니까?" (예: release/1.75.0)
2. **JIRA 이슈**: "어떤 JIRA 이슈 번호를 사용하시겠습니까?" (예: MO-5113)

## 워크플로우

1. 사용자에게 release 브랜치 확인
2. 사용자에게 JIRA 이슈 번호 확인
3. release 브랜치로 체크아웃 및 최신화 (`git checkout release/{versionName} && git pull`)
4. `common-config.gradle`에서 현재 versionCode 조회
5. 브랜치 생성: `feature/{ISSUE-KEY}-dist-{versionCode+1}`
5. `common-config.gradle`에서 versionCode +1, versionName을 release 브랜치 버전으로 업데이트
6. 변경사항 커밋 및 푸시
7. "PR도 생성할까요?" 질문 → 승인 시 아래 설정으로 Draft PR 생성
   - **Base**: release 브랜치
   - **Assignee**: GitHub API로 현재 사용자 조회
   - **Milestone**: release 브랜치 버전과 동일한 이름의 마일스톤 자동 매칭 (예: release/1.75.0 → 마일스톤 `1.75.0`)
   - **Label**: `build/configuration`

## 버전 파일

```
common-config.gradle
```

```groovy
ext.versions = [
  versionCode: 291,
  versionName: "1.75.0"
]
```

## 브랜치 명명 규칙

- 형식: `feature/{ISSUE-KEY}-dist-{versionCode+1}`
- versionCode는 현재 값 + 1
- 예: 현재 290이면 → `feature/MO-5113-dist-291`

## 명령어

```bash
# release 브랜치 체크아웃 및 최신화
git checkout release/{versionName} && git pull

# versionCode 조회
grep -E "versionCode|versionName" common-config.gradle

# 배포 브랜치 생성
git checkout -b feature/{ISSUE-KEY}-dist-{newVersionCode} release/{versionName}

# common-config.gradle 수정 후 커밋
git add common-config.gradle
git commit -m "[{ISSUE-KEY}] v{versionName}({newVersionCode}) 배포"

# 푸시
git push -u origin feature/{ISSUE-KEY}-dist-{newVersionCode}

# 마일스톤 번호 조회 (버전명으로 매칭)
gh api repos/:owner/:repo/milestones --jq '.[] | select(.title == "{versionName}") | .number'

# PR 생성
gh pr create \
  --title "[{ISSUE-KEY}] v{versionName}({newVersionCode}) 배포" \
  --body "## 배경\n- {ISSUE-KEY}" \
  --base release/{versionName} \
  --assignee "$(gh api user --jq '.login')" \
  --milestone "{versionName}" \
  --label "build/configuration" \
  --draft
```

## 예시

```
입력: "배포 브랜치 생성해"

1. 질문: "어느 release 브랜치로부터 생성하시겠습니까?"
   → 응답: "release/1.75.0"

2. 질문: "어떤 JIRA 이슈 번호를 사용하시겠습니까?"
   → 응답: "MO-5113"

3. git checkout release/1.75.0 && git pull

4. versionCode 조회 (예: 290)

5. git checkout -b feature/MO-5113-dist-291 release/1.75.0
6. common-config.gradle 수정: versionCode=291, versionName="1.75.0"
7. git commit -m "[MO-5113] v1.75.0(291) 배포"
8. git push -u origin feature/MO-5113-dist-291

9. 질문: "PR도 생성할까요?"
   → 승인 시:
   gh pr create \
     --title "[MO-5113] v1.75.0(291) 배포" \
     --body "## 배경\n- MO-5113" \
     --base release/1.75.0 \
     --assignee "$(gh api user --jq '.login')" \
     --milestone "1.75.0" \
     --label "build/configuration" \
     --draft
```

## 커밋 메시지 형식

- 형식: `[{ISSUE-KEY}] v{version}({newVersionCode}) 배포`
- 예: `[MO-5113] v1.75.0(291) 배포`

## PR 본문 형식

```markdown
## 배경
- {ISSUE-KEY}
```

## 주의사항

- 반드시 release 브랜치와 JIRA 이슈 번호를 사용자에게 확인
- versionName/versionCode 업데이트는 배포 브랜치에서만 수행
- versionCode는 자동으로 조회하여 +1 적용
