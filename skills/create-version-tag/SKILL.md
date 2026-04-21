---
name: create-version-tag
description: 버전 태그 생성. "태그 생성", "버전 태그", "release 태그" 요청 시 사용.
---

# Version Tag 생성

release 브랜치가 master에 머지된 후 버전 태그를 생성합니다.

## 태그 형식

```
v{major}.{minor}.{patch}
```

예: `v1.75.0`, `v1.76.0`

## 워크플로우

1. master 브랜치에서 release 머지 커밋 찾기
2. 해당 커밋에 태그 생성
3. 원격에 태그 푸시

## 명령어

```bash
# 1. release 머지 커밋 찾기
git log remotes/origin/master --oneline | grep -i "release/{versionName}"

# 2. 태그 생성
git tag v{versionName} {commit-hash}

# 3. 원격에 푸시
git push origin v{versionName}
```

## 예시

```
입력: "1.75.0 태그 생성해"

1. git log remotes/origin/master --oneline | grep -i "release/1.75"
   → 9a62e67e8 Merge pull request #4421 from rxcompany/release/1.75.0

2. git tag v1.75.0 9a62e67e8

3. git push origin v1.75.0
```

## 주의사항

- release 브랜치가 master에 머지된 후에만 실행
- 태그명은 `v` + versionName 형식
- 머지 커밋 해시를 정확히 찾아야 함
