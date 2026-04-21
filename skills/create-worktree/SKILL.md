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
4. .kiro 디렉토리 설정 (심볼릭 링크)

## 명령어

```bash
# 메인 worktree 경로 확인
MAIN_WORKTREE=$(git worktree list --porcelain | head -1 | sed 's/worktree //')
MAIN_KIRO="$MAIN_WORKTREE/.kiro"

# 기존 브랜치로 worktree 생성
git worktree add ../{worktree-name} {branch-name}

# 새 브랜치로 worktree 생성
git worktree add -b {new-branch-name} ../{worktree-name}

# .kiro 설정 심볼릭 링크 연결
cd ../{worktree-name}
mkdir -p .kiro
ln -sf "$MAIN_KIRO/agents" .kiro/agents
ln -sf "$MAIN_KIRO/steering" .kiro/steering
cp -r "$MAIN_KIRO/settings" .kiro/settings

# skills는 체크아웃 시 실제 디렉토리로 생성되므로 삭제 후 심볼릭 링크 생성
rm -rf .kiro/skills
ln -sf "$MAIN_KIRO/skills" .kiro/skills

# 심볼릭 링크로 대체된 tracked 파일들의 변경 감지 무시
git ls-files .kiro/skills/ | xargs -I{} git update-index --skip-worktree {}

# .kiro/skills 심볼릭 링크가 untracked로 표시되지 않도록 처리
echo ".kiro/skills" >> .gitignore
git update-index --skip-worktree .gitignore
```

## .kiro 설정 연결 규칙

- `agents/`, `skills/`, `steering/` → 심볼릭 링크 (메인과 공유, 변경 즉시 반영)
- `settings/` → 복사 (LSP 등 프로세스 충돌 방지)
- 심볼릭 링크로 대체된 `skills/` 하위 파일은 `skip-worktree` 플래그로 git 변경 감지 제외

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
   mkdir -p .kiro
   ln -sf "$MAIN_KIRO/agents" .kiro/agents
   ln -sf "$MAIN_KIRO/steering" .kiro/steering
   cp -r "$MAIN_KIRO/settings" .kiro/settings
   rm -rf .kiro/skills
   ln -sf "$MAIN_KIRO/skills" .kiro/skills
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

## 링크드 워크트리 브랜치 전환 시 주의

`.kiro/skills/`가 심볼릭 링크이므로, `git checkout`이 심볼릭 링크를 실제 디렉토리로 덮어쓸 수 있음.
브랜치 전환 시 아래 순서를 따를 것:

```bash
# 1. skip-worktree 해제
git ls-files .kiro/skills/ | xargs -I{} git update-index --no-skip-worktree {}

# 2. 변경 discard + 브랜치 전환
git checkout -- .kiro/skills/
git checkout {target-branch}

# 3. 심볼릭 링크 복원
MAIN_KIRO=$(git worktree list --porcelain | head -1 | sed 's/worktree //')/.kiro
rm -rf .kiro/skills
ln -sf "$MAIN_KIRO/skills" .kiro/skills

# 4. skip-worktree 재설정
git ls-files .kiro/skills/ | xargs -I{} git update-index --skip-worktree {}
```
