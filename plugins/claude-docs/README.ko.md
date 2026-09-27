[English](README.md) | 한국어

# claude-docs

프로젝트의 Claude Code 문서를 얇은 `CLAUDE.md`와 `.claude/rules/`로 이루어진 표준 구조로 만드는 스킬이다. 문서가 없는 프로젝트에는 새로 만들고, 이미 두꺼운 `CLAUDE.md`가 있는 프로젝트는 내용을 살려 새 구조로 옮긴다.

## 설치

```bash
claude plugin marketplace add mahns1201/claude-plugins
claude plugin install claude-docs@mahns
```

## 사용

프로젝트 루트에서 Claude Code 세션을 열고 실행한다.

```text
/claude-docs:setup
```

인자 없이 실행하면 `audit` 모드다. 현재 문서를 읽기만 하고 표준과 어디가 다른지 보고한다. 루트 `CLAUDE.md`가 몇 줄인지, `paths` 없이 매 세션 로드되는 규칙이 몇 줄인지, `@` 임포트나 잘못된 위치의 파일이 있는지, `settings.json`에 환경 변수가 들어가 있지는 않은지 같은 것들이다. 마지막에 다음 모드를 제안한다. 무엇을 바꾸기 전에 먼저 한 번 돌려 보는 용도다.

```text
/claude-docs:setup init
```

`CLAUDE.md`도 `.claude/rules/`도 없는 프로젝트에서 쓴다. 공통 규칙 파일을 복사한 뒤, 빌드 파일에서 스택과 명령어를, 디렉터리 구조와 import 관계에서 모듈과 의존 방향을, 기존 코드에서 이름 규칙과 테스트 구조를 읽어 프로젝트 문서를 만든다. 코드에서 확인한 것만 쓰고 확인하지 못한 자리는 `TODO`로 남기므로, 끝나면 한 번 훑어보며 `TODO`를 채우면 된다.

```text
/claude-docs:setup migrate
```

이미 Claude 문서가 있는 프로젝트에서 쓴다. 기존 `CLAUDE.md`와 규칙 파일을 절 단위로 읽어 새 구조의 어느 파일로 갈지 분류하고, 갈 곳이 없는 내용은 따로 모아 어떻게 할지 묻는다. 계획을 표로 보여주고 확인을 받은 뒤에야 파일을 쓴다. 원본은 git이 추적하고 있으면 삭제하고(히스토리에 남는다), 아니면 `.claude/legacy/`로 옮긴다. 삭제와 이동은 파일마다 권한 확인이 뜬다. 사용자 파일을 지우는 유일한 단계라 미리 허용해 두지 않았다.

`init`과 `migrate`에서는 권한 확인이 뜬다. Claude Code가 `.claude/` 디렉터리를 보호하고 작업 디렉터리 밖의 셸 접근을 막는데, 플러그인은 둘 다 미리 허용할 수 없다. 그래서 `.claude/` 아래에 쓰는 명령과 플러그인 디렉터리에서 복사하는 명령마다 한 번씩 묻는다. 종류별 첫 프롬프트에서 "이 세션에서 항상 허용"을 고르면 된다. `migrate`에서 원본 파일을 지우고 옮기는 `rm`, `mv`만 하나씩 확인하는 편이 낫다.

Opus 이상 모델과 effort xhigh 이상을 권장한다. 코드베이스를 읽고 규칙을 추론하는 작업이라 작은 모델은 얕거나 지어낸 규칙을 만들기 쉽다.

## 만들어지는 것

루트 `CLAUDE.md`는 프로젝트 개요, 명령어, 그리고 작업 유형별로 어떤 규칙 파일을 읽을지 알려주는 표만 갖는다. 50줄 이하다.

`.claude/rules/` 아래에는 두 종류의 파일이 생긴다. `CLAUDE_CODE.md`와 `MARKDOWN.md`는 어느 프로젝트에서나 같은 공통 규칙으로, 내용을 바꾸지 않고 복사된다. `ARCHITECTURE.md`, `NAMING.md`, `TEST.md`, `REVIEW.md`는 프로젝트마다 다르므로 코드베이스 분석 결과로 채워진다. 각 규칙 파일에는 `paths`가 붙어 관련 파일을 읽을 때만 로드된다. `NAMING.md`와 `REVIEW.md`는 모든 파일에 해당하므로 `**/*`를 쓴다.

모듈마다 그 모듈 루트에 작은 `CLAUDE.md`가 생긴다. `ARCHITECTURE.md`가 전체 구조를 다루고, 모듈 `CLAUDE.md`가 그 안의 관례를 다룬다. Claude가 그 모듈의 파일을 읽을 때 자동으로 로드된다.

`.claude/settings.json`은 팀 공유 설정, `.claude/settings.local.json`은 개인 설정이다. 환경 변수는 local에만 두고, `CLAUDE.local.md`와 함께 `.gitignore`에 추가된다.

분석 중에 표준 구조에 없는 영역이 보이면(예: DB 스키마, API 계약) 추가 규칙 파일을 제안한다. 만들지는 않고 제안만 하며, 사용자가 고른 것만 만든다.

## 하지 않는 것

git에 쓰지 않는다. 커밋, 머지, 푸시는 스킬 안에서 막혀 있고, 끝난 뒤 결과를 보고 사용자가 커밋한다.

개인 파일을 건드리지 않는다. `CLAUDE.local.md`와 `.claude/settings.local.json`이 이미 있으면 그대로 둔다.

근거 없는 규칙을 만들지 않는다. 코드에서 확인한 것만 쓰고, 나머지는 `TODO`다.

기존 내용을 조용히 버리지 않는다. `migrate`에서 갈 곳 없는 내용은 목록으로 보여주고 사용자가 정한다.
