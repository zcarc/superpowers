---
name: requesting-code-review
description: 작업 완료 시, 주요 기능 구현 후, 또는 머지 전 작업 내용이 요구 사항을 충족하는지 검증하기 위해 사용
---

# 코드 리뷰 요청 (Requesting Code Review)

문제가 더 커지기 전에 `superpowers:code-reviewer` 서브에이전트를 파견하여 문제를 포착하십시오. 리뷰어에게는 평가를 위한 정밀하게 구성된 문맥만 제공되며, 여러분의 세션 히스토리는 전달되지 않습니다. 이는 리뷰어가 여러분의 사고 과정이 아닌 결과물에만 집중하게 하며, 여러분은 자신의 문맥을 유지하며 작업을 계속할 수 있게 해줍니다.

**핵심 원칙:** 의미 있는 체크포인트에서 리뷰하십시오.

## 리뷰 요청 시기

**필수 사항:**
- 서브에이전트 주도 개발(subagent-driven development)의 모든 작업 완료 후
- 주요 기능 구현 완료 후
- main 브랜치로 머지하기 전

**선택 사항이지만 가치 있는 경우:**
- 작업이 막혔을 때 (새로운 시각이 필요함)
- 리팩토링 전 (기준점 확인)
- 복잡한 버그 수정 후

## 요청 방법

**1. git SHA 가져오기:**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 또는 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. code-reviewer 서브에이전트 파견:**

`superpowers:code-reviewer` 유형의 Task 도구를 사용하고, `code-reviewer.md`에 있는 템플릿을 채우십시오.

**자리 표시자(Placeholders):**
- `{WHAT_WAS_IMPLEMENTED}` - 방금 구축한 항목
- `{PLAN_OR_REQUIREMENTS}` - 수행해야 할 작업 내용
- `{BASE_SHA}` - 시작 커밋
- `{HEAD_SHA}` - 종료 커밋
- `{DESCRIPTION}` - 간략한 요약

**3. 피드백에 따른 조치:**
- **치명적(Critical)** 문제는 즉시 수정하십시오.
- **중요(Important)** 문제는 진행하기 전에 수정하십시오.
- **사소한(Minor)** 문제는 나중에 처리할 수 있도록 메모해 두십시오.
- 리뷰어가 틀린 경우 기술적 근거와 함께 반론을 제기하십시오.

## 예시

```
[모든 구현 작업 완료]

여러분: 브랜치를 마무리하기 전에 마지막으로 코드 리뷰를 요청하겠습니다.

BASE_SHA=$(git merge-base HEAD origin/main)  # 또는 작업 시리즈의 시작 SHA
HEAD_SHA=$(git rev-parse HEAD)

[superpowers:code-reviewer 서브에이전트 파견]
  WHAT_WAS_IMPLEMENTED: 완료된 모든 작업에 걸친 기능 구현
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 계획된 검증, 복구 및 보고 작업 완료

[서브에이전트 반환]:
  장점: 깔끔한 아키텍처, 실제 테스트 포함
  이슈:
    중요: 진행 상태 표시기 누락
    사소: 보고 간격을 위한 매직 넘버(100) 사용
  평가: 수정을 거치면 머지 가능

여러분: [진행 상태 표시기 수정]
[필요한 경우 리뷰를 재요청한 후, 브랜치 마무리]
```

## 워크플로우와의 통합

**서브에이전트 주도 개발 (Subagent-Driven Development):**
- 모든 작업 완료 후 한 번 리뷰를 요청하십시오.
- 브랜치를 마무리하기 전에 작업 간 교차 이슈를 포착하십시오.
- 머지하거나 브랜치를 완료하기 전에 중요한 이슈를 수정하십시오.

**계획 실행 (Executing Plans):**
- 각 배치(현 계획에서는 3개 작업) 후에 리뷰를 요청하십시오.
- 피드백을 받고 적용한 후 계속 진행하십시오.

**임의 개발 (Ad-Hoc Development):**
- 머지 전에 리뷰하십시오.
- 막혔을 때 리뷰하십시오.

## 주의 신호 (Red Flags)

**절대 금지:**
- "간단하니까"라는 이유로 리뷰 건너뛰기
- 치명적(Critical) 문제 무시
- 수정되지 않은 중요(Important) 문제가 있는 상태로 진행
- 타당한 기술적 피드백에 대해 감정적으로 대응

**리뷰어가 틀린 경우:**
- 기술적 근거를 바탕으로 반론을 제기하십시오.
- 정상 작동함을 증명하는 코드나 테스트를 보여주십시오.
- 명확한 설명을 요청하십시오.

템플릿 확인: `requesting-code-review/code-reviewer.md`
