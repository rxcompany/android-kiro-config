# android-kiro-config

Prizm Android 프로젝트의 Kiro CLI 설정 저장소.

## 셋업

```bash
# 프로젝트 루트에서 실행
git clone https://github.com/rxcompany/android-kiro-config.git .kiro
```

## 구조

```
.kiro/
├── agents/                 # 에이전트 설정 (전체 개인)
├── settings/               # MCP 서버 등 (전체 개인)
├── skills/                 # 스킬 정의 (팀 공유)
│   ├── */SKILL.md
│   └── private/            # 개인 스킬
├── steering/               # 컨텍스트 (팀 공유)
│   ├── memory-bank/        # 프로젝트 가이드라인, 기술 스택, 도메인 용어
│   ├── dependency-management.md
│   └── private/            # 개인 steering rules
└── private/                # 개인 전용 공간
```

## 개인 설정

아래 경로들은 `.gitignore`로 제외됩니다. 각자 환경에 맞게 생성하세요.

| 경로 | 용도 |
|------|------|
| `agents/` | 에이전트 설정 (전체 개인) |
| `settings/` | MCP 서버 설정 (전체 개인) |
| `skills/credentials.md` | JIRA, Notion 등 인증 정보 |
| `skills/private/` | 개인 스킬 |
| `steering/private/` | 개인 steering rules |
| `private/` | 최상위 개인 전용 공간 |

## Git credential 설정

HTTPS로 push하려면 gh CLI credential helper 설정이 필요합니다:

```bash
gh auth setup-git
```
