---
name: trigger-release-build
description: "Bitrise 릴리즈 빌드 트리거", "빌드 시작", "빌드 돌려", "배포 빌드", "원격 빌드", "릴리즈 빌드" 요청 시 사용.
---

# Bitrise 빌드 트리거

브랜치와 워크플로우를 지정하여 Bitrise 빌드를 시작합니다.

## 워크플로우

1. **파라미터 확인**: 브랜치, 워크플로우 확인 (미지정 시 사용자에게 질문)
2. **빌드 트리거**: Bitrise API 호출
3. **결과 출력**: 빌드 번호, URL 표시

## 파라미터

- `branch`: 빌드할 브랜치 이름 (필수)
- `workflow`: 워크플로우 (선택, 기본값: `inhouseRelease`)
  - `inhouseRelease`: QA 빌드
  - `productRelease`: RC 빌드

## 명령어

```bash
curl -s -X POST "https://app.bitrise.io/app/53316146d9037fb2/build/start.json" \
  -H "Content-Type: application/json" \
  -d '{
    "hook_info": {
      "type": "bitrise",
      "api_token": "<BITRISE_API_TOKEN from .kiro/skills/credentials.md>"
    },
    "build_params": {
      "branch": "{branch}",
      "workflow_id": "{workflow}"
    }
  }'
```

## 결과 처리

- 성공 시: 브랜치, 워크플로우, 빌드 번호, 빌드 URL 출력
- 실패 시: 에러 메시지 출력

## 예시

```
입력: "feature/MO-5047-something 브랜치 inhouse 워크플로우로 빌드해"

결과:
✅ 빌드 시작!
브랜치: feature/MO-5047-something
워크플로우: inhouse
빌드 번호: 1234
빌드 URL: https://app.bitrise.io/build/xxxxx
```

## 실행 전 확인

빌드 요청 시 반드시 사용자에게 다음을 질문한 후 실행합니다:
1. **브랜치**: 아래 명령어로 최신 release 브랜치를 조회하여 기본값으로 제안. **현재 체크아웃된 브랜치가 아닌 release 브랜치를 사용해야 한다.**
2. **빌드 유형**: `inhouseRelease`(QA) 또는 `productRelease`(RC)

### 최신 release 브랜치 조회

```bash
git ls-remote --heads origin 'release/*' | sed 's|.*refs/heads/||' | sort -t. -k1,1n -k2,2n -k3,3n | tail -1
```
