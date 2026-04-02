# Superpowers 스킬 워크플로우 변경 요약

기준 날짜: 2026-04-02

범위:
- `skills-custom/writing-plans/SKILL.md`
- `skills-custom/executing-plans/SKILL.md`
- `skills-custom/subagent-driven-development/SKILL.md`
- `skills-custom/parallel-subagent-execution/SKILL.md`
- `skills-custom/requesting-code-review/SKILL.md`
- `skills-custom/receiving-code-review/SKILL.md`
- `skills-custom/requesting-code-review/code-reviewer.md`
- `skills-custom/subagent-driven-development/implementer-prompt.md`
- `skills-custom/parallel-subagent-execution/implementer-prompt.md`
- `agents/code-reviewer.md`

관련 가이드:
- 현재 최종 워크플로우 설명은 `SKILL_WORKFLOW_GUIDE_KO.md`

## 왜 수정했는가

이번 수정의 목적은 아래 4가지를 문서에 명확히 반영하는 것이었다.

- `self-review`, `integration verification`, `formal review`의 경계를 분명히 하기
- 3가지 실행 모드 모두 `formal review`를 `모든 task/wave 완료 후 최종 1회`로 맞추기
- `requesting-code-review`와 `receiving-code-review`의 호출 경계를 분명히 하기
- `parallel-subagent-execution`에 부족했던 운영 가이드 추가하기

## 공통 변경 원칙

- task별 `implementer self-review`는 유지
- task별 routine `formal review`는 하지 않음
- `formal review`는 최종 통합 결과 기준으로 1회 수행
- `receiving-code-review`는 실제 피드백이 있을 때만 사용
- `file ownership`은 OS 권한이 아니라 `assigned file scope`를 뜻함
- `personal project` 같은 표현 대신 일반적인 workflow 규칙으로 서술

## 파일별 변경 요약

### `skills-custom/writing-plans/SKILL.md`

- Before:
  - 실행 모드 설명은 있었지만 `self-review`, `integration verification`, `formal review`의 차이가 분명하지 않았다.
  - 모드별 review 시점은 읽는 사람이 해석해야 했다.
- After:
  - `Review Terms` 섹션을 추가했다.
  - 3가지 실행 모드 모두 최종 `formal review 1회`가 기본이라는 점을 명시했다.
- Why:
  - 상위 planning 문서에서 용어와 review 타이밍을 먼저 고정해야 아래 실행 문서도 일관되게 읽힌다.

### `skills-custom/executing-plans/SKILL.md`

- Before:
  - review 게이트가 선택적이고 위험도 기반처럼 읽혔다.
  - task별 review를 하지 않는다는 점은 충분히 분명하지 않았다.
- After:
  - `Final Formal Review Gate`로 바꿨다.
  - 모든 task 완료 후 최종 통합 결과에 대해 `requesting-code-review`를 수행하도록 정리했다.
  - actionable feedback이 있을 때만 `receiving-code-review`를 사용하도록 명시했다.
- Why:
  - Inline 모드도 최종 1회 formal review 구조로 통일하려는 목적이다.

### `skills-custom/subagent-driven-development/SKILL.md`

- Before:
  - 전체 흐름은 최종 1회 review에 가까웠지만, `self-review`와 `formal review` 경계가 문장으로는 약했다.
  - 일부 표현은 그냥 `final review`라고만 적혀 있었다.
- After:
  - `Review Boundaries`를 추가했다.
  - implementer `self-review`는 local quality check일 뿐 formal review가 아니라고 못 박았다.
  - formal review는 모든 task 완료 후 1회라고 명시했다.
  - overview, example, quality gates 표현도 `final formal review`로 맞췄다.
- Why:
  - task별 리뷰가 도는 것으로 오해하지 않도록 하기 위해서다.

### `skills-custom/parallel-subagent-execution/SKILL.md`

- Before:
  - 병렬 실행의 뼈대는 있었지만 `Model Selection`, `Handling Implementer Status`, `file ownership`의 의미가 문서상 부족했다.
  - wave verification과 formal review의 경계가 약했다.
- After:
  - `Review Boundaries`를 추가했다.
  - `Model Selection`을 추가했다.
  - `Handling Implementer Status`를 추가했다.
  - `file ownership`이 `assigned file scope`라는 뜻이라고 명시했다.
  - wave verification은 formal review가 아니고, formal review는 모든 wave 완료 후 1회라고 명시했다.
- Why:
  - 병렬 모드는 제약이 많아서 운영 규칙을 명확히 적어야 실제로 안전하게 쓸 수 있다.

### `skills-custom/requesting-code-review/SKILL.md`

- Before:
  - review 요청 시점은 있었지만 최종 통합 결과 중심이라는 점이 지금보다 약했다.
  - placeholder 이름도 template와 완전히 맞지 않았다.
- After:
  - description을 `completed integrated change set` 중심으로 바꿨다.
  - review timing을 `모든 implementation task 완료 후 1회`로 명확히 적었다.
  - routine per-task formal review는 기본값이 아니라고 명시했다.
  - placeholder를 `PLAN_REFERENCE` 기준으로 통일했다.
- Why:
  - review를 언제, 무엇에 대해 요청하는지 문서가 직접 답하게 하기 위해서다.

### `skills-custom/receiving-code-review/SKILL.md`

- Before:
  - formal review 후속 단계처럼 읽히면서도, 사람 리뷰와 외부 리뷰를 포함하는지 경계가 애매했다.
  - 일부 personalized phrasing이 남아 있었다.
- After:
  - description을 `review feedback from a human, external reviewer, or review subagent` 기준으로 바꿨다.
  - 피드백이 실제로 있을 때만 호출한다고 명시했다.
  - no actionable findings면 별도 reception step이 필요 없다고 적었다.
  - 일부 personalized 표현을 중립적인 workflow 언어로 바꿨다.
- Why:
  - `requesting`과 `receiving`의 경계를 분명히 하려면 이 문서가 가장 명확해야 한다.

### `skills-custom/requesting-code-review/code-reviewer.md`

- Before:
  - formal review 대상이 최종 통합 결과라는 점이 지금보다 약했다.
  - placeholder 이름이 calling 문서와 완전히 맞지 않았다.
- After:
  - `completed integrated change set`를 검토하는 formal review라는 점을 앞부분에 못 박았다.
  - `PLAN_REFERENCE`로 placeholder를 통일했다.
- Why:
  - review template 자체가 최종 통합 결과 중심으로 읽혀야 실제 reviewer 출력도 흔들리지 않는다.

### `skills-custom/subagent-driven-development/implementer-prompt.md`

- Before:
  - self-review가 workflow의 formal review와 어떻게 다른지 prompt 안에 직접 적혀 있지 않았다.
  - 수정 중 한때 들여쓰기 문제가 생겼으나 함께 바로 정리했다.
- After:
  - self-review는 local implementer check이지 formal review가 아니라고 명시했다.
  - prompt 블록 들여쓰기를 정상화했다.
- Why:
  - implementer prompt에서도 같은 용어 경계를 유지해야 한다.

### `skills-custom/parallel-subagent-execution/implementer-prompt.md`

- Before:
  - file ownership의 뜻을 처음 읽는 사람이 오해할 수 있었다.
  - self-review와 formal review의 차이가 직접 적혀 있지 않았다.
- After:
  - file ownership은 OS 권한이 아니라 task별 assigned file scope라고 명시했다.
  - self-review는 local implementer check이지 formal review가 아니라고 명시했다.
- Why:
  - 병렬 모드에서는 ownership과 review 경계 오해가 곧 통합 문제로 이어질 수 있다.

### `agents/code-reviewer.md`

- Before:
  - `major project step` 또는 `completed implementation batch`처럼 읽혀 task 중간 review를 떠올리게 할 여지가 있었다.
  - severity 표현도 template와 완전히 일치하지 않았다.
- After:
  - `completed integrated result` 또는 `full task series`를 review 대상으로 바꿨다.
  - severity를 `Critical / Important / Minor`로 통일했다.
  - project-specific 표현을 줄였다.
- Why:
  - reviewer agent 설명 자체가 최종 formal review 정책과 같은 방향을 가리켜야 한다.

## 수정 과정에서 같이 정리한 항목

초기 수정 후 검토에서 아래도 함께 정리했다.

- review template placeholder 이름 불일치
- `receiving-code-review` 경계 설명 충돌
- `subagent-driven-development/implementer-prompt.md` 들여쓰기 문제
- reviewer agent와 template 사이 severity 용어 불일치
- 남아 있던 personalized 또는 project-specific 표현 일부

## 작성 후 확인

문서 수정 후 아래를 확인했다.

- `git diff --check` 기준 공백/형식 오류 없음
- 변경된 문서들만 대상으로 정합성 검토 수행
- 별도 reviewer 관점 검토에서 나온 지적 사항 반영 완료

## 요약

이번 변경으로 현재 문서는 아래 원칙을 명시적으로 따르도록 정리되었다.

- task별 `self-review`는 유지
- `formal review`는 최종 통합 결과 기준으로 1회
- `receiving-code-review`는 실제 feedback이 있을 때만 사용
- 병렬 모드는 ownership, model selection, status handling까지 문서화

즉, 속도와 안정성 사이에서 `task마다 formal review`가 아니라 `최종 1회 formal review`를 기본값으로 두는 방향이 문서에 반영되었다.
