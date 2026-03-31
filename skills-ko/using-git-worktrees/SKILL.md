---
name: using-git-worktrees
description: 현재 워크스페이스로부터 격리가 필요한 기능 개발을 시작하거나 구현 계획을 실행하기 전에 사용 - 스마트한 디렉토리 선택 및 안전 검증을 통해 격리된 git 워크트리를 생성함
---

# Git 워크트리 사용 (Using Git Worktrees)

## 개요

Git 워크트리는 동일한 저장소를 공유하는 격리된 작업 공간을 생성하여 브랜치 전환 없이 여러 브랜치에서 동시에 작업할 수 있게 해줍니다.

**핵심 원칙:** 체계적인 디렉토리 선택 + 안전 검증 = 신뢰할 수 있는 격리 환경.

**시작 시 알림:** "격리된 작업 공간을 설정하기 위해 using-git-worktrees 기술을 사용하겠습니다."

## 디렉토리 선택 프로세스

다음 우선순위에 따라 디렉토리를 선택하십시오:

### 1. 기존 디렉토리 확인

```bash
# 우선순위에 따라 확인
ls -d .worktrees 2>/dev/null     # 우선권 (숨김 폴더)
ls -d worktrees 2>/dev/null      # 대안
```

**발견된 경우:** 해당 디렉토리를 사용하십시오. 둘 다 존재한다면 `.worktrees`가 우선합니다.

### 2. CLAUDE.md 확인

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**설정이 명시된 경우:** 묻지 말고 해당 설정을 따르십시오.

### 3. 사용자에게 질문

기존 디렉토리가 없고 CLAUDE.md에도 설정이 없는 경우:

```
워크트리 디렉토리를 찾을 수 없습니다. 어디에 워크트리를 생성할까요?

1. .worktrees/ (프로젝트 로컬, 숨김)
2. ~/.config/superpowers/worktrees/<프로젝트 이름>/ (글로벌 위치)

어느 곳을 선호하십니까?
```

## 안전 검증

### 프로젝트 로컬 디렉토리 (.worktrees 또는 worktrees)의 경우

**워크트리를 생성하기 전에 해당 디렉토리가 git에서 무시(ignore)되는지 반드시 확인해야 합니다:**

```bash
# 디렉토리가 무시되는지 확인 (로컬, 글로벌, 시스템 gitignore 모두 반영)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**무시되지 않는 경우:**

Jesse의 규칙인 "고장난 것은 즉시 고쳐라(Fix broken things immediately)"에 따라:
1. .gitignore에 해당 라인을 추가하십시오.
2. 변경 사항을 커밋하십시오.
3. 그 다음 워크트리 생성을 진행하십시오.

**이유:** 워크트리 내용이 실수로 저장소에 커밋되는 것을 방지하기 위함입니다.

## 생성 단계

### 1. 프로젝트 이름 감지

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 워크트리 생성

```bash
# 전체 경로 결정
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# 새 브랜치와 함께 워크트리 생성
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 프로젝트 설정 실행

프로젝트 유형을 자동 감지하여 적절한 설정을 실행하십시오:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 깨끗한 기준선(Baseline) 확인

워크트리가 깨끗한 상태에서 시작되는지 확인하기 위해 테스트를 실행하십시오:

```bash
# 예시 - 프로젝트에 맞는 명령어를 사용하십시오
npm test
cargo test
pytest
go test ./...
```

**테스트가 실패하는 경우:** 실패 내용을 보고하고, 진행할지 아니면 조사할지 사용자에게 물어보십시오.

**테스트가 통과하는 경우:** 준비 완료를 보고하십시오.

### 5. 위치 보고

```
워크트리 준비 완료: <전체 경로>
테스트 통과 (<N>개 테스트 통과, 0개 실패)
<기능 이름>을 구현할 준비가 되었습니다.
```

## 빠른 참조

| 상황 | 조치 |
|-----------|--------|
| `.worktrees/` 존재함 | 사용 (무시 여부 확인 필수) |
| `worktrees/` 존재함 | 사용 (무시 여부 확인 필수) |
| 둘 다 존재함 | `.worktrees/` 사용 |
| 둘 다 없음 | CLAUDE.md 확인 → 사용자에게 질문 |
| 디렉토리가 무시되지 않음 | .gitignore에 추가 후 커밋 |
| 기준선 테스트 실패 | 실패 보고 후 사용자에게 질문 |
| package.json/Cargo.toml 없음 | 의존성 설치 건너띔 |

## 흔한 실수

### 무시(ignore) 확인 건너뛰기

- **문제:** 워크트리 내용이 추적되어 git 상태를 오염시킴
- **해결책:** 프로젝트 로컬 워크트리를 생성하기 전에 항상 `git check-ignore`를 사용하십시오.

### 디렉토리 위치 임의 가정

- **문제:** 일관성을 해치고 프로젝트 컨벤션을 위반함
- **해결책:** 우선순위를 따르십시오: 기존 디렉토리 > CLAUDE.md > 질문

### 테스트 실패 상태로 진행

- **문제:** 새로운 버그와 기존의 문제를 구별할 수 없게 됨
- **해결책:** 실패를 보고하고 진행에 대한 명시적인 허락을 받으십시오.

### 설정 명령어 하드코딩

- **문제:** 다른 도구를 사용하는 프로젝트에서 작동하지 않음
- **해결책:** 프로젝트 파일(package.json 등)에서 자동 감지하십시오.

## 워크플로우 예시

```
여러분: 격리된 작업 공간을 설정하기 위해 using-git-worktrees 기술을 사용하겠습니다.

[.worktrees/ 확인 - 존재함]
[무시 여부 확인 - git check-ignore로 .worktrees/가 무시됨을 확인]
[워크트리 생성: git worktree add .worktrees/auth -b feature/auth]
[npm install 실행]
[npm test 실행 - 47개 통과]

워크트리 준비 완료: /Users/jesse/myproject/.worktrees/auth
테스트 통과 (47개 테스트 통과, 0개 실패)
auth 기능을 구현할 준비가 되었습니다.
```

## 주의 신호 (Red Flags)

**절대 금지:**
- 무시 여부를 확인하지 않고 워크트리 생성 (프로젝트 로컬인 경우)
- 기준선 테스트 검증 건너뛰기
- 테스트 실패 시 묻지 않고 진행
- 모호한 상황에서 디렉토리 위치를 임의로 가정
- CLAUDE.md 확인 생략

**항상 수행:**
- 디렉토리 우선순위 준수: 기존 > CLAUDE.md > 질문
- 프로젝트 로컬의 경우 디렉토리 무시 여부 확인
- 프로젝트 설정 자동 감지 및 실행
- 깨끗한 테스트 기준선 검증

## 통합

**호출하는 기술:**
- **brainstorming** (4단계) - 설계 승인 후 구현이 이어질 때 필수
- **subagent-driven-development** - 작업 실행 전 필수
- **executing-plans** - 작업 실행 전 필수
- 격리된 작업 공간이 필요한 모든 기술

**연계 기술:**
- **finishing-a-development-branch** - 작업 완료 후 정리를 위해 필수
