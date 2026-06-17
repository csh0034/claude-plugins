# claude-plugins

개인용 Claude Code 플러그인 레포입니다. 이 레포는 마켓플레이스이면서 플러그인 하나(`common`)를 담고 있습니다.
스킬은 `skills/` 아래에 하나씩 추가합니다.

## 수록 스킬 (plugin: common)

| 스킬 | 호출 | 설명 |
|---|---|---|
| `toy` | `/common:toy` | 기술 주제를 번호 매긴 Markdown 문서 docs 레포로 정리 (비교표 README + 컨벤션 CLAUDE.md + 문서별 한글 커밋) |

## 설치

다른 머신/환경의 Claude Code에서 이 플러그인을 쓰려면 아래를 실행합니다. `<github-user>/claude-plugins`는 이 레포 경로로 바꿉니다.

```
/plugin marketplace add <github-user>/claude-plugins
/plugin install common@csh0034-marketplace
```

설치 후 새 세션부터 스킬이 인식됩니다. 이미 추가한 마켓플레이스를 갱신하려면 `/plugin marketplace update`.

## 사용법

- 슬래시 명령: `/common:toy`
- 또는 자연어로: "~ 정리해줘", "~ 문서로 만들어줘", "조사해서 docs로", "write up docs for ~"

## 구조

```
.
├── .claude-plugin/
│   ├── marketplace.json          # 마켓플레이스 카탈로그 (source: "./")
│   └── plugin.json               # 플러그인 매니페스트 (name: common)
└── skills/
    └── toy/
        ├── SKILL.md
        └── references/templates.md
```

## 새 스킬 추가하는 법

1. `skills/<new-skill>/SKILL.md` 작성 (frontmatter의 `name`은 `<new-skill>`)
2. 커밋·푸시
3. `/plugin marketplace update`로 갱신하면 `/common:<new-skill>`로 호출 가능

플러그인은 `common` 하나를 유지하고, 기능은 그 안에 스킬로 늘려간다.

## 업데이트

스킬을 수정하면 `.claude-plugin/plugin.json`의 `version`을 올리고 푸시한다. `version`을 안 올리면 commit SHA로 추적돼 업데이트 파악이 번거롭다.
