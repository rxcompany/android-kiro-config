---
name: prepare-release
description: 릴리즈 준비. "릴리즈 준비", "버전 올려", "release 브랜치" 요청 시 사용.
---

# Release 준비

release 브랜치를 생성합니다. 버전 업데이트는 하지 않습니다.

## 필수 정보 수집

1. **새 버전명**: "새 versionName을 입력해주세요" (예: 1.76.0)

## 워크플로우

1. develop 브랜치에서 release 브랜치 생성
2. 리모트에 푸시

## 명령어

```bash
# develop 최신화
git checkout develop && git pull

# release 브랜치 생성
git checkout -b release/{versionName} develop

# 푸시
git push -u origin release/{versionName}
```

## 예시

```
입력: "1.76.0 릴리즈 준비해"

1. git checkout -b release/1.76.0 develop
2. git push -u origin release/1.76.0
```

## 주의사항

- develop 브랜치에서 시작
- versionName/versionCode는 수정하지 않음 (배포 브랜치에서만 업데이트)
