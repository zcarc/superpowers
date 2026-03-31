# 비주얼 컴패니언 가이드

목업, 다이어그램 및 옵션을 보여주기 위한 브라우저 기반 시각적 브레인스토밍 컴패니언입니다.

## 사용 시점

세션별이 아닌 질문별로 결정하세요. 기준은: **사용자가 읽는 것보다 보는 것이 더 잘 이해되는가?** 입니다.

콘텐츠 자체가 시각적인 경우 **브라우저를 사용**하세요:

- **UI 목업** — 와이어프레임, 레이아웃, 탐색 구조, 컴포넌트 디자인
- **아키텍처 다이어그램** — 시스템 컴포넌트, 데이터 흐름, 관계 맵
- **나란히 배치된 시각적 비교** — 두 가지 레이아웃, 색상 구성, 디자인 방향 비교
- **디자인 폴리싱** — 모양과 느낌, 간격, 시각적 계층 구조에 대한 질문
- **공간적 관계** — 상태 머신, 순서도, 다이어그램으로 렌더링된 엔티티 관계

콘텐츠가 텍스트 또는 표 형식인 경우 **터미널을 사용**하세요:

- **요구 사항 및 범위 질문** — "X는 무엇을 의미하나요?", "어떤 기능이 범위 내에 있나요?"
- **개념적 A/B/C 선택** — 단어로 설명된 접근 방식 중 선택
- **장단점 목록** — 장단점, 비교표
- **기술적 결정** — API 디자인, 데이터 모델링, 아키텍처 접근 방식 선택
- **명확한 질문** — 답이 시각적 선호도가 아닌 단어인 모든 것

UI 주제에 대한 질문이라고 해서 자동으로 시각적인 질문은 아닙니다. "어떤 종류의 마법사(wizard)를 원하시나요?"는 개념적이므로 터미널을 사용합니다. "이 마법사 레이아웃 중 어느 쪽이 적절해 보이나요?"는 시각적이므로 브라우저를 사용합니다.

## 작동 방식

서버는 특정 디렉토리에서 HTML 파일을 감시하고 브라우저에 최신 파일을 제공합니다. 당신은 `screen_dir`에 HTML 콘텐츠를 작성하고, 사용자는 브라우저에서 이를 보고 옵션을 클릭하여 선택할 수 있습니다. 선택 사항은 다음 턴에 읽을 수 있도록 `state_dir/events`에 기록됩니다.

**콘텐츠 조각 vs 전체 문서:** HTML 파일이 `<!DOCTYPE` 또는 `<html`로 시작하면 서버는 이를 있는 그대로 제공합니다 (헬퍼 스크립트만 삽입). 그렇지 않으면 서버는 자동으로 콘텐츠를 프레임 템플릿(헤더, CSS 테마, 선택 표시기 및 모든 상호 작용 인프라 포함)으로 감쌉니다. **기본적으로 콘텐츠 조각만 작성하세요.** 페이지를 완전히 제어해야 할 때만 전체 문서를 작성하세요.

## 세션 시작하기

```bash
# 지속성(목업이 프로젝트에 저장됨)이 있는 서버 시작
scripts/start-server.sh --project-dir /path/to/project

# 응답 형식: {"type":"server-started","port":52341,"url":"http://localhost:52341",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#           "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

응답에서 `screen_dir`과 `state_dir`을 저장합니다. 사용자에게 URL을 열도록 안내하세요.

**연결 정보 찾기:** 서버는 시작 JSON을 `$STATE_DIR/server-info`에 기록합니다. 백그라운드에서 서버를 실행하고 표준 출력을 캡처하지 못한 경우 해당 파일을 읽어 URL과 포트를 가져옵니다. `--project-dir`을 사용하는 경우 세션 디렉토리에 대해 `<project>/.superpowers/brainstorm/`을 확인하세요.

**참고:** 프로젝트 루트를 `--project-dir`로 전달하여 목업이 `.superpowers/brainstorm/`에 유지되도록 하세요. 이렇게 하면 서버를 재시작해도 손실되지 않습니다. 그렇지 않으면 파일이 `/tmp`로 이동하여 삭제됩니다. 사용자에게 `.superpowers/`를 `.gitignore`에 추가하도록 상기시키세요.

**다양한 환경에서 서버 실행:**

```bash
# 기본 모드는 환경이 허용하는 경우 서버를 백그라운드에서 실행합니다.
scripts/start-server.sh --project-dir /path/to/project
```

서버는 대화 턴 간에 계속 실행되어야 합니다. OpenCode에서는 `bash` 도구로 스크립트를 실행하고 반환된 `screen_dir`, `state_dir` 및 URL을 보관하세요.

사용 중인 환경에서 분리된 프로세스가 종료되는 경우 `--foreground`를 사용하고 설정에서 제공하는 지속적/백그라운드 실행 메커니즘을 통해 실행하세요. 표준 출력을 캡처하지 않고 실행한 경우 다음 턴에 `$STATE_DIR/server-info`를 읽어 URL과 포트를 가져오세요.

브라우저에서 URL에 접근할 수 없는 경우(원격/컨테이너 환경에서 흔히 발생), 루프백이 아닌 호스트에 바인딩하세요:

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

`--url-host`를 사용하여 반환되는 URL JSON의 호스트 이름을 제어할 수 있습니다.

## 반복 루프 (The Loop)

1. **서버가 실행 중인지 확인**한 다음 `screen_dir`에 새 HTML 파일을 **작성**합니다:
   - 작성하기 전에 `$STATE_DIR/server-info`가 존재하는지 확인하세요. 존재하지 않거나(`$STATE_DIR/server-stopped`가 존재하는 경우) 서버가 종료된 것이므로 계속하기 전에 `start-server.sh`로 다시 시작해야 합니다. 서버는 30분 동안 활동이 없으면 자동으로 종료됩니다.
   - 의미 있는 파일 이름을 사용하세요: `platform.html`, `visual-style.html`, `layout.html`
   - **파일 이름을 재사용하지 마세요.** 각 화면은 새 파일이어야 합니다.
   - `write` 도구를 사용하세요. **cat/heredoc은 터미널에 노이즈를 만드므로 사용하지 마세요.**
   - 서버는 자동으로 가장 최신 파일을 제공합니다.

2. **사용자에게 무엇을 기대할지 알리고 턴을 종료합니다:**
   - 처음뿐만 아니라 매 단계마다 URL을 상기시켜 주세요.
   - 화면의 내용을 텍스트로 간단히 요약합니다 (예: "홈페이지의 3가지 레이아웃 옵션을 보여주고 있습니다.").
   - 터미널에서 응답하도록 요청합니다: "살펴보시고 어떻게 생각하는지 알려주세요. 원하시면 옵션을 클릭하여 선택하실 수 있습니다."

3. **다음 턴에** — 사용자가 터미널에서 응답한 후:
   - `$STATE_DIR/events`가 있다면 읽으세요. 사용자 브라우저 상호작용(클릭, 선택)이 JSON 라인 형식으로 포함되어 있습니다.
   - 사용자의 터미널 메시지와 병합하여 전체적인 상황을 파악하세요.
   - 터미널 메시지가 기본 피드백이며 `state_dir/events`는 구조화된 상호작용 데이터를 제공합니다.

4. **반복 또는 진행** — 피드백에 따라 현재 화면을 변경해야 하면 새 파일(예: `layout-v2.html`)을 작성합니다. 현재 단계가 검증된 경우에만 다음 질문으로 넘어갑니다.

5. **터미널로 복귀 시 언로드** — 다음 단계에 브라우저가 필요하지 않은 경우(예: 명확한 질문, 장단점 논의 등), 대기 화면을 푸시하여 오래된 콘텐츠를 지우세요.

   ```html
   <!-- 파일명: waiting.html (또는 waiting-2.html 등) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">터미널에서 계속 진행 중입니다...</p>
   </div>
   ```

   이렇게 하면 대화가 이동했는데도 사용자가 해결된 선택 항목을 계속 보고 있는 것을 방지할 수 있습니다. 다음 시각적 질문이 나오면 평소처럼 새 콘텐츠 파일을 푸시하세요.

6. 완료될 때까지 반복합니다.

## 콘텐츠 조각 작성하기

페이지 내부로 들어갈 콘텐츠만 작성하세요. 서버가 프레임 템플릿(헤더, 테마 CSS, 선택 표시기 및 상호 작용 인프라 포함)으로 자동으로 감쌉니다.

**최소 예제:**

```html
<h2>어떤 레이아웃이 더 나은가요?</h2>
<p class="subtitle">가독성과 시각적 계층 구조를 고려해 보세요</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>1단 구성 (Single Column)</h3>
      <p>깔끔하고 집중도 높은 읽기 환경</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>2단 구성 (Two Column)</h3>
      <p>메인 콘텐츠와 함께 표시되는 사이드바 네비게이션</p>
    </div>
  </div>
</div>
```

이것으로 충분합니다. `<html>`, CSS, `<script>` 태그는 필요하지 않습니다. 서버가 모두 제공합니다.

## 사용 가능한 CSS 클래스

프레임 템플릿은 다음과 같은 콘텐츠용 CSS 클래스를 제공합니다:

### 옵션 (A/B/C 선택)

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>제목</h3>
      <p>설명</p>
    </div>
  </div>
</div>
```

**다중 선택:** 컨테이너에 `data-multiselect`를 추가하여 사용자가 여러 옵션을 선택할 수 있게 합니다. 클릭할 때마다 각 항목이 토글됩니다. 인디케이터 표시줄에 선택 개수가 표시됩니다.

```html
<div class="options" data-multiselect>
  <!-- 동일한 옵션 마크업 — 여러 개 선택/해제 가능 -->
</div>
```

### 카드 (시각적 디자인)

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- 목업 콘텐츠 --></div>
    <div class="card-body">
      <h3>이름</h3>
      <p>설명</p>
    </div>
  </div>
</div>
```

### 목업 컨테이너

```html
<div class="mockup">
  <div class="mockup-header">미리보기: 대시보드 레이아웃</div>
  <div class="mockup-body"><!-- 목업 HTML --></div>
</div>
```

### 분할 보기 (나란히 배치)

```html
<div class="split">
  <div class="mockup"><!-- 왼쪽 --></div>
  <div class="mockup"><!-- 오른쪽 --></div>
</div>
```

### 장단점 (Pros/Cons)

```html
<div class="pros-cons">
  <div class="pros"><h4>장점</h4><ul><li>장점 내용</li></ul></div>
  <div class="cons"><h4>단점</h4><ul><li>단점 내용</li></ul></div>
</div>
```

### 목업 요소 (와이어프레임 빌딩 블록)

```html
<div class="mock-nav">로고 | 홈 | 정보 | 문의</div>
<div style="display: flex;">
  <div class="mock-sidebar">네비게이션</div>
  <div class="mock-content">기본 콘텐츠 영역</div>
</div>
<button class="mock-button">액션 버튼</button>
<input class="mock-input" placeholder="입력 필드">
<div class="placeholder">플레이스홀더 영역</div>
```

### 타이포그래피 및 섹션

- `h2` — 페이지 제목
- `h3` — 섹션 머리글
- `.subtitle` — 제목 아래 보조 텍스트
- `.section` — 아래쪽 여백이 있는 콘텐츠 블록
- `.label` — 작고 굵은 대문자 레이블 텍스트

## 브라우저 이벤트 형식

사용자가 브라우저에서 옵션을 클릭하면 그 상호작용이 `$STATE_DIR/events`에 JSON 객체 형식(한 줄에 하나씩)으로 기록됩니다. 새 화면을 푸시하면 파일이 자동으로 지워집니다.

```jsonl
{"type":"click","choice":"a","text":"옵션 A - 심플 레이아웃","timestamp":1706000101}
{"type":"click","choice":"c","text":"옵션 C - 복잡한 그리드","timestamp":1706000108}
{"type":"click","choice":"b","text":"옵션 B - 하이브리드","timestamp":1706000115}
```

전체 이벤트 스트림은 사용자의 탐색 경로를 보여줍니다. 사용자가 최종 결정을 내리기 전에 여러 옵션을 클릭할 수 있습니다. 마지막 `choice` 이벤트가 최종 선택인 경우가 많지만, 클릭 패턴을 통해 망설임이나 선호도를 파악하여 질문에 활용할 수 있습니다.

`$STATE_DIR/events`가 없다면 사용자가 브라우저와 상호작용하지 않은 것이므로 터미널 텍스트만 사용하세요.

## 디자인 팁

- **질문에 맞춰 충실도(fidelity) 조절** — 레이아웃은 와이어프레임으로, 폴리싱 질문은 정교하게
- **각 페이지에서 질문 설명** — 단순히 "하나 선택"이 아니라 "어떤 레이아웃이 더 전문적인가요?" 등으로 설명
- **진행 전 단계적 반복** — 피드백에 따라 현재 화면이 바뀌면 새 버전 작성
- **화면당 2~4개 옵션 권장**
- **중요한 경우 실제 콘텐츠 사용** — 사진 포트폴리오의 경우 실제 이미지(Unsplash) 사용. 플레이스홀더 콘텐츠는 디자인 이슈를 가릴 수 있습니다.
- **목업 단순화** — 픽셀 단위의 완벽한 디자인이 아닌 레이아웃과 구조에 집중

## 파일 이름 지정

- 의미 있는 이름을 사용하세요: `platform.html`, `visual-style.html`, `layout.html`
- 파일 이름을 재사용하지 마세요. 각 화면은 반드시 새 파일이어야 합니다.
- 반복 작업 시: `layout-v2.html`, `layout-v3.html`처럼 접미사를 추가하세요.
- 서버는 수정 시간에 따라 가장 최신 파일을 제공합니다.

## 정리하기

```bash
scripts/stop-server.sh $SESSION_DIR
```

세션 시작 시 `--project-dir`을 사용했다면 목업 파일은 프로젝트 폴더의 `.superpowers/brainstorm/`에 유지됩니다. `/tmp` 세션만 서버 종료 시 삭제됩니다.

## 참조 문서

- 프레임 템플릿(CSS 참조): `scripts/frame-template.html`
- 헬퍼 스크립트(클라이언트측): `scripts/helper.js`
