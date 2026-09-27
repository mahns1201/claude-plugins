[English](README.md) | 한국어

# claude-plugins

Claude Code를 쓰다 보면 `CLAUDE.md`가 자란다. 빌드 명령 옆에 아키텍처 설명이 붙고, 그 아래 네이밍 규칙과 테스트 작성법과 리뷰 체크리스트가 쌓인다. 그 파일은 매 세션 시작 때 통째로 컨텍스트에 실리고, 길어질수록 Claude는 그 안의 지시를 덜 지킨다. 다른 프로젝트를 시작하면 같은 문서를 처음부터 다시 쓴다.

이 레포는 그 문제에 대한 답을 플러그인 하나로 묶은 것이다. 루트 `CLAUDE.md`는 프로젝트 개요, 명령어, 그리고 "어떤 작업에 어떤 규칙 파일을 읽어야 하는지" 표만 갖는다. 규칙 자체는 `.claude/rules/` 아래 파일로 나누고, 각 파일에 `paths`를 붙여 관련 파일을 읽을 때만 로드되게 한다. 모듈마다 그 모듈 루트에 작은 `CLAUDE.md`를 두어 모듈 안에서만 유효한 관례를 담는다. 어느 프로젝트에 가도 같은 규칙은 그대로 복사하고, 프로젝트마다 다른 내용은 코드베이스를 읽어서 채운다.

## 어떻게 동작하는가

이 레포는 Claude Code 플러그인 마켓플레이스다. 등록하면 `claude-docs` 플러그인을 설치할 수 있고, 그 안에 `setup` 스킬 하나가 들어 있다. 스킬은 세 가지 모드로 동작한다.

`audit`은 아무것도 쓰지 않는다. 현재 프로젝트의 Claude 문서를 훑어 표준과 어디가 다른지, 매 세션 몇 줄이 항상 로드되는지, `@` 임포트나 잘못된 위치의 파일이 있는지 보고하고 다음에 어떤 모드를 쓰면 좋을지 제안한다. 인자 없이 실행하면 이 모드다.

`init`은 Claude 문서가 하나도 없는 프로젝트를 위한 것이다. 공통 규칙 파일을 복사하고, 빌드 파일과 디렉터리 구조와 기존 코드를 읽어 프로젝트 문서를 만든다. 코드에서 확인한 것만 쓰고, 확인하지 못한 자리는 `TODO`로 남긴다.

`migrate`는 이미 `CLAUDE.md`가 있는 프로젝트를 위한 것이다. 기존 문서를 절 단위로 읽어 새 구조의 어느 파일로 가야 하는지 분류하고, 갈 곳이 없는 내용은 따로 모아 사용자에게 묻는다. 계획을 한 번 확인받은 뒤 새 문서를 쓰고 원본을 정리한다. 내용을 조용히 버리는 일은 없다.

플러그인 사용법은 `plugins/claude-docs/README.ko.md`에 있다.

## 레포 구조

```text
claude-plugins/                        # 마켓플레이스 루트
├── .claude-plugin/marketplace.json        # 플러그인 카탈로그 (name: mahns)
├── plugins/claude-docs/                   # 배포되는 유일한 단위
│   ├── .claude-plugin/plugin.json
│   ├── README.md, README.ko.md
│   └── skills/setup/
│       ├── SKILL.md                       # 모드 판단과 공통 절차
│       ├── references/                    # 모드별 절차: AUDIT.md, INIT.md, MIGRATE.md
│       ├── common/                        # 그대로 복사되는 공통 파일
│       └── samples/                       # 분석 결과로 채워지는 문서의 골격
└── README.md, README.ko.md
```

사용자 머신에 설치되는 것은 `plugins/claude-docs/` 디렉터리만이다. 루트의 README는 이 레포를 개발하거나 방문하는 사람을 위한 것이다.

## 설계에서 정한 것

플러그인은 `CLAUDE.md`나 `rules/`를 직접 사용자 프로젝트에 배포할 수 없다. Claude Code가 플러그인 루트의 `CLAUDE.md`를 읽지 않기 때문이다. 그래서 스킬이 파일을 복사하고 생성하는 방식을 택했다.

스킬 안에 셸 스크립트를 두지 않는다. 복사는 `cp`, 무엇을 어디에 쓸지는 Claude가 판단한다. 파일이 여섯 개 남짓인 규모에서 스크립트는 유지 비용만 늘렸다.

스킬은 git에 쓰지 않는다. 커밋, 머지, 푸시는 프론트매터에서 막아 두었고, 실행이 끝나면 사용자가 결과를 보고 커밋한다.

문서는 영어로 작성한다. README는 영어를 기본으로 두고 한국어판 `README.ko.md`를 따로 둔다.

## 개발과 배포

수정 후에는 매니페스트 둘을 검증한다.

```bash
claude plugin validate .
claude plugin validate --strict ./plugins/claude-docs
```

수정하면서 실제 세션에서 써 보려면 플러그인 디렉터리를 직접 로드한다. 파일을 그 자리에서 읽으므로 수정 뒤 `/reload-plugins`만 하면 반영된다.

```bash
claude --plugin-dir ./plugins/claude-docs
```

설치 경로 자체를 시험하려면 이 디렉터리를 로컬 마켓플레이스로 등록한다. 이때는 플러그인이 `~/.claude/plugins/cache/`로 복사되므로, 수정 내용은 uninstall 후 다시 install해야 반영된다.

```bash
claude plugin marketplace add ./
claude plugin install claude-docs@mahns
```

배포는 GitHub에 push하는 것으로 끝난다. 플러그인 이름 `claude-docs`과 마켓플레이스 이름 `mahns`는 배포 후 바꾸지 않는다. 바꾸면 기존 설치가 전부 깨진다. 릴리스마다 `plugins/claude-docs/.claude-plugin/plugin.json`의 `version`을 올려야 사용자가 `claude plugin update`로 새 버전을 받는다.

## 첫 배포에서 미룬 것

스킬 예제, 서브에이전트 예제, PRD 규칙, MCP 설정, 코드 변경 정책은 첫 배포에서 뺐다. 다음 배포에서 `common/`이나 `samples/`에 파일을 추가하고 `SKILL.md`의 파일 분류 표에 한 줄 넣으면 된다.
