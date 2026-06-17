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
