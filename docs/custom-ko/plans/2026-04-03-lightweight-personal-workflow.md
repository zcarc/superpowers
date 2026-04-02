# Lightweight Personal Workflow 구현 계획 (Implementation Plan)

> **에이전트 작업자용:** 필수 하위 스킬(REQUIRED SUB-SKILL): 이 계획에 가장 적합한 실행 모드를 선택하세요: `superpowers:executing-plans`, `superpowers:subagent-driven-development` 또는 `superpowers:parallel-subagent-execution`. 체크박스 (`- [ ]`) 구문을 사용하여 단계별 추적을 수행합니다.

**목표:** `skills-custom/` 및 `agents-custom/` 전반에 걸쳐 레인(lane)을 인식하는 경량(lightweight) 개인 워크플로우를 구현하여, 작고 지엽적인 작업은 빠르게 진행하되 위험도가 높은 작업에는 안전망을 온전히 유지하도록 합니다.

**아키텍처 설계:** 워크플로우 진입점에 레인 선택을 배치하고, 기획, 실행, 리뷰 스킬 파일들에 걸쳐 레인 인식 규칙을 전파합니다. 또한 코드 리뷰를 가벼운(lightweight) 리뷰어 에이전트와 심층(deep) 리뷰어 에이전트로 분리합니다. 이 첫 번째 구현에서는 별도의 에이전트 및 프롬프트 파일을 통해 리뷰 범위를 강제합니다. 이 저장소에는 안정적인 로컬 OpenCode 모델 카탈로그가 정의되어 있지 않으므로, 제공자 종속적인 `model:` ID를 하드코딩하지 않습니다.

**기술 스택:** 마크다운 스킬 파일, 마크다운 에이전트 파일, OpenCode Task 기반 하위 에이전트(subagents), 확인을 위한 `rg` (ripgrep) 명령어

**사양(Spec):** `docs/custom-ko/specs/2026-04-03-lightweight-personal-workflow-design.md`

---

## 확정된 결정 사항 (Resolved Decisions)

- 레인 선택은 `skills-custom/using-superpowers/SKILL.md`에 직접 포함됩니다.
- Light Lane(가벼운 레인)은 전체 `brainstorming` 워크플로우를 호출하는 대신, 기본적으로 간결한 인라인(대화형) 설계 방식을 사용합니다.
- 만약 Light Lane이 이미 선택되었다면, `writing-plans`는 3가지 모드의 실행 선택기를 강제로 묻지 않습니다.
- `code-reviewer-light`가 추가되었지만, 이번 첫 구현에서는 제공자(provider)에 종속적인 `model:` 정보를 추가하지 않습니다. 이는 저장소에서 아직 안정된 로컬 모델 ID에 바인딩하도록 정의되어 있지 않기 때문입니다.
- `code-reviewer.md`는 여전히 심층 리뷰어 역할을 유지하며, 명시적으로 모델-선택 경계(model-selection boundary)에 대한 주의사항을 부여받습니다.

## 파일 구조 (File Structure)

| 파일 | 역할 (Responsibility) | 수행할 작업 (Action) |
| --- | --- | --- |
| `skills-custom/using-superpowers/SKILL.md` | 진입점 워크플로우 규칙 | 레인 선택 및 개인 워크플로우 예외 추가 |
| `skills-custom/brainstorming/SKILL.md` | 전체 설계 워크플로우 | Full Lane / 명시적인 브레인스토밍 요청 전용으로 제한 |
| `skills-custom/writing-plans/SKILL.md` | 기획 워크플로우 | Light Lane 계획을 인라인으로 처리하고 실행 모드 선택기 생략 |
| `skills-custom/executing-plans/SKILL.md` | 인라인 실행 워크플로우 | 최종 리뷰를 레인이나 리뷰 트리거에 따라 조건부로 동작하게 함 |
| `skills-custom/requesting-code-review/SKILL.md` | 리뷰 요청 워크플로우 | 가벼운 리뷰어 / 심층 리뷰어 선택 분리 및 레인-인식 기반의 리뷰 조건 추가 |
| `skills-custom/receiving-code-review/SKILL.md` | 리뷰 피드백 처리 | 가벼운 리뷰어 및 심층 리뷰어의 결과물을 모두 명확히 처리하도록 명시 |
| `skills-custom/subagent-driven-development/SKILL.md` | 순차적 서브 에이전트 실행 | Full Lane 전용으로 표시 |
| `skills-custom/parallel-subagent-execution/SKILL.md` | 병렬 서브 에이전트 실행 | Full Lane 전용으로 표시 |
| `agents-custom/code-reviewer.md` | 심층 검토 에이전트 | 명시적인 모델-선택 경계 규칙 추가 |
| `agents-custom/code-reviewer-light.md` | 신규 가벼운 범위의 검토 에이전트 | 국소적인 범위만을 검토하는 리뷰어 생성 |
| `skills-custom/requesting-code-review/code-reviewer.md` | 심층 검토 프롬프트 템플릿 | 심층 리뷰 전용이라는 경계 표기 추가 |
| `skills-custom/requesting-code-review/code-reviewer-light.md` | 신규 가벼운 범위의 검토 프롬프트 템플릿 | 국소적인 범위만을 검토하는 리뷰 템플릿을 생성 |

---

### Task 1: `using-superpowers`에 레인 선택(Lane selection) 추가하기

**대상 파일:**
- 수정: `skills-custom/using-superpowers/SKILL.md`

- [ ] **Step 1: 개인 워크플로우 프로필 섹션 추가**

`## User Instructions` 다음 섹션에 아래 내용을 정확히 추가하세요:

```markdown
## Personal Workflow Profiles

이 커스텀 포크는 두 가지의 워크플로우 레인(lane)을 지원합니다.

### Light Lane

다음 조건들을 모두 충족할 때 Light Lane을 사용하세요:
- 변경 사항이 작고 지엽적이다(local).
- 요구사항이 명확하거나, 1~2가지 질문만으로 분명해질 수 있다.
- 예상되는 수정 범위가 파일 1~2개 이내이거나 좁은 단위의 특정 영역이다.
- 스키마, 영속성(persistence), 인증(auth), 배포(deployment) 혹은 공용 컨트랙트에 대한 변경이 없다.
- 회귀(regression) 위험도가 낮다.
- 사용자가 공식(formal) 리뷰를 명시적으로 요청하지 않았다.

Light Lane 작업을 수행할 때:
- 즉각적인 코드 수정(direct action)이 허용됩니다.
- 대화 중간에 간략하게 요약하는 방식의 인라인 설계(inline design check)가 허용됩니다.
- 전체 워크플로우-스킬 체인 구동을 기본적으로 강제하지 않습니다.
- 작업 범위나 위험도가 증가할 경우 즉시 Full Lane으로 단계를 격상(escalate)시킵니다.

### Full Lane

다음 조건 중 하나라도 해당하는 경우 Full Lane을 사용하세요:
- 다수의 하위 시스템이나 넓은 범위의 파일 간 연관성이 얽혀 있다.
- 공용 추상화 객체, 퍼블릭 컨트랙트, 아키텍처, 영속성 데이터, 인증 정책 혹은 마이그레이션 로직이 변경된다.
- 요구사항이 모호하여 아키텍처 설계에 영향을 미칠 여지가 있다.
- 통합 혹은 회귀 버그에 대한 위험도가 유의미한 수준이다.
- 사용자가 공식 리뷰(formal review)를 명확하게 요청했다.

Full Lane 작업의 경우 기존의 일반적인 스킬 통제(skill-first workflow) 규칙이 동일하게 적용됩니다.
```

- [ ] **Step 2: 절대적인 룰 문구를 레인-인식 기반의 문구로 교체**

현재 `## The Rule` 하위의 문구를 다음 텍스트로 정확히 대체하세요:

```markdown
## The Rule

Full Lane 작업의 경우 **절대로 응답이나 작업을 진행하기 전에 관련된 스킬이나 요청받은 스킬을 먼저 호출(Invoke)**하세요.

다만 이 커스텀 포크에서는, 명백하게 지엽적이고 리스크가 적은 Light Lane 작업일 경우 무조건 전체 워크플로우-스킬 체인을 강제하지 않으며, 즉시 문제 수정을 진행하거나 간결한 인라인 설계만을 진행할 수 있습니다. 만약 작업 도중 범위가 커져 지엽적이고 리스크가 적다는 조건에서 벗어난다면, 즉시 Full Lane으로 작업 수준을 올려 진행하세요.
```

- [ ] **Step 3: 스킬 우선도 예시 업데이트**

`## Skill Priority` 항목의 두 가지 예시 라인을 아래와 같이 변경하세요:

```markdown
"Let's build X" -> 대상 작업이 명백히 Light Lane의 인라인 설계 과정에 적합하지 않다면, Full Lane 기준에 따라 가장 먼저 brainstorming 스킬을 호출합니다.
"Fix this bug" -> 실제 버그라면 가장 먼저 debugging 스킬을 호출하며, 작업이 Light Lane에서 안전하게 처리되지 않는 수준이라면 그 후 도메인별 스킬을 활용합니다.
```

- [ ] **Step 4: 레인 모델에 맞추어 흐름도 및 Red-flag 내용 수정**

`skill_flow` 다이어그램에서, `"User message received"` 부터 `"Might any skill apply?"` 까지의 직접 연결 라인을 다음과 같은 분기로 교체하세요.

```dot
    "User message received" -> "Light Lane?";
    "Light Lane?" [shape=diamond];
    "Light Lane?" -> "Respond (including clarifications)" [label="yes - direct action or inline design"];
    "Light Lane?" -> "Might any skill apply?" [label="no - Full Lane"];
```

Red Flags 표에서 아래 행을:

```markdown
| "The skill is overkill" | Simple things become complex. Use it. |
```

다음과 같이 대체하세요:

```markdown
| "The skill is overkill" | 만약 작업이 진정으로 확실한 Light Lane에 해당한다면 인라인 방식(inline path)을 진행하세요. 그렇지 않다면 스킬을 사용하세요. |
```

- [ ] **Step 5: 레인 선택 진입점 확인**

다음 검증 명령어를 실행하세요:

```bash
rg -n "Personal Workflow Profiles|Light Lane|Full Lane|brief inline design check" skills-custom/using-superpowers/SKILL.md
```

예상 결과:
- `## Personal Workflow Profiles` 1회 매치
- `### Light Lane` 1회 매치
- `### Full Lane` 1회 매치
- `brief inline design check` 1회 매치

- [ ] **Step 6: 커밋**

```bash
git add skills-custom/using-superpowers/SKILL.md
git commit -m "docs(using-superpowers): add lane-aware personal workflow entrypoint"
```

---

### Task 2: 설계 및 기획 워크플로우에 Light-Lane 인식 적용

**대상 파일:**
- 수정: `skills-custom/brainstorming/SKILL.md`
- 수정: `skills-custom/writing-plans/SKILL.md`

- [ ] **Step 1: `brainstorming` 스킬에 Full-Lane 경계 추가**

도입부 단락과 `<PRECONDITION>` 사이에 다음 새로운 섹션을 삽입하세요:

```markdown
## Boundary

이 스킬은 Full Lane에 해당하는 작업이거나, 사용자가 브레인스토밍의 조력을 명시적으로 원할 때만 호출해야 합니다.

Light Lane 작업의 경우, 기본적으로 이 스킬을 호출하지 마세요. 대신 일반적인 대화 맥락 안에서 짧은 인라인 설계(inline design) 요약만을 작성하세요.
```

- [ ] **Step 2: `brainstorming`의 하드-게이트(hard-gate) 대체**

현재 존재하는 `<HARD-GATE>` 블록을 다음 내용으로 대체하세요:

```markdown
<HARD-GATE>
이 스킬이 호출되었다면, 사용자가 설계를 완전히 승인하기 전까지는 **절대로** 실제 구현 관련 스킬을 호출하거나, 어떠한 코드를 작성하거나, 프로젝트 구조 모양새를 만들거나(scaffold), 어떤 구현 행동도 먼저 취하지 마세요.

단, Light Lane 작업의 경우 전체 브레인스토밍 워크플로우를 호출하지 않고 `using-superpowers`에 정의된 요약된 인라인 설계 방식(inline design path)을 사용할 수 있습니다.
</HARD-GATE>
```

- [ ] **Step 3: `brainstorming`의 Anti-pattern 섹션 대체**

`## Anti-Pattern: "This Is Too Simple To Need A Design"` 항목 및 해당 단락을 다음과 같이 바꾸세요:

```markdown
## Anti-Pattern: Invoking Full Brainstorming For Obviously Light Work

우리의 문제는 '설계(Design)를 전적으로 생략하는 것' 자체보다, 아예 '생각하는 일(Thinking)을 생략하는 것'에 있습니다.

Light Lane 작업의 경우 설계 과정 자체가 짧고 대화 형식인 인라인 구조 안에서 진행될 수 있습니다.
그러나 Full Lane 작업의 경우 무조건 이 전체 브레인스토밍 워크플로우 절차를 밟아야 합니다.
```

- [ ] **Step 4: `writing-plans`에 레인 인식 기획 방법 추가**

`## Small-Scope Exception` 부분 이후에 아래 섹션을 넣으세요:

```markdown
## Lane-Aware Planning

만약 이미 Light Lane이 선택된 상황이라면:
- 기본적으로 계획문서 작성을 인라인 상태로 둡니다.
- 별도로 범위가 격상되지 않은 이상 Small-Scope Exception 모드를 적용하세요.
- 세 가지 기능 중 어떤 실행 모드(execution chooser)를 고를지 사용자에게 제시하지 않습니다.
- 진지한 위험이 가중되어 권한 확장이 필요하지 않은 한, 단순 인라인 실행(Inline Execution)을 기본으로 권유합니다.

만약 환경이 Full Lane으로 진행되고 있는 경우:
- 기존에 정의된 공식적인 plan document 파일 구성 워크플로우를 따릅니다.
- 비용이나 정책적 안전 측면에서 어떤 실행 모드를 선택할지가 유의미하게 차이가 발생할 때에만 실행 모드 선택기를 제시합니다.
```

- [ ] **Step 5: `writing-plans`의 `Execution Handoff` 첫 두 문단 변경**

해당 섹션의 초입부를 아래와 같이 수정하세요:

```markdown
## Execution Handoff

Light Lane이 이미 설정되었을 경우, 사용자에게 세 가지 실행 옵션 모드(three-mode execution chooser)를 묻지 마세요. 만일 작업이 격상(escalate)되지 않았다면, 작성해 둔 인라인 작업 목록(inline task list) 혹은 이미 저장해 둔 plan 문서를 Inline Execution용 산출물에 준하게 취급하세요.

Full Lane이 선택되어 있고 공식 plan 문서를 생성하고 저장했다면, 그 plan을 철저히 검토하고 가장 이상적인 동작의 실행 옵션 모드를 추려 분석한 뒤 사용자에게 어떤 것을 고를지 제안해 주세요.

Full Lane이 선택되었지만 작은 범위에 해당하는 작업 단위들을 담은 소그룹 작업 목록만 만들었다면, 이 목록 자체를 계획 생성 산출물로 간주해 독립적으로 올바른 실행 모드를 제안하고 넘어가세요.
```

- [ ] **Step 6: 기획 및 설계의 경계 조건들 스크립트 기반 검증하기**

다음을 실행합니다:

```bash
rg -n "## Boundary|Light Lane work|Lane-Aware Planning|do not present the three-mode execution chooser" skills-custom/brainstorming/SKILL.md skills-custom/writing-plans/SKILL.md
```

예상 결과:
- `brainstorming`에 `## Boundary` 섹션이 매치되어야 합니다
- 두 파일 종합하여 여러 개의 `Light Lane work` 텍스트 매치
- `## Lane-Aware Planning` 매치
- `do not present the three-mode execution chooser` 텍스트 매치

- [ ] **Step 7: 커밋**

```bash
git add skills-custom/brainstorming/SKILL.md skills-custom/writing-plans/SKILL.md
git commit -m "docs(workflow): make brainstorming and planning lane-aware"
```

---

### Task 3: 실행(Execution) 및 코드 리뷰 응답(receiving) 과정을 레인 인식 구조로 확장하기

**대상 파일:**
- 수정: `skills-custom/executing-plans/SKILL.md`
- 수정: `skills-custom/receiving-code-review/SKILL.md`
- 수정: `skills-custom/subagent-driven-development/SKILL.md`
- 수정: `skills-custom/parallel-subagent-execution/SKILL.md`

- [ ] **Step 1: `executing-plans`의 Task 종료 부분(Step 3)을 레인 인식을 하도록 변경하기**

`### Step 3: Final Formal Review Gate`와 그 내부의 리스트 아이템들을 아래 텍스트들로 변경합니다:

```markdown
### Step 3: Finish According To Lane

모든 태스크가 끝나고 자체 검증이 완료된 시점에 적용할 후속 규칙입니다:

- 수행한 작업이 Light Lane이었고 리뷰를 요청해야 할 어떤 트리거도 발견되지 않았다면:
  - 지역적(local) 자체 검사/테스트를 한 번 더 거칩니다.
  - 검증에 대한 증거 문서를 짧게 요약합니다.
  - 별도의 공식 코드 리뷰(formal review) 과정 없이 즉시 진행(완료)합니다.

- 수행한 변경사항이 Full Lane이었거나, 혹은 사용자가 명확하게 리뷰를 바랐거나, 코드 리뷰를 해야 하는 특정 임계치(thresholds)에 부합했다면:
  - 사용자에게 알림: "구현이 완료된 코드를 검토받기 위해 requesting-code-review 스킬을 로드하고 사용하겠습니다."
  - 그다음, 구현 및 통합된 최종 결과물에 대하여 `superpowers:requesting-code-review` 스킬을 사용하여 리뷰를 수행합니다.
  - 리뷰 후 도출된 피드백이 존재하면, `superpowers:receiving-code-review` 스킬을 응용해 도출된 이슈들을 먼저 평가하고 Important 또는 Critical 단계의 결함을 처리합니다.
  - 특별히 실행해야 할 지적 사항이나 피드백 결과가 없다면 계속(완료) 진행하세요.
```

- [ ] **Step 2: `executing-plans`의 Integration 섹션 업데이트**

두 가지 리뷰 관련 항목을 다음과 같이 변경하세요:

```markdown
- **superpowers:requesting-code-review** - Full Lane 작업이거나 리뷰 임계치(thresholds)에 도달했을 때 사용합니다.
- **superpowers:receiving-code-review** - 가벼운(light) 또는 심층(deep) 리뷰에서 평가가 필요한 피드백이 반환되었을 때 사용합니다.
```

- [ ] **Step 3: `receiving-code-review`에서 가벼운 리뷰어와 심층 리뷰어의 결과물 명확히 하기**

현재 `a review subagent`라고 되어 있는 `## Scope Restriction`의 목록 항목을 다음과 같이 변경하세요:

```markdown
- `code-reviewer` 또는 `code-reviewer-light`와 같은 리뷰 하위 에이전트(review subagent)
```

- [ ] **Step 4: 여러 개의 서브 에이전트 관련된 워크플로우가 오직 Full Lane 전용임을 명확히 하기**

`skills-custom/subagent-driven-development/SKILL.md` 문서 안의 `**Boundary:**` 단락 바로 뒷줄에 아래 한 줄을 추가하세요:

```markdown
**Lane:** 이 워크플로우 구조는 오로지 Full Lane 환경에서만 허용됩니다. 대상 목표 범위가 매우 작고 제한적이라면 메인 에이전트의 인라인 실행(inline execution)을 대신 사용하세요.
```

`skills-custom/parallel-subagent-execution/SKILL.md` 문서 안의 `**Boundary:**` 단락 바로 뒷줄에도 아래 한 줄을 추가하세요:

```markdown
**Lane:** 이 워크플로우 구조 역시 항상 Full Lane 환경에서만 허용됩니다. 타겟이 아주 단순하거나 지엽적이라면 병렬 서브 에이전트를 동원하지 마세요.
```

- [ ] **Step 5: 레인 인식 실행 경계 검증**

터미널에서 호출:

```bash
rg -n "Finish According To Lane|code-reviewer-light|This workflow is Full Lane only" skills-custom/executing-plans/SKILL.md skills-custom/receiving-code-review/SKILL.md skills-custom/subagent-driven-development/SKILL.md skills-custom/parallel-subagent-execution/SKILL.md
```

예상 결과:
- `Finish According To Lane` 1회 매치
- `code-reviewer-light` 1회 매치
- 각 서브에이전트 별 워크플로우마다 `This workflow is Full Lane only` (또는 해당 번역문) 각각 매치

- [ ] **Step 6: 커밋 사항 실행하기**

```bash
git add skills-custom/executing-plans/SKILL.md skills-custom/receiving-code-review/SKILL.md skills-custom/subagent-driven-development/SKILL.md skills-custom/parallel-subagent-execution/SKILL.md
git commit -m "docs(workflow): add lane-aware execution and review boundaries"
```

---

### Task 4: 추가적인 레인 인식 기반의 코드 검토 에이전트들을 생성

**대상 파일:**
- 수정: `skills-custom/requesting-code-review/SKILL.md`
- 수정: `skills-custom/requesting-code-review/code-reviewer.md`

- [ ] **Step 1: `requesting-code-review`에 Review Lanes 섹션 추가**

`## Review Threshold` 이후에 다음 섹션을 삽입하세요:

```markdown
## Review Lanes

- **Light Review** - 깊고 공식적인 리뷰는 불필요하지만, 작고 지엽적이며 위험도가 낮은 변경 사항에 대해 리뷰가 유용할 때 사용합니다.
- **Deep Formal Review** - Full Lane 작업이거나, 아키텍처, 컨트랙트, 영속성 데이터에 대한 변경이 있거나, 유의미한 회귀 버그 위험이 있을 때 사용합니다.

## Reviewer Selection

다음 조건들을 모두 만족할 때 `superpowers:code-reviewer-light`를 사용하세요:
- 리뷰 범위가 파일 1~2개 이내이거나 좁은 단위의 특정 영역이다.
- 변경 사항이 영속성 데이터, 인증 정책, 마이그레이션 혹은 공용 컨트랙트를 건드리지 않는다.
- 목표가 지엽적인 정확성 확보, 요구사항의 일치 확인, 수정된 영향을 받는 테스트 코드들의 충분성 점검일 때.

다음 조건 중 하나라도 만족할 때 `superpowers:code-reviewer`를 사용하세요:
- 작업이 Full Lane에 해당할 때.
- 리뷰 범위가 여러 파일에 걸쳐 있거나 일련의 통합된 태스크일 때.
- 아키텍처, 공용 추상화, 퍼블릭 컨트랙트, 영속성 데이터, 인증 정책, 혹은 마이그레이션 로직이 변경되었을 때.
- 사용자가 공식 리뷰(formal review)를 명시적으로 원할 때.
```

- [ ] **Step 2: 필수 리뷰 목록을 Full-Lane 전용 문구로 대체**

`**Mandatory:**` 목록을 다음 내용으로 변경하세요:

```markdown
**Required for Deep Formal Review:**
- `executing-plans`에서 Full Lane 작업을 마친 후
- `subagent-driven-development`의 모든 태스크가 완료된 후
- `parallel-subagent-execution`의 모든 웨이브가 완료된 후
- Full Lane 리뷰 조건(triggers)을 만족하여 병합(merge)을 진행하기 전
```

- [ ] **Step 3: `How to Request`에 명시된 디스패치 지침 교체**

`## How to Request` 아래의 Step 2를 다음 내용으로 대체하세요:

```markdown
**2. Select and dispatch the review agent (리뷰 에이전트 선택 및 병렬 호출):**

- Light Review를 위해서는 `code-reviewer-light.md` 템플릿을 채우고 `superpowers:code-reviewer-light`를 함께 호출하는 단위의 Task 도구를 사용하세요.
- Deep Formal Review를 위해서는 `code-reviewer.md` 템플릿을 채워 `superpowers:code-reviewer`에게 Task 요청을 넘기는 도구를 사용하세요.
```

- [ ] **Step 4: 레드-플래그(red-flag) 문구를 필수 심층 리뷰에만 해당하도록 축소**

`## Red Flags` 아래의 첫 번째 `Never:` 항목을 다음과 같이 변경하세요:

```markdown
- "작업이 너무 쉽다"는 이유로 꼭 진행되어야 하는 Full Lane 전용 공식 리뷰 생략하기
```

- [ ] **Step 5: `Integration with Workflows` 하위의 `Executing Plans` 서브섹션 업데이트**

현재 존재하는 `**Executing Plans:**` 하단의 내용들을 아래와 같이 대체하세요:

```markdown
**Executing Plans:**
- 작업이 Full Lane에 해당하거나 리뷰 임계치(thresholds)가 충족된 경우, 모든 작업이 완료된 후에 항상 전체 리뷰를 수행하세요.
- 좁고 지역적(local) 범위의 점검에는 `code-reviewer-light`를 이용하고, 심층적인 공식 리뷰(deep formal review)에는 `code-reviewer`를 활용하세요.
- Light Lane 작업에서는 기본적으로 심층적인 공식 리뷰 과정을 디스패치하지 않습니다.
```

- [ ] **Step 6: 기존의 심층 프롬프트 파일에 심층-리뷰 전용이라는 경계 명시 추가**

`skills-custom/requesting-code-review/code-reviewer.md` 바로 파일 상단부 `# Code Review Agent` 첫 라인 바로 밑에 이 줄을 추가합니다:

```markdown
이 프롬프트 구문은 오로지 심층적인 공식(deep formal) 형태의 리뷰어를 위한 전용 가이드라인입니다.
`code-reviewer-light`가 아닌 `superpowers:code-reviewer`와 함께 사용하세요.
```

- [ ] **Step 7: 리뷰 에이전트 분기 디스패치 관련 증거 검증하기**

명령어 실행:

```bash
rg -n "Review Lanes|Reviewer Selection|code-reviewer-light|심층적인 공식.*형태의 리뷰어를 위한 전용 가이드라인" skills-custom/requesting-code-review/SKILL.md skills-custom/requesting-code-review/code-reviewer.md
```

예상 결과:
- `## Review Lanes` 1회 매치 발견됨.
- `## Reviewer Selection` 1회 매치.
- `code-reviewer-light` 키워드 다수 매치.
- `심층적인 공식...형태의 리뷰어를 위한 전용 가이드라인` 문장 매치됨

- [ ] **Step 8: 커밋 처리**

```bash
git add skills-custom/requesting-code-review/SKILL.md skills-custom/requesting-code-review/code-reviewer.md
git commit -m "docs(review): split light and deep review dispatch rules"
```

---

### Task 5: 가벼운 리뷰어용 생성물(artifacts)을 생성하고 심층 리뷰어의 경계 업데이트

**대상 파일:**
- 수정: `agents-custom/code-reviewer.md`
- 생성: `agents-custom/code-reviewer-light.md`
- 생성: `skills-custom/requesting-code-review/code-reviewer-light.md`

- [ ] **Step 1: 심층 리뷰어(deep reviewer)에 대한 모델-선택(model-selection) 경계 명시**

`agents-custom/code-reviewer.md` 내부의 `## Operating Constraints` 하단 끝부분에 아래 설명 단락을 추가하세요:

```markdown
## Model Selection Boundary

이 에이전트는 사용할 대상 모델을 스스로 선택하지 않습니다.
이 에이전트를 호출(dispatch)하는 워크플로우 측에서 리뷰어 유형 및 정적으로 바인딩된 특정 모델을 직접 선택합니다.
```

- [ ] **Step 2: `agents-custom/code-reviewer-light.md` 파일 생성하기**

정확히 아래 콘텐츠만을 포함하는 파일을 생성하세요:

```markdown
---
name: code-reviewer-light
description: |
  핸드오프 또는 시스템 통합 전, 작고 지엽적이며 위험도가 낮은 작업 내용에서 명백한 논리적 버그나 변경부 내의 불일치 여부, 테스트 수정의 적절성을 짧게 집중적으로 체크하고자 할 때 사용하는 에이전트입니다.
---

당신은 작고 지엽적(local) 범위의 작업에 한해 코드를 점검하는 가벼운(lightweight) 코드 리뷰어 에이전트입니다.

## Operating Constraints

당신은 오로지 코드 리뷰 관련 작업만(review-only) 수행하는 에이전트입니다.

해야 할 일 (Do):
- 수정된 내용 전반의 diff 내역 확인
- 작업 결과를 명시된 계획(제시되었던 요구사항)과 비교 대조
- 변경에 따른 부가 테스트 케이스 및 검증 증거 확인
- 명백한 지역적 회귀(regressions) 버그를 플래그 처리
- 도출된 결과물(findings) 내용물 반환하기

하지 말아야 할 일 (Do NOT):
- 리뷰 중인 코드들의 총합이 명백히 상위 확장을 요구하지 않는 한, 아키텍처 점검이나 운영 준비 상태 검증 등으로 리뷰 범위를 자의적으로 과도하게 확대하지 말 것
- 또다른 작업계획 워크플로우들을 호출하지 말 것
- 메인 진행을 돕는 또다른 에이전트(agents)나 하위 태스크용 에이전트(subagents)를 디스패치하지 말 것
- 플랜(plans) 문서 작성하지 말 것
- 브랜치 작업완료 및 커밋 등의 직접적인 행동을 개시하지 말 것
- 명확한 요구 지시가 없다면 코드를 직접 수정하지 말 것

## Review Focus

1. **Scope Match (작업 범위 점검)**
   - 요구받았던 범위(local scope) 내에서 작업 코드가 벗어나진 않았는가?
   - 명백하게 불필요한 범위적 과잉 생산(scope creep)이 있지 않은가?

2. **Local Correctness (지엽적 정확성)**
   - 변경이 반영된 로직 구조가 원래 주어졌던 요구 조건과 정확히 일치하는 동작인가?
   - 이 수정 구간에서 미처 파악하지 못한 허술한 엣지 케이스(edge case) 오류를 간과하진 않았는가?

3. **Touched Tests Or Verification (관련 로직들의 테스트 상황 증명)**
   - 변경된 동작들이 근처에 존재하는 테스트 코드들이나 특정 항목의 집중 점검 조건으로 무사히 평가를 통과를 보장하는 내용인가?
   - 이 지엽적 변경 사항들에 대하여 제시되었던 결과물이나 증명 증거가 완전히 신뢰할 수 있게 서술되어 있는가?

4. **Escalation Check (단계 격상 여부 점검)**
   - 애초에 해당 코드 변경량(diff)이나 구조 변화가 생각처럼 작거나 지엽적(local)이지도 않고 리스크 위험성이 낮다기도 애매한 규모라면, 확실하게 그를 고지하고 심층 공식 리뷰(deep formal review)로 단계를 격상해야 함을 권고하십시오.

## Output

다음 내용을 반환하세요:
- Strengths (강점, 잘된 점)
- Critical, Important, Minor 등급으로 분류된 지엽적 이슈 (Issues)
- 이 변경 사항들에 대해 심층 공식 리뷰(deep formal review)로 넘길 지를 권고할 것인지 여부 서술 
- 짧게 재구성된 핵심 결과 정리안
```

- [ ] **Step 3: `skills-custom/requesting-code-review/code-reviewer-light.md` 생성하기**

아래 내용을 정확히 복사해 기재된 파일을 생성하세요:

```markdown
# Lightweight Code Review Agent

당신은 지엽적(local) 코드의 정확성을 위해 작고 부담 없는 규모, 위험도가 낮은 변경된 코드의 코드 리뷰를 수행합니다.

**Your task:**
1. 대상 구현 파악: {WHAT_WAS_IMPLEMENTED}
2. 주어진 원 계획서와 비교: {PLAN_REFERENCE}
3. 국소적인 지엽 범위에서의 로직 정확성 판단, 작업 스코프에의 한정성, 주변부의 코드 증명/테스트 여부 확인
4. 결함을 위험도(severity)별로 정렬
5. 이 코드 변경사항 건에 대한 심층(deep) 리뷰어 연계 여부 및 에스컬레이션(escalated) 판단

아래의 Git 구간 범위를 전체 변경 비교 대조군으로 삼으세요. 현재의 본 프로세스는 절대 심도깊게 시스템을 파는 공식 아키텍처 리뷰 과정이 아닌, 단순히 가벼운(lightweight) 검토용 리뷰 절차임을 잊어선 안 됩니다.

## What Was Implemented

{DESCRIPTION}

## Requirements/Plan

{PLAN_REFERENCE}

## Git Range to Review

**Base:** {BASE_SHA}
**Head:** {HEAD_SHA}

```bash
git diff --stat {BASE_SHA}..{HEAD_SHA}
git diff {BASE_SHA}..{HEAD_SHA}
```

## Review Checklist

**Scope:**
- 실제로 변경 점들이 사전에 가이드라인화 된 허용 범위 내에서 이루어진 수준들인가?
- 원 범주 외의 곳을 과도하게 무단 침범한 흔적이 없는가?

**Correctness:**
- 작업된 로직이 주어진 기본 요구 사양과 맞아 떨어지는 구현 형태인가?
- 손 댄 코드들 내에 당연히 커버해야 했음에도 깜빡 놓친 눈에 띄는 가장자리 상황(edge cases)의 예외사항을 놓친 흔적은 있는가?

**Tests / Verification:**
- 바꾼 시스템 파트 로직이나 행동들에 대해 이를 직, 간접적으로 뒷받침하며 확인하는 근막 수준의 테스트, 검증 점검 데이터들이 마련되어 작동되는 상황인가?
- 로그 결과 내용 등이 현재 이 지엽적인 단위 수정 범위에서의 안정성을 보증할 수 있는 충분한 입증 결과 역할을 하는가?

**Escalation:**
- 코드들의 차이점 크기가 처음 기대했던 것과 달리 매우 방대하고 큰 구조에 위험성을 지니며 구조적인(architectural) 수준인가?

## Output Format

### Strengths
[전체적으로 잘 만들어진 지점을 평가. 가급적 명확히 할 것.]

### Issues

#### Critical (Must Fix)
[당장 빌드 오류나 로컬 런칭을 심각하게 손상시킬 수 있는 내용, 명백한 회귀 결점, 대놓고 심각성을 안고 있는 수준]

#### Important (Should Fix)
[애초의 설계 조건 및 요구 명세와의 동작 불일치성, 관련 영역의 단위테스트 등 부재, 확실히 논리적 수준의 하자]

#### Minor (Nice to Have)
[사소한 구조 고침 및 코드 클리닝 수준]

### Escalation

**Escalate to deep formal review? (공식적인 심층적인 리뷰어에게 전달 필요합니까?)** [Yes/No]

**Why:** [1~2문장으로 이유 기술]

### Assessment

**Ready to continue? (다음 단계를 계속할 준비가 됩니까?)** [Yes/With fixes/No]

**Reasoning:** [1~2 구절로 이루어진 기술 평가 요약]
```

- [ ] **Step 4: 새로 생성한 문서(artifacts) 관련 문자열 스크립트 기반 검증 실행하기**

명령어 수행:

```bash
rg -n "code-reviewer-light|가벼운\\(lightweight\\) 코드 리뷰어 에이전트|공식적인 심층적인 리뷰어에게 전달 필요합니까" agents-custom/code-reviewer.md agents-custom/code-reviewer-light.md skills-custom/requesting-code-review/code-reviewer-light.md
```

예정 결과:
- `agents-custom/code-reviewer.md` 단일 내에서 사용할 대상 모델을 스스로 선택하지 않습니다 매치.
- `agents-custom/code-reviewer-light.md` 에 "가벼운(lightweight) 코드 리뷰어 에이전트입니다" 매치.
- `skills-custom/requesting-code-review/code-reviewer-light.md` 내부에서 "공식적인 심층적인 리뷰어에게 전달 필요합니까" 검색됨.

- [ ] **Step 5: 커밋 처리**

```bash
git add agents-custom/code-reviewer.md agents-custom/code-reviewer-light.md skills-custom/requesting-code-review/code-reviewer-light.md
git commit -m "feat(review): add lightweight reviewer artifacts for personal workflow"
```

---

### Task 6: 파일 전반에 걸친 일관성 점검 진행 (Run a cross-file consistency sweep)

**대상 파일:**
- (필요시) 수정: `skills-custom/using-superpowers/SKILL.md`
- (필요시) 수정: `skills-custom/brainstorming/SKILL.md`
- (필요시) 수정: `skills-custom/writing-plans/SKILL.md`
- (필요시) 수정: `skills-custom/executing-plans/SKILL.md`
- (필요시) 수정: `skills-custom/requesting-code-review/SKILL.md`
- (필요시) 수정: `skills-custom/receiving-code-review/SKILL.md`
- (필요시) 수정: `skills-custom/subagent-driven-development/SKILL.md`
- (필요시) 수정: `skills-custom/parallel-subagent-execution/SKILL.md`
- (필요시) 수정: `agents-custom/code-reviewer.md`
- (필요시) 수정: `agents-custom/code-reviewer-light.md`
- (필요시) 수정: `skills-custom/requesting-code-review/code-reviewer.md`
- (필요시) 수정: `skills-custom/requesting-code-review/code-reviewer-light.md`

- [ ] **Step 1: 전체 변경 대상의 파일 군에 대한 레인(Lane) 관련 용어 및 코드 리뷰 참조 텍스트 존재 확인하기**

터미널 입력:

```bash
rg -n "Light Lane|Full Lane|code-reviewer-light|deep formal review|three-mode execution chooser" skills-custom agents-custom
```

기대 검증 결과:
- `Light Lane`: 진입 단계(entry), 계획서(planning), 처리 단계(execution) 등의 파일 세트에서 등장합니다.
- `Full Lane`: 프로세스 진입, 하부 에이전트 스크립트, 리뷰 처리 파일 전반에 나타납니다.
- `code-reviewer-light`: 신규 리뷰 에이전트 정보 및 시스템 요청 호출부 모두에서 발견됩니다.
- `deep formal review` (또는 심층 공식 리뷰에 관련된 문구): 깊이 심도있는 분석 전용 프롬프트 및 가벼운 코드 리뷰 도중 스펙 격상 안내 과정 안에서 발견됩니다.
- `three-mode execution chooser`: 해당 단어 표현은 `writing-plans` 에 단독으로 국한하여 노출이 되어야 합니다.

- [ ] **Step 2: 기술된 전반적 단어 통일성 관련해서 모든 적용 대상의 파일을 한 번 더 육안 리딩 점검**

아래 사항을 확실하게 체크하십시오:
- Light Lane 은 철저하게 작고 범위가 국한적인 작업 (small, local), 위험성이 적고 안전한 작업 (low-risk) 으로 규정되고 다뤄집니까?
- Full Lane 은 항시 큰 범위 혹은 아키텍처 수준, 위험이 도사리거나 사용자가 의도를 띠고 명확하게 리뷰 단계로 통과시킬 경우에 적용됨으로 기술되었습니까?
- `requesting-code-review` 문서에서 더 이상 심층적인 리뷰만이 무조건 모든 작업을 대상으로 치러야 하는 과정으로 강요하지 않음을 확인했습니까.
- `executing-plans` 과정의 최종 처분 단계 서술의 텍스트가 무조건 공식 리뷰 완료를 강요하고 얽매고 있지 않음을 살펴보았습니까.
- `brainstorming` 스킬도 모든 수정 지점마다 처음의 기나긴 전체 브레인스토밍 워크플로우를 타야만 함을 못박아 단언하지 않도록 변경되었습니까.

- [ ] **Step 3: 최후 일관성 교정의 마무리 작성 편집 진행**

변경 사항 중 새로운 이 레인 관리 모델과 상충하는 표현이 혹시 있다면, 해당 문자 표현 부분만 즉각 고치십시오. 절대로 스펙에 미리 할당되지 않았던 나만의 상상으로 새로운 개념에 대한 워크플로우를 문장으로 추가해 구상하지는 마십시오.

- [ ] **Step 4: 모든 검증 처리 후 커밋**

이전 Step 3 과정을 통해 직접적인 파일 조작 수정 사항이 수반되었다면, 아래와 같이 커밋 반영하세요:

```bash
git add skills-custom agents-custom
git commit -m "docs(workflow): normalize lane terminology across custom skills"
```

반면 아무 손을 대지 않으셔도 되었다면, 상기 서술된 본 스크립트 커밋 단계를 생략하세요.

