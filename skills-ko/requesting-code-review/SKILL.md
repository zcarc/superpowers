---
name: requesting-code-review
description: 사용자가 리뷰를 요청하거나 완료된 변경 사항이 통합 전 리뷰 리스크 임계값을 초과할 때 사용합니다.
---

# 코드 리뷰 요청 (Requesting Code Review)

머지 또는 핸드오프 전 이슈를 잡기 위해 `superpowers:code-reviewer` 하위 에이전트를 발송합니다. 리뷰어는 평가를 위해 정밀하게 작성된 컨텍스트를 제공받으며, 여러분의 세션 히스토리는 절대 전달되지 않습니다. 이는 리뷰어가 여러분의 사고 과정이 아닌 작업 결과물에 집중하게 하고, 여러분은 통합 작업을 위해 자신의 컨텍스트를 유지할 수 있게 합니다.

**코어 원칙:** 의미 있는 체크포인트에서 리뷰를 수행합니다.

## 리뷰 요청 시점

**필수:**
- 하위 에이전트 기반 개발(subagent-driven development)의 모든 작업 완료 후
- 병렬 하위 에이전트 실행(parallel subagent execution)의 모든 웨이브(waves) 완료 후
- 주요 기능 구현 완료 후
- 메인 브랜치로 머지하기 전

**선택 사항이지만 가치 있는 경우:**
- 작업이 막혔을 때 (새로운 시각 필요)
- 리팩토링 전 (베이스라인 확인)
- 복잡한 버그 수정 후

## 리뷰 임계값 (Review Threshold)

다음 중 하나 이상에 해당할 때 리뷰를 요청하십시오:
- 사용자가 명시적으로 리뷰를 요청함
- 5개 이상의 파일이 변경됨
- 퍼시스턴스, 스키마 또는 저장소 로직이 변경됨
- 공유 추상화나 아키텍처가 변경됨
- 상당한 통합 또는 회귀(regression) 리스크가 있음

다음의 경우 기본적으로 리뷰를 요청하지 마십시오:
- 소규모의 로컬 버그 수정
- 격리된 디스플레이 또는 포맷팅 변경
- 텍스트/복사만 있는 변경 사항

## 요청 방법

**1. git SHA 가져오기:**
```bash
# 기본: 전체 브랜치 또는 완료된 작업 시리즈 리뷰
BASE_SHA=$(git merge-base HEAD origin/main)  # 또는 origin/master / 실제 베이스 브랜치
HEAD_SHA=$(git rev-parse HEAD)

# 단일 커밋 지점 확인용:
# BASE_SHA=$(git rev-parse HEAD~1)
```

**2. code-reviewer 하위 에이전트 발송:**

Task 도구에서 `superpowers:code-reviewer` 타입을 사용하고 `code-reviewer.md` 템플릿을 채웁니다.

**플레이스홀더:**
- `{WHAT_WAS_IMPLEMENTED}` - 방금 구현한 내용
- `{PLAN_OR_REQUIREMENTS}` - 수행해야 했던 작업
- `{BASE_SHA}` - 시작 커밋
- `{HEAD_SHA}` - 종료 커밋
- `{DESCRIPTION}` - 간략한 요약

**3. 피드백에 따른 조치:**
- 심각(Critical) 이슈는 즉시 수정하십시오.
- 진행하기 전에 중요(Important) 이슈를 수정하십시오.
- 사소한(Minor) 이슈는 나중에 처리하도록 기록해 두십시오.
- 리뷰어가 틀린 경우 근거를 가지고 반박하십시오.

## 예시

```
[모든 구현 작업 완료]

사용자: 이 브랜치를 완료하기 전에 최종 코드 리뷰를 요청하겠습니다.

BASE_SHA=$(git merge-base HEAD origin/main)  # 실제 베이스 브랜치 / 작업 시리즈 시작 SHA
HEAD_SHA=$(git rev-parse HEAD)

[superpowers:code-reviewer 하위 에이전트 발송]
  WHAT_WAS_IMPLEMENTED: 완료된 모든 작업을 통한 기능 구현
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 계획된 검증, 복구 및 보고 작업 완료

[하위 에이전트 복귀]:
  장점: 깔끔한 아키텍처, 실질적인 테스트
  문제점:
    중요(Important): 진행률 표시기 누락
    사소(Minor): 보고 간격에 매직 넘버(100) 사용
  평가: 수정 후 머지 준비 완료

사용자: [진행률 표시기 수정]
[필요한 경우 리뷰 재요청 후 브랜치 완료 요청 수행]
```

## 워크플로우 통합

**하위 에이전트 기반 개발 (Subagent-Driven Development):**
- 모든 작업 완료 후 한 번 리뷰하십시오.
- 브랜치를 완료하기 전 작업 간 이슈를 잡으십시오.
- 요청된 통합 단계가 실행되기 전에 중요한 이슈를 수정하십시오.

**플랜 실행 (Executing Plans):**
- 메인 에이전트 직접 실행의 경우 리뷰는 선택 사항이며 리스크에 기반합니다.
- 변경 사항이 위험하거나 여러 작업에 걸쳐 있는 경우 주요 배치 작업 후 리뷰하십시오.
- 완료된 변경 집합이 상당한 경우 항상 머지 전에 리뷰를 요청하십시오.

**병렬 하위 에이전트 실행 (Parallel Subagent Execution):**
- 모든 웨이브가 통합된 후 한 번 리뷰하십시오.
- 브랜치를 완료하기 전 웨이브 간 통합 이슈를 잡으십시오.
- 요청된 통합 단계가 실행되기 전에 중요한 이슈를 수정하십시오.

**임의 개발 (Ad-Hoc Development):**
- 머지 전에 리뷰하십시오.
- 막혔을 때 리뷰하십시오.

## 주의 신호 (Red Flags)

**절대 금지:**
- "간단하니까"라는 이유로 리뷰 건너뛰기
- 치명적(Critical) 문제 무시
- 수정되지 않은 중요(Important) 문제가 있는 상태로 진행
- 타당한 기술적 피드백에 대해 논쟁하기

**하지만 다음 사항도 금지:**
- 모든 인라인 버그 수정에 대해 자동으로 리뷰 발송
- 소규모의 로컬 디스플레이 또는 포맷팅 수정에 대해 리뷰를 필수로 취급

**리뷰어가 틀린 경우:**
- 기술적 근거를 바탕으로 반론을 제기하십시오.
- 정상 작동함을 증명하는 코드나 테스트를 보여주십시오.
- 명확한 설명을 요청하십시오.

템플릿 확인: `requesting-code-review/code-reviewer.md`
