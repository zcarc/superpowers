# Superpowers 스킬 및 에이전트 워크플로우 가이드

기준 날짜: 2026-04-02

검토 범위:
- `skills-custom/**/SKILL.md`
- `skills-custom/**/implementer-prompt.md`
- `skills-custom/requesting-code-review/code-reviewer.md`
- `agents/code-reviewer.md`

## 검토 요청 배경

이 문서는 아래 질문에 답하기 위해 작성했다.

- 루트 경로 기준으로 `skills-custom`, `agents`를 읽고 현재 workflow 이해가 맞는지 확인해 달라.
- 특히 `writing-skills` 관점에서도 문서 방향이 적절한지 보고 싶다.
- 답변만 있으면 왜 이런 정리가 나왔는지 맥락이 약하므로, 처음 가정한 흐름과 검토 포인트를 함께 남긴다.

처음 가정한 워크플로우:

1. `using-superpowers`
2. `brainstorming`
3. `writing-plans`
4. 다음 모드 중 하나 선택
   - `Inline Execution`
   - `Subagent-Driven`
   - `Parallel Subagents`
5. `using-git-worktrees`
6. `test-driven-development`
7. `verification-before-completion`

처음 확인하고 싶었던 해석:

- `Inline Execution`
  - 변경 범위가 아주 작거나 간단한 작업일 때 사용
  - subagent를 나누기보다 메인 에이전트에서 순차 실행하는 편이 나을 때 선택
  - 작은 작업이면 별도 plan 문서보다 대화 안의 짧은 체크리스트가 적절할 수 있음

- `Subagent-Driven`
  - plan의 task를 하나씩 나눠 fresh subagent로 순차 실행하는 방식
  - 메인 세션 컨텍스트 오염, 토큰 효율, 속도 측면에서 inline보다 유리할 때 선택

- `Parallel Subagents`
  - `Subagent-Driven`의 병렬 확장 형태
  - task끼리 충돌이 없으면 동시에 실행해 더 빠르게 처리하는 방식

추가로 검토하고 싶었던 쟁점:

- `requesting-code-review`, `receiving-code-review`는 언제 호출되는가
- 리뷰는 `writing-plans`의 모든 task 구현과 테스트가 끝난 뒤 한 번만 하는 것이 맞는가
- `finishing-a-development-branch`는 자동 호출이 맞는가, 아니면 사용자 명시 요청 기반이 더 적절한가

## 한눈에 보는 결론

현재 문서 기준으로는 리뷰 체계가 아래처럼 정리되어 있다.

- `self-review`, `integration verification`, `formal review`는 서로 다른 단계다.
- 3가지 실행 모드 모두 `formal review`는 `모든 task` 또는 `모든 wave`가 끝난 뒤 `최종 1회`만 수행한다.
- `requesting-code-review`는 최종 통합 결과에 대한 `formal review`를 요청할 때 쓴다.
- `receiving-code-review`는 실제 리뷰 피드백이 있을 때만 쓴다.
- `finishing-a-development-branch`는 사용자가 커밋, 머지, 푸시, PR을 명시적으로 요청했을 때만 쓴다.

현재 기준 기본 골격:

`using-superpowers -> brainstorming(필요 시) -> writing-plans -> 실행 모드 선택 -> using-git-worktrees -> 구현 진행(TDD 반복) -> 통합 검증 -> 최종 formal review 1회 -> feedback 있으면 receiving-code-review -> verification-before-completion -> 사용자가 원할 때만 finishing-a-development-branch`

조건부 분기:
- 버그, 테스트 실패, 이상 동작이 나오면 `systematic-debugging`
- 최종 formal review에서 actionable feedback이 나오면 `receiving-code-review`
- 사용자가 통합을 요청하면 `finishing-a-development-branch`

상세 변경 이력은 `SKILL_WORKFLOW_CHANGELOG_KO.md`에 따로 정리했다.

## 공통 워크플로우

### 1. 가장 먼저 보는 것

메인 세션은 먼저 `using-superpowers`를 따른다고 이해하면 된다.

의미:
- 응답이나 행동 전에 관련 skill이 있는지 먼저 확인한다.
- relevant한 skill이 있으면 반드시 불러야 한다.
- 다만 사용자 지시가 skill보다 우선한다.

예:
- 새 기능 설계 요청이면 `brainstorming`
- 버그 수정이면 `systematic-debugging`
- 구현 시작이면 `test-driven-development`

### 2. brainstorming이 붙는 경우

`brainstorming`은 보통 아래 상황에서 붙는다.

- 새 기능 추가
- 동작 변경
- 설계가 필요한 구현 작업
- 사용자가 설계 논의를 원할 때

즉, 모든 질문에 무조건 붙는 것은 아니다.

`brainstorming`의 끝은 구현이 아니라 `writing-plans` 호출이다.

### 3. writing-plans의 역할

`writing-plans`는 승인된 설계나 이미 정리된 요구사항을 바탕으로 구현 계획을 만든다.

출력 형태는 2가지다.

- 큰 작업: `docs/superpowers/plans/...` 아래 plan 문서 생성
- 작은 작업: 대화 안의 짧은 체크리스트로 대체

그 다음에는 아래 3가지 실행 모드 중 하나를 고른다.

- `Inline Execution`
- `Subagent-Driven`
- `Parallel Subagents`

### 4. 리뷰 용어 정리

현재 문서는 리뷰 관련 용어를 아래처럼 나눈다.

- `Implementer self-review`
  - task 담당자가 자기 작업을 넘기기 전에 스스로 점검하는 단계
  - 로컬 품질 점검이지 formal review가 아니다
- `Integration verification`
  - task 또는 wave를 통합한 뒤 테스트와 확인을 수행하는 단계
  - 이것도 formal review가 아니다
- `Formal review`
  - `requesting-code-review`로 `completed integrated result`를 reviewer에게 검토시키는 단계
  - 기본 시점은 모든 implementation task가 끝난 뒤 1회다

이 구분이 현재 문서의 핵심이다.

## 실행 모드별 전체 흐름

아래는 `새 세션에서 이미 plan이 있거나`, `writing-plans` 직후에 실행 모드를 고른 상황까지 포함해서 정리한 흐름이다.

### 1. Inline Execution

`using-superpowers -> executing-plans -> using-git-worktrees -> 메인 에이전트가 task를 직접 순차 실행 -> 구현마다 test-driven-development -> 필요 시 systematic-debugging -> 모든 task 완료 후 최종 formal review 1회 -> feedback 있으면 receiving-code-review -> verification-before-completion -> 사용자가 원할 때만 finishing-a-development-branch`

설명:
- 메인 에이전트가 직접 구현한다.
- task별 formal review는 하지 않는다.
- 모든 task 완료 후 `requesting-code-review`를 통해 최종 formal review를 1회 수행한다.
- `receiving-code-review`는 그 formal review가 실제 피드백을 돌려줄 때만 등장한다.
- 즉, `Inline Execution = 서브에이전트 0개`는 아니다.

적합한 경우:
- 변경 범위가 작다.
- 파일 수가 적다.
- task 간 결합이 강하다.
- 메인 세션에서 순차 처리하는 편이 낫다.

### 2. Subagent-Driven

`using-superpowers -> subagent-driven-development -> using-git-worktrees -> 메인 에이전트가 plan의 task를 하나씩 꺼냄 -> task마다 implementer subagent를 순차 호출 -> 각 subagent가 구현, 테스트, self-review 후 보고 -> 메인 에이전트가 task를 순차 진행 -> 모든 task 완료 후 최종 formal review 1회 -> feedback 있으면 receiving-code-review -> verification-before-completion -> 사용자가 원할 때만 finishing-a-development-branch`

설명:
- task별로 fresh subagent를 쓴다.
- 각 implementer의 `self-review`는 task 내부 점검일 뿐 formal review가 아니다.
- formal review는 `모든 task 완료 후` `completed integrated result`에 대해 1회만 수행한다.

적합한 경우:
- 작업이 하나의 작은 덩어리로 끝나지 않는다.
- task 분리로 이해와 제어가 쉬워진다.
- 병렬 실행까지는 위험하거나 애매하다.
- 메인 세션 컨텍스트 오염을 줄이고 싶다.

### 3. Parallel Subagents

`using-superpowers -> parallel-subagent-execution -> task를 wave로 분할 -> 각 task에 branch/worktree/file ownership 부여 -> wave 단위로 implementer subagent들을 병렬 호출 -> 각 subagent가 구현, 테스트, self-review 후 보고 -> 메인 에이전트가 wave 통합 및 verification -> 다음 wave 반복 -> 모든 wave 완료 후 최종 formal review 1회 -> feedback 있으면 receiving-code-review -> verification-before-completion -> 사용자가 원할 때만 finishing-a-development-branch`

설명:
- 단순히 서브에이전트를 많이 띄우는 방식이 아니다.
- 현재 문서는 아래를 강하게 요구한다.
- task 간 의존성 없음
- write scope 비중첩
- file ownership 명시
- task별 별도 branch/worktree
- wave별 통합 검증
- 같은 wave 안에서도 task마다 다른 모델을 선택할 수 있음
- `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, `BLOCKED` 상태를 wave 통합 전에 처리해야 함

`file ownership` 의미:
- OS 권한이나 사용자 계정 권한이 아니다.
- 각 task가 수정해도 되는 `assigned file scope`를 뜻한다.

적합한 경우:
- task가 독립적이다.
- 같은 파일을 수정하지 않는다.
- 병렬 처리 시 실제 속도 이점이 크다.
- 통합 지점이 미리 정의되어 있다.

## TDD와 검증은 어디에 들어가는가

### test-driven-development

`test-driven-development`는 일회성 절차가 아니다.

실제로는 구현 단위마다 반복된다.

반복 패턴:
- 실패하는 테스트 작성
- 실패 확인
- 최소 구현
- 테스트 통과 확인
- 필요 시 리팩터링

즉, `실행 모드 위에 얹히는 구현 규율`로 보는 것이 맞다.

### verification-before-completion

`verification-before-completion`도 마지막에 한 번 붙는 장식이 아니다.

아래 상황마다 계속 적용된다.

- 작업 완료라고 말하기 전
- 테스트가 통과했다고 말하기 전
- formal review 결과를 반영했다고 말하기 전
- 커밋 전
- PR 생성 전
- 다음 task로 넘어가며 성공을 주장하기 전

즉, `최종 단계`라기보다 `완료 주장 전 증거 확인 규칙`이다.

## 리뷰 관련 skill과 agent 정리

### requesting-code-review

리뷰를 시작할 때 사용하는 skill이다.

현재 문서 기준:
- 최종 formal review의 기본 시점은 `모든 implementation task 완료 후 1회`
- review 대상은 `completed integrated result` 또는 `full task series`
- routine per-task formal review는 기본 워크플로우가 아니다

### receiving-code-review

리뷰 피드백을 받은 뒤에 쓰는 skill이다.

현재 문서 기준:
- human partner, external reviewer, review subagent의 피드백을 기술적으로 검토할 때 사용한다
- 피드백이 실제로 있을 때만 쓴다
- formal review가 `no actionable findings`로 끝나면 별도 reception 단계는 필요 없다

즉, `requesting-code-review`의 후속으로 자주 등장하지만, 개념적으로는 `review feedback 처리 skill`이다.

### code-reviewer agent

실제 formal review를 수행하는 agent다.

현재 문서 기준 역할:
- completed integrated result 또는 full task series 검토
- diff 확인
- 테스트 확인
- plan 대비 구현 검토
- `Critical / Important / Minor` 분류

주의:
- review-only agent다
- workflow skill을 다시 호출하거나 코드를 수정하는 역할은 아니다

## finishing-a-development-branch는 언제 쓰는가

현재 문서 기준으로는 자동 호출이 아니라 `명시적 사용자 요청 기반`이다.

트리거 예시:
- 커밋해줘
- 머지해줘
- 푸시해줘
- PR 만들어줘

즉, 구현이 끝났다고 저절로 호출되는 skill이 아니다.

## 네가 원했던 방향과 현재 문서의 관계

네가 원했던 방향은 아래였다.

- 3가지 모드 모두 task별 formal review는 하지 않기
- 모든 task의 구현과 테스트가 끝난 뒤 최종 review 1회만 하기
- 속도와 안정성의 절충을 맞추기

현재 문서는 이 방향을 명시적으로 반영하도록 정리되었다.

정리하면:
- `Inline Execution`: 모든 task 완료 후 최종 formal review 1회
- `Subagent-Driven`: task별 self-review + 모든 task 완료 후 최종 formal review 1회
- `Parallel Subagents`: task별 self-review + wave별 verification + 모든 wave 완료 후 최종 formal review 1회

## 현재 남아 있는 문서상 메모

### 1. using-git-worktrees 호출 시점

문서끼리 약간의 불일치가 남아 있다.

- `brainstorming`은 설계 승인 후 바로 `writing-plans`로 넘어간다고 말한다.
- `writing-plans`는 이 작업이 dedicated worktree에서 실행된다고 적는다.
- `using-git-worktrees`는 brainstorming 단계에서 호출된다고 적는다.

실무적으로는 아래처럼 이해하는 것이 가장 자연스럽다.

`설계/계획 완료 -> 실행 모드 진입 직전 또는 구현 시작 직전 -> using-git-worktrees`

### 2. brainstorming의 frontmatter description

`writing-skills` 기준으로 보면 description은 `언제 쓰는가` 중심이어야 한다.

그런데 현재 `brainstorming` description은 `언제 사용하는가`와 `무엇을 하는가`가 섞여 있어 다소 길고 workflow 설명 성격이 있다.

즉, 검색성과 문서 규칙 측면에서는 조금 다듬을 여지가 있다.

## 최종 정리

현재 문서 기준으로 가장 정확한 이해는 아래와 같다.

- 메인 세션은 먼저 `using-superpowers`
- 설계성 작업이면 `brainstorming`
- 승인된 설계나 요구사항이 있으면 `writing-plans`
- 실행 모드 선택 후 `using-git-worktrees`
- 구현 중에는 `test-driven-development` 반복
- 버그나 실패가 생기면 `systematic-debugging`
- `self-review`, `integration verification`, `formal review`는 서로 다름
- formal review는 3가지 모드 모두 `모든 task/wave 완료 후 1회`
- `receiving-code-review`는 피드백이 실제로 있을 때만 사용
- `finishing-a-development-branch`는 사용자가 통합을 명시적으로 요청했을 때만 사용
