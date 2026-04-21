---
name: create-worktree
description: Git worktree 생성 및 .kiro 설정 연결. "워크트리 생성", "worktree 생성", "병렬 작업 환경" 요청 시 사용.
---

# Git Worktree 생성

메인 저장소에서 Git worktree를 생성하고 .kiro 설정을 심볼릭 링크로 연결합니다.

## 필수 정보 수집

1. **worktree 이름**: "worktree 이름을 지정해주세요" (예: prizm-wt1, prizm-agent-1)
2. **브랜치**: "체크아웃할 브랜치를 지정해주세요. 새 브랜치를 생성할까요?" (예: feature/MO-5001-xxx 또는 기존 브랜치)

## 워크플로우

1. 사용자에게 worktree 이름과 브랜치 확인
2. 메인 worktree 경로 확인 (`git worktree list`에서 첫 번째 항목)
3. worktree 생성 (메인과 같은 레벨의 sibling 디렉토리)
4. .kiro 심볼릭 링크 연결
5. .idea/vcs.xml에 .kiro VCS root 추가

## 명령어

```bash
# 메인 worktree 경로 확인
MAIN_WORKTREE=$(git worktree list --porcelain | head -1 | sed 's/worktree //')

# 기존 브랜치로 worktree 생성
git worktree add ../{worktree-name} {branch-name}

# 새 브랜치로 worktree 생성
git worktree add -b {new-branch-name} ../{worktree-name}

# .kiro 전체를 심볼릭 링크로 연결
cd ../{worktree-name}
ln -sf "$MAIN_WORKTREE/.kiro" .kiro

# .idea/vcs.xml에 .kiro VCS root 추가
sed -i '' 's|<mapping directory="" vcs="Git" />|<mapping directory="" vcs="Git" />\n    <mapping directory="$PROJECT_DIR$/.kiro" vcs="Git" />|' .idea/vcs.xml
```

## .kiro 설정 연결 규칙

- `.kiro` 전체를 메인 worktree의 심볼릭 링크로 연결 (단일 링크)
- `.kiro/`는 프로젝트 `.gitignore`에 포함되어 있으므로 `skip-worktree` 불필요
- `.idea/vcs.xml`에 `.kiro` VCS root를 추가하여 IDE에서 변경사항 확인 가능

## 예시

```
입력: "워크트리 생성해줘"

1. 질문: "worktree 이름을 지정해주세요"
   → 응답: "prizm-wt1"

2. 질문: "체크아웃할 브랜치를 지정해주세요. 새 브랜치를 생성할까요?"
   → 응답: "feature/MO-5001-new-feature"

3. MAIN_WORKTREE=/Users/develog/StudioProjects/prizm-android

4. git worktree add ../prizm-wt1 feature/MO-5001-new-feature

5. cd ../prizm-wt1
   ln -sf "$MAIN_WORKTREE/.kiro" .kiro

6. sed -i '' 's|<mapping directory="" vcs="Git" />|...|' .idea/vcs.xml
```

## 관리 명령어 안내

worktree 생성 완료 후 아래 안내를 출력합니다:

```
✅ worktree 생성 완료: ../{worktree-name} ({branch-name})

사용법:
  cd ../{worktree-name} && kiro-cli chat

관리:
  git worktree list              # 목록 확인
  git worktree remove ../{name}  # 삭제
```

## 주의사항

- 같은 브랜치를 두 worktree에서 동시에 체크아웃할 수 없음
- worktree 디렉토리는 메인과 같은 레벨(sibling)에 생성
- 반드시 worktree 이름과 브랜치를 사용자에게 확인 후 실행
