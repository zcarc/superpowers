# Claude Code vs OpenCode Agents

## 기준

- 작성 기준일: 2026-03-31
- 기준 문서:
  - Claude Code sub-agents: https://code.claude.com/docs/ko/sub-agents
  - OpenCode agents: https://opencode.ai/docs/ko/agents/

## 정리 원칙

- 표에는 공식 문서에서 직접 확인한 agent / subagent 개념과 설정 방식만 적었다.
- 이름이 비슷해도 구조가 다르면 같은 기능으로 단정하지 않고 메모에 차이를 적었다.
- tool 비교 문서와 달리, agent는 단순 1:1 매핑보다 "운영 방식" 차이가 커서 개념/구성/호출 방식 중심으로 비교했다.

## 에이전트 방식 비교표

| 비교 항목 | Claude Code | OpenCode | 메모 |
|---|---|---|---|
| 상위 구조 | 메인 대화 + subagent 중심 | primary agent + subagent의 이원 구조 | OpenCode는 `primary`와 `subagent`를 공식 구조로 전면에 두고 설명한다. Claude Code 문서는 subagent를 중심으로 설명한다. |
| 내장 agent 구성 | Explore, Plan, general-purpose 등 내장 subagent 제공 | Build, Plan(primary) + General, Explore(subagent) 제공 | Claude의 Plan은 subagent이고, OpenCode의 Plan은 primary agent라서 이름은 비슷해도 역할 계층이 다르다. |
| 기본 사용자 상호작용 단위 | 메인 대화가 기본, 필요 시 Claude가 subagent에 위임 | primary agent를 직접 사용하고 필요 시 subagent 호출 | OpenCode는 세션 중 primary agent를 전환하는 개념이 더 전면에 있다. |
| subagent 자동 위임 | subagent의 `description`을 보고 Claude가 자동 위임 시점 판단 | primary agent가 `description`을 바탕으로 subagent를 자동 호출 가능 | 두 제품 모두 설명(description)이 자동 위임 판단의 핵심이다. |
| 수동 호출 방식 | `/agents`로 생성/관리, 프롬프트나 `claude --agent`로 사용 가능 | primary agent 전환 또는 메시지에서 `@subagent` mention | OpenCode는 `@general` 같은 직접 호출 문법을 공식 문서에 명시한다. |
| 정의 포맷 | YAML frontmatter가 있는 Markdown, `/agents` UI, `--agents` JSON, 플러그인 agent | `opencode.json` 또는 Markdown agent 파일 | 둘 다 Markdown 정의를 지원하지만, OpenCode는 JSON config 기반 구성이 더 전면에 보인다. |
| 프로젝트/전역 저장 위치 | `.claude/agents/`, `~/.claude/agents/`, 플러그인 `agents/`, 세션 한정 `--agents` | `.opencode/agents/`, `~/.config/opencode/agents/`, `opencode.json` | Claude는 세션 한정 CLI 정의가 공식 옵션으로 있다. OpenCode는 config/파일 기반 구성이 중심이다. |
| 이름과 설명 | `name`, `description`이 필수 frontmatter | `description`은 필수, Markdown 파일명은 agent 이름이 됨 | OpenCode Markdown agent는 파일명 기반 naming을 사용한다. |
| 시스템 프롬프트 위치 | frontmatter 아래 Markdown 본문 또는 JSON `prompt` | Markdown 본문 또는 JSON `prompt` | 둘 다 프롬프트 본문과 메타데이터를 분리한다. |
| model 설정 | `model` 지정 가능, 기본은 `inherit` | `model` 지정 가능, 미지정 시 primary는 전역 model, subagent는 호출한 primary의 model 상속 | 기본 상속 규칙은 둘 다 있지만, OpenCode는 primary/subagent 계층에 따라 상속 규칙을 문서화한다. |
| tool 제한 방식 | 기본적으로 부모 대화의 tool 상속, `tools` / `disallowedTools`로 제한 | `tools`에서 각 tool을 `true` / `false`로 제어 | Claude는 allowlist/denylist 방식, OpenCode는 bool override 방식이다. |
| permission 제어 | `permissionMode`로 권한 모드 지정 | `permission`에서 `ask` / `allow` / `deny` 지정 | OpenCode는 `bash` 명령별 glob 패턴 permission까지 공식 문서에서 설명한다. |
| subagent 생성 제어 | 메인 스레드 agent는 `Agent(...)` 허용 목록으로 생성 가능한 subagent 유형 제한 가능 | `permission.task`로 Task tool을 통해 호출 가능한 subagent 범위 제어 | 둘 다 "어떤 agent를 다시 호출할 수 있는가"를 제어하지만 설정 방식이 다르다. |
| 중첩 제약 | subagent는 다른 subagent를 생성할 수 없음 | hidden subagent를 Task tool로 programmatic 호출 가능 | OpenCode 문서는 internal helper 계열 subagent를 숨기고 Task tool로만 호출하는 패턴을 설명한다. Claude는 subagent 중첩 금지를 명시한다. |
| 추가 agent 옵션 | `skills`, `mcpServers`, `hooks`, `memory`, `background`, `isolation`, `maxTurns`, `effort` 등 | `temperature`, `steps`, `hidden`, `color`, `topP`, `permission.task` 등 | 확장 포인트의 성격이 다르다. Claude는 subagent 개별 기능 확장이 넓고, OpenCode는 agent mode/permission/task orchestration 쪽이 더 명시적이다. |

## 메모

- Claude Code의 핵심 단위는 "메인 대화가 필요할 때 subagent에 위임한다"는 흐름이다.
- OpenCode의 핵심 단위는 "primary agent를 직접 선택/전환하고, 필요하면 subagent를 자동 또는 `@mention`으로 호출한다"는 흐름이다.
- 이름만 보면 `Plan`, `Explore`, `General`이 서로 대응되는 것처럼 보이지만, 실제 계층은 다르다. 특히 Claude의 `Plan`은 subagent이고 OpenCode의 `Plan`은 primary agent다.
- Claude의 `Agent(...)`와 OpenCode의 `permission.task`는 둘 다 하위 agent 호출 범위를 제어하지만, Claude는 생성 가능 subagent 제한에 가깝고 OpenCode는 Task tool 호출 권한 제어에 가깝다.

## 요약

- Claude Code는 subagent 중심 위임 모델에 가깝다.
- OpenCode는 primary/subagent를 모두 1급 개념으로 다루는 multi-agent 구성 모델에 가깝다.
- 두 제품 모두 Markdown 기반 agent 정의와 설명 기반 자동 위임을 지원하지만, 설정 키와 런타임 구조는 상당히 다르다.
