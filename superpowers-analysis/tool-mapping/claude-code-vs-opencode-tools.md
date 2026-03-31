# Claude Code vs OpenCode Tools

## 기준

- 작성 기준일: 2026-03-31
- 기준 문서:
  - Claude Code tools reference: https://code.claude.com/docs/ko/tools-reference
  - OpenCode tools: https://opencode.ai/docs/ko/tools/
  - 보조 확인: OpenCode modes: https://opencode.ai/docs/modes/

## 정리 원칙

- 표에는 공식 문서에 보이는 tool 이름만 적었다.
- 한쪽에 비슷한 tool이 없으면 대응 칸을 비워뒀다.
- OpenCode의 `grep`/`glob`는 tools 페이지에서 "internal"로만 설명되므로 대응 tool로 보지 않았다.
- `EnterPlanMode`와 `ExitPlanMode`는 Claude Code의 tool 이름이다. OpenCode는 별도 modes 문서에서 `plan` mode를 설명하지만 tools 페이지에는 대응 tool명이 없다.

## 도구 비교표

| Claude Code tool | Claude Code 역할 | OpenCode tool | OpenCode 역할 |
|---|---|---|---|
| Agent | subagent를 생성해 작업을 위임 |  |  |
| AskUserQuestion | 요구사항 확인용 질문 수행 | question | 실행 중 사용자에게 질문 |
| Bash | shell 명령 실행 | bash | shell 명령 실행 |
| CronCreate | 반복/일회성 예약 작업 생성 |  |  |
| CronDelete | 예약 작업 취소 |  |  |
| CronList | 예약 작업 목록 조회 |  |  |
| Edit | 기존 파일에 대상 편집 수행 | edit | 정확한 문자열 교체로 기존 파일 수정 |
| EnterPlanMode | 코딩 전 접근 방식을 설계하는 Plan Mode 진입 |  |  |
| EnterWorktree | 격리된 git worktree 생성 및 전환 |  |  |
| ExitPlanMode | 승인용 계획을 제시하고 Plan Mode 종료 |  |  |
| ExitWorktree | worktree 세션 종료 및 원래 디렉토리 복귀 |  |  |
| Glob | 패턴 기반으로 파일 찾기 |  |  |
| Grep | 파일 내용에서 패턴 검색 |  |  |
| ListMcpResourcesTool | MCP 서버가 노출한 리소스 목록 조회 |  |  |
| LSP | 언어 서버 기반 코드 인텔리전스 | lsp | 언어 서버 기반 코드 인텔리전스 |
| NotebookEdit | Jupyter 노트북 셀 수정 |  |  |
| Read | 파일 내용 읽기 | read | 파일 읽기 |
| ReadMcpResourceTool | 특정 MCP 리소스 읽기 |  |  |
| Skill | skill 실행/로드 | skill | `SKILL.md` 로드 후 대화에 내용 반환 |
| TaskCreate | 작업 목록에 새 작업 생성 |  |  |
| TaskGet | 특정 작업의 상세 조회 |  |  |
| TaskList | 현재 작업 목록 조회 |  |  |
| TaskOutput (deprecated) | 백그라운드 작업 출력 조회 |  |  |
| TaskStop | 실행 중인 백그라운드 작업 종료 |  |  |
| TaskUpdate | 작업 상태/세부 정보 업데이트 또는 삭제 |  |  |
| TodoWrite | 세션 작업 체크리스트 관리 | todowrite | 코딩 세션 중 todo 목록 관리 |
| ToolSearch | 지연 로드된 tool 검색 및 로드 |  |  |
| WebFetch | 지정 URL의 콘텐츠 가져오기 | webfetch | 웹 콘텐츠 fetch |
| WebSearch | 웹 검색 수행 | websearch | Exa 기반 웹 검색 |
| Write | 파일 생성 또는 덮어쓰기 | write | 새 파일 생성 또는 기존 파일 덮어쓰기 |
|  |  | list | 파일/디렉토리 목록 조회 |
|  |  | patch | 파일에 patch 적용 |

## 메모

- OpenCode tools 페이지에는 `list`, `patch`, `question`이 있지만 Claude Code tools reference에는 같은 이름의 direct 대응 tool이 없다.
- Claude Code tools reference에는 task/worktree/cron/MCP resource 관련 tool이 많지만 OpenCode tools 페이지에는 대응 tool이 보이지 않는다.
- OpenCode는 modes 문서에서 `plan` mode를 설명하지만, Claude Code처럼 `EnterPlanMode` / `ExitPlanMode`라는 tool 이름으로 공개하지는 않는다.
