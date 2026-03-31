# 심층 방어 검증 (Defense-in-Depth Validation)

## 개요

잘못된 데이터로 인한 버그를 수정할 때, 한 곳에만 검증 로직을 추가하는 것으로 충분하다고 느낄 때가 많습니다. 하지만 그 단일 체크는 다른 코드 경로, 리팩토링 또는 모의 객체(mock)에 의해 우회될 수 있습니다.

**핵심 원칙:** 데이터가 통과하는 **모든** 계층에서 검증하십시오. 버그가 구조적으로 발생할 수 없도록 만드십시오.

## 왜 여러 계층인가?

단일 검증: "버그를 수정했습니다."
다중 계층 검증: "버그가 발생하는 것을 불가능하게 만들었습니다."

각 계층은 서로 다른 케이스를 포착합니다:
- 진입점 검증은 대부분의 버그를 포착합니다.
- 비즈니스 로직 검증은 예외 케이스를 포착합니다.
- 환경 가드(Guard)는 특정 문맥에서의 위험을 방지합니다.
- 디버그 로깅은 다른 계층이 실패했을 때 원인 파악을 돕습니다.

## 4계층 방어 모델

### 1계층: 진입점 검증 (Entry Point Validation)
**목적:** API 경계에서 명백히 잘못된 입력을 거부합니다.

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory는 비어 있을 수 없습니다.');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory가 존재하지 않습니다: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory가 디렉토리가 아닙니다: ${workingDirectory}`);
  }
  // ... 진행
}
```

### 2계층: 비즈니스 로직 검증 (Business Logic Validation)
**목적:** 데이터가 해당 작업에 적합한지 확인합니다.

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('워크스페이스 초기화를 위해 projectDir이 필요합니다.');
  }
  // ... 진행
}
```

### 3계층: 환경 가드 (Environment Guards)
**목적:** 특정 문맥(예: 테스트 환경)에서 위험한 작업이 수행되는 것을 방지합니다.

```typescript
async function gitInit(directory: string) {
  // 테스트 중에는 임시 디렉토리 밖에서 git init 실행을 거부함
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `테스트 중에 임시 디렉토리 외부에서 git init 실행을 거부함: ${directory}`
      );
    }
  }
  // ... 진행
}
```

### 4계층: 디버그 계측 (Debug Instrumentation)
**목적:** 사후 분석을 위해 문맥을 캡처합니다.

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('git init 실행 직전', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... 진행
}
```

## 패턴 적용하기

버그를 발견했을 때:

1. **데이터 흐름 추적** - 잘못된 값이 어디에서 시작되어 어디에서 사용되는가?
2. **모든 체크포인트 매핑** - 데이터가 통과하는 모든 지점을 나열하십시오.
3. **각 계층에 검증 추가** - 진입점, 비즈니스 로직, 환경 가드, 디버그 로깅을 추가하십시오.
4. **각 계층 테스트** - 1계층을 우회해보고, 2계층이 이를 포착하는지 확인하십시오.

## 실제 사례

버그: 빈 `projectDir`로 인해 소스 코드 폴더에서 `git init`이 실행됨

**데이터 흐름:**
1. 테스트 설정 → 빈 문자열 발생
2. `Project.create(name, '')` 호출
3. `WorkspaceManager.createWorkspace('')` 호출
4. `git init`이 `process.cwd()`에서 실행됨

**추가된 4계층 방어:**
- 1계층: `Project.create()`에서 비어 있지 않음/존재함/쓰기 가능 여부 검증
- 2계층: `WorkspaceManager`에서 projectDir이 비어 있지 않음 확인
- 3계층: `WorktreeManager`에서 테스트 중 임시 디렉토리 밖의 git init 거부
- 4계층: git init 실행 직전 스택 트레이스 로깅

**결과:** 1847개 테스트 모두 통과, 버그 재현 불가능해짐

## 핵심 통찰

4개 계층 모두가 필요했습니다. 테스트 과정에서 각 계층은 다른 계층이 놓친 버그를 포착했습니다:
- 서로 다른 코드 경로가 진입점 검증을 우회함
- 모의 객체(mock)가 비즈니스 로직 체크를 우회함
- 다양한 플랫폼의 예외 케이스 처리를 위해 환경 가드가 필요함
- 디버그 로깅이 구조적인 오용 사례를 식별함

**하나의 검증 지점에서 멈추지 마십시오.** 모든 계층에 체크 로직을 추가하십시오.
