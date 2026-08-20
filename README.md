# claude-plugins

개인용 Claude Code 플러그인. 플러그인 `common`과 그 안의 스킬을 담는다.

## 설치

Claude Code 세션에서 아래를 순서대로 실행한다.

```
/plugin marketplace add https://github.com/csh0034/claude-plugins
```

```
/plugin install common@csh0034-claude-plugins
```

설치 후 새 세션부터 스킬이 인식된다. 이미 추가한 마켓플레이스를 갱신하려면 `/plugin marketplace update`.

## 수록 스킬

| 스킬 | 호출 | 설명 |
|---|---|---|
| `toy` | `/common:toy` | 기술 주제를 번호 매긴 Markdown 문서 docs 레포로 정리 |
| `setup-creator` | `/common:setup-creator` | 프로젝트를 조사해 신규 참여자용 `/setup` 세팅 가이드 스킬 생성 |

## 사용 예제

### `toy`

기술 주제 하나를 정리한 docs 레포를 현재 디렉토리에 만든다.

- **빈 디렉토리에서 실행한다.** 산출물을 현재 작업 디렉토리에 생성하므로 새 디렉토리에서 시작한다.
- 문서 단위로 한글 커밋한다.
- 비교표 중심 README와 작성 컨벤션 CLAUDE.md를 세팅한다.
- 필요하면 사용자 확인을 받아 코드 샘플까지 작성한다.
- `/common:toy`로 명시적으로 호출해야 동작한다.

```
/common:toy kafka
```

### `setup-creator`

현재 프로젝트를 조사해 `.claude/skills/setup/SKILL.md`를 생성한다. 이후 그 프로젝트에서 `/setup`으로 초기 세팅을 안내받는다.

- **세팅하려는 프로젝트 루트에서 실행한다.**
- CI 설정·빌드 파일·버전 고정 파일·`.env.example`·compose를 근거로 세팅 순서를 만든다.
- 시크릿 획득 경로나 사내 전용 의존성처럼 레포에서 알 수 없는 건 물어본다.
- 생성된 `/setup`은 확인 명령은 바로 실행하고, 변경이 생기는 명령은 사용자 확인 후 실행한다.
- 대상 프로젝트의 코드·설정은 수정하지 않는다. 커밋도 하지 않는다.

```
/common:setup-creator
```
