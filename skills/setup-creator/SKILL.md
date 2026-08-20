---
name: setup-creator
description: 현재 프로젝트를 조사해 신규 참여자용 초기 세팅 가이드 스킬(`.claude/skills/setup/SKILL.md`)을 생성한다. 런타임·툴 버전, env·config, docker·인프라 의존성, 빌드·실행·테스트 검증을 의존 순서대로 안내하고, 확인 명령은 바로 실행하되 변경이 생기는 명령은 사용자 확인 후 실행하도록 작성한다. `/common:setup-creator`로 명시적으로 호출했을 때만 사용한다. 자동으로 트리거하지 않는다.
user-invocable: true
disable-model-invocation: true
allowed-tools: Read, Write, Edit, Bash, AskUserQuestion
---

# setup-creator

**`/setup` 스킬을 만드는 스킬.** 현재 프로젝트를 조사해서, 이 프로젝트를 처음 세팅하는 사람이 순서대로 따라갈 수 있는 `.claude/skills/setup/SKILL.md`를 생성한다.

산출물은 그 파일 하나다. `SETUP.md` 같은 별도 문서를 만들지 않는다.

핵심 태도: **프로젝트에 실제로 있는 것만 쓴다.** 조사로 확인되지 않은 건 추측해서 채우지 말고 사용자에게 묻거나 `확인 필요`로 남긴다.

## 워크플로

### 1. 대상 확인

- `pwd`, `git rev-parse --show-toplevel`로 대상 프로젝트를 확정해 되읽어 준다. 레포 루트가 아니면 루트를 대상으로 할지 확인한다.
- `ls .claude/skills/setup/ 2>/dev/null` — 이미 있으면 **덮어쓰기 전에 확인**을 받는다. 기존 내용을 먼저 읽고, 갱신인지 재작성인지 묻는다.

### 2. 프로젝트 조사 (읽기 전용)

아래 순서로 본다. **CI 설정이 가장 신뢰도 높은 근거다** — 실제로 돌아가는 명령이 거기 적혀 있다.

| 우선 | 소스 | 얻는 것 |
|---|---|---|
| 1 | `.github/workflows/*`, `.gitlab-ci.yml`, `Jenkinsfile` | 실제 설치·빌드·테스트 명령, 런타임 버전, 서비스 컨테이너 |
| 2 | `package.json` / `build.gradle(.kts)` / `pom.xml` / `pyproject.toml` / `go.mod` / `Cargo.toml` | 패키지 매니저, 스크립트, 필요 런타임 |
| 3 | `.nvmrc`, `.tool-versions`, `.python-version`, `mise.toml`, `Dockerfile`의 base image, gradle toolchain | 고정된 버전 |
| 4 | `.env.example`, `.env.sample`, `application*.yml`, `config/*` | 필수 env 키, config 프로필 |
| 5 | `docker-compose*.yml`, `compose*.yml` | 인프라 서비스, 포트, 초기화 스크립트 |
| 6 | 마이그레이션 디렉토리(`migrations/`, `db/migrate`, flyway/liquibase 경로), 시드 스크립트 | DB 준비 절차 |
| 7 | 기존 `README.md`, `CONTRIBUTING.md`, `docs/`, `Makefile`, `Taskfile.yml` | 이미 문서화된 세팅 절차 |
| 8 | 모노레포 여부(`pnpm-workspace.yaml`, `turbo.json`, `settings.gradle`, `go.work`) | 워크스페이스 단위 세팅 필요 여부 |

- 조사 결과를 **표로 요약해 보고**한다: 항목 / 확인된 값 / 근거 파일. 근거 없는 줄은 만들지 않는다.
- 프로젝트에 없는 범주(예: docker 안 씀)는 그냥 뺀다. 빈 섹션을 만들지 않는다.

### 3. 조사로 알 수 없는 것만 질문

`AskUserQuestion`으로 묻는다. **조사로 이미 답이 나온 건 다시 묻지 않는다.** 주로 다음이 남는다.

- **시크릿 값 획득 경로**: `.env.example`에 키 이름은 있어도 값을 어디서 받는지는 레포에 없다. (누구에게 요청 / 시크릿 매니저 / 사내 위키 / 개인 발급)
- **사내 전용 의존성**: VPN, 프라이빗 레지스트리 인증(`~/.npmrc`, `settings.xml`), 사내 DNS 필요 여부.
- **선택 항목**: compose 서비스나 프로필 중 로컬 세팅에 필수가 아닌 것.
- **지원 환경**: macOS 전용인지, arm64/amd64 이슈가 있는지.
- **툴 설치 방식**: brew / mise / asdf / 수동 중 팀 표준.

애매하면 추측하지 말고 묻는다. 사용자가 "모른다"고 하면 그 항목은 `확인 필요`로 남긴다.

### 4. 세팅 순서 확정

의존 순서를 지킨다. 앞 단계가 끝나야 뒤 단계가 성립하는 순서다.

```
툴·런타임 → 저장소·의존성 설치 → 시크릿·env → 인프라(docker) → DB 준비 → 앱 실행 → 검증(테스트)
```

- 단계 목록을 되읽어 확인받은 뒤 작성에 들어간다.
- 단계는 프로젝트에 필요한 만큼만. 억지로 7단계를 다 채우지 않는다.

### 5. `.claude/skills/setup/SKILL.md` 작성

아래 템플릿을 따른다. `<>`는 조사 결과로 채운다.

````markdown
---
name: setup
description: <프로젝트명> 로컬 개발환경을 처음부터 세팅한다. <툴 버전 / env / docker / DB / 실행 검증> 순으로 진행하며, 확인 명령은 바로 실행하고 변경이 생기는 명령은 사용자 확인 후 실행한다. `/setup`으로 명시적으로 호출했을 때만 사용한다.
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash, Read, Edit, AskUserQuestion
---

# setup

<프로젝트 한 줄 설명>의 로컬 개발환경 세팅 가이드.

## 실행 규칙

- **확인 명령은 바로 실행한다** — 버전 조회, `docker ps`, `lsof -i`, 파일 존재 확인 등 읽기 전용.
- **변경이 생기는 명령은 실행 전 사용자에게 확인받는다** — 툴 설치, 파일 생성·수정, 컨테이너 기동, 의존성 설치, DB 마이그레이션·시드.
- 각 단계는 **완료 기준**을 통과해야 다음으로 넘어간다. 실패하면 멈추고 원인을 보고한다.
- 시크릿 값은 사용자에게 받아 쓰고, 파일이나 대화에 값을 남기지 않는다.

## 0. 사전 확인

```bash
<git remote -v / 필요한 CLI 존재 확인>
```

## 1. 런타임·툴 버전

| 툴 | 필요 버전 | 근거 |
|---|---|---|
| <node> | <20.x> | <.nvmrc> |

확인:
```bash
<node -v>
```

불일치하면 → <설치·전환 방법>. **설치는 확인 후 실행.**

완료 기준: <모든 툴이 요구 버전으로 조회됨>

## 2. 의존성 설치

```bash
<pnpm install --frozen-lockfile>
```
**변경 명령 — 확인 후 실행.** <프라이빗 레지스트리 인증 필요 여부>

완료 기준: <설치 성공, lock 파일 변경 없음>

## 3. env·config

| 키 | 용도 | 값 획득 경로 |
|---|---|---|
| <DB_URL> | <로컬 DB 접속> | <compose 기본값 사용> |
| <API_KEY> | <외부 API> | <확인 필요 — 팀에 요청> |

```bash
cp .env.example .env
```
**변경 명령 — 확인 후 실행.** 기존 `.env`가 있으면 덮어쓰지 않고 부족한 키만 추가한다.

완료 기준: <필수 키가 모두 채워짐. 빈 값 없음>

## 4. 인프라 (docker)

확인:
```bash
docker info >/dev/null && docker compose ps
<lsof -i :5432 등 포트 충돌 확인>
```

기동:
```bash
docker compose up -d <서비스>
```
**변경 명령 — 확인 후 실행.**

완료 기준: <서비스가 healthy. 포트 응답 확인 명령>

## 5. DB 준비

```bash
<마이그레이션 명령>
<시드 명령>
```
**변경 명령 — 확인 후 실행. 기존 로컬 데이터를 지울 수 있으므로 반드시 확인받는다.**

완료 기준: <테이블 존재 확인 명령>

## 6. 실행·검증

```bash
<빌드 명령>
<실행 명령>
<테스트 명령>
```

완료 기준: <앱이 http://localhost:PORT 응답, 테스트 통과>

## 트러블슈팅

| 증상 | 원인 | 조치 |
|---|---|---|
| <포트 이미 사용 중> | <기존 컨테이너> | <docker compose down> |

## 확인 필요

- <레포에서 확인되지 않아 미확정인 항목>
````

작성 규칙:

- 명령은 **실제 존재하는 것만** 쓴다. `package.json`에 없는 스크립트, compose에 없는 서비스명을 지어내지 않는다.
- 단계마다 **완료 기준**을 반드시 넣는다. 기준이 없으면 그 단계는 검증 불가다.
- 시크릿 값을 파일에 하드코딩하지 않는다. 획득 경로만 적는다.
- 서술은 한국어, 명령·경로·식별자는 원문 유지.
- 프로젝트에 해당 없는 섹션은 삭제한다.

### 6. 생성물 검증 (반드시 수행)

- frontmatter의 `name`이 `setup`이고 디렉토리명과 일치하는지 확인.
- 적어 넣은 **확인 명령(읽기 전용)을 실제로 돌려본다.** 명령이 없거나 실패하면 고친다.
- 참조한 파일 경로(`.env.example`, compose 파일 등)가 실재하는지 `ls`로 확인.
- 스크립트명이 `package.json`/`Makefile`/gradle task에 실제로 있는지 대조.
- 변경 명령은 실행하지 않는다. 문법과 실재 여부만 대조한다.
- 통과/수정 항목을 간단히 요약 보고한다.

### 7. 마무리 보고

- 생성 경로, 단계 목록, 각 단계의 근거 파일을 요약한다.
- `확인 필요`로 남은 항목을 따로 나열한다.
- 커밋은 하지 않는다. 대상 프로젝트의 커밋 컨벤션은 이 스킬이 모른다 — 사용자가 직접 커밋하도록 알린다.

## 안 하는 것

- 대상 프로젝트의 코드·설정을 **수정하지 않는다.** 조사는 읽기 전용, 쓰기는 `.claude/skills/setup/SKILL.md` 하나뿐.
- 실제 세팅을 대신 진행하지 않는다. 이 스킬은 가이드를 만들고, 세팅은 생성된 `/setup`이 한다.
- `.env`를 만들거나 시크릿을 조회하지 않는다.
