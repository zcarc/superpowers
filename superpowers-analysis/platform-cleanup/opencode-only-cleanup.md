# OpenCode 전용 정리 기록

## 목적

이 문서는 저장소를 OpenCode 중심으로 정리하면서 제거하거나 수정한 Codex / Gemini 관련 항목을 기록한 참고 문서다.

## 정리 기준

- `skills/using-superpowers/SKILL.md`를 OpenCode 전용으로 바꾼다.
- 현재 실행 경로와 사용자-facing 문서에서 Codex / Gemini 의존 항목을 제거한다.
- 과거 설계 문서, 구현 계획 문서, 릴리즈 노트 같은 히스토리성 문서는 참고 기록으로 남겨둘 수 있다.

## 판단 근거

### 1. `using-superpowers`의 OpenCode 전용화

- `skills/using-superpowers/SKILL.md`는 원래 Claude Code, Gemini CLI, Codex를 함께 설명하고 있었다.
- OpenCode만 사용할 경우:
  - `Skill` tool 표기는 OpenCode native `skill` 기준으로 정리 가능
  - `TodoWrite`는 `todowrite` 기준으로 정리 가능
  - `EnterPlanMode` 같은 Claude Code tool 명칭은 OpenCode tools 체계와 맞지 않으므로 제거 대상

### 2. Codex 관련 항목 정리 이유

- `skills/using-superpowers/references/codex-tools.md`는 `using-superpowers`가 플랫폼 중립 또는 OpenCode 전용으로 바뀐 이후 현재 실행 경로에서 더 이상 사용되지 않았다.
- `.codex/INSTALL.md` 와 `docs/README.codex.md` 는 Codex 설치/사용 문서였다.
- `README.md` 에도 Codex 설치 섹션이 남아 있었다.
- `skills/brainstorming/visual-companion.md` 와 `skills/brainstorming/scripts/start-server.sh` 에 Codex 전용 처리 설명이 남아 있었다.

### 3. Gemini 관련 항목 정리 이유

- `GEMINI.md` 는 OpenCode와 직접 관련은 없지만, 저장소에서 Gemini 지원의 실제 진입 파일이었다.
- `gemini-extension.json` 은 `GEMINI.md` 를 직접 가리키고 있었다.
- `skills/using-superpowers/references/gemini-tools.md` 는 Gemini용 tool mapping 참조 파일이었다.
- `README.md` 와 `skills/brainstorming/visual-companion.md` 에도 Gemini 관련 안내가 남아 있었다.

## 삭제한 파일

### Codex

- `.codex/INSTALL.md`
- `docs/README.codex.md`
- `skills/using-superpowers/references/codex-tools.md`

### Gemini

- `GEMINI.md`
- `gemini-extension.json`
- `skills/using-superpowers/references/gemini-tools.md`

## 수정한 파일

### 핵심 스킬

- `skills/using-superpowers/SKILL.md`
  - OpenCode 전용 설명으로 변경
  - `Skill` -> OpenCode native `skill`
  - `TodoWrite` -> `todowrite`
  - Claude / Gemini / Codex 설명 제거
  - 그래프에서 `EnterPlanMode` 제거

### 사용자-facing 문서

- `README.md`
  - Codex 설치 섹션 제거
  - Gemini CLI 설치 섹션 제거
  - 설치 안내를 OpenCode 기준으로 축소

### 브레인스토밍 관련 문서/스크립트

- `skills/brainstorming/visual-companion.md`
  - Codex / Gemini 전용 서버 실행 안내 제거
- `skills/brainstorming/scripts/start-server.sh`
  - Codex 전용 auto-foreground 분기 제거
  - `--background` 설명에서 Codex 언급 제거

### 기타 스킬 문구

- `skills/executing-plans/SKILL.md`
  - "Claude Code or Codex" 예시를 제거하고 플랫폼 중립 표현으로 수정

## 정리 후 상태

- 현재 실행 경로와 사용자-facing 문서 기준으로는 OpenCode 외 Codex / Gemini 지원 흔적을 제거했다.
- 다만 아래 종류의 파일에는 Codex / Gemini 언급이 남아 있을 수 있다:
  - `RELEASE-NOTES.md`
  - `docs/plans/`
  - `docs/superpowers/plans/`
  - `docs/superpowers/specs/`
  - 생성/검증 기록 성격의 내부 참고 문서

이 파일들은 현재 동작 경로가 아니라 과거 설계와 이력 보존 목적의 문서로 취급했다.

## 함께 보면 좋은 참고 문서

- `superpowers-analysis/skill-entrypoint.md`
- `superpowers-analysis/tool-mapping/claude-code-vs-opencode-tools.md`

## 요약

이번 정리는 "OpenCode만 사용한다"는 전제에서 진행했다.

- `using-superpowers`는 OpenCode 전용으로 바꿨다.
- Codex / Gemini 런타임 진입 파일과 설치 문서는 제거했다.
- 브레인스토밍 보조 문서와 스크립트의 플랫폼별 예시도 OpenCode 기준으로 단순화했다.
- 히스토리성 문서는 현재 참고 기록으로 남겨뒀다.
