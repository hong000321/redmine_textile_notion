# Redmine 위키 에디터 프로젝트 요약

## 목표
Notion처럼 마크다운 즉시 렌더링되는 에디터로, 내용을 **Redmine Textile** 형식으로 변환/복사할 수 있는 단독 HTML 파일 제작

---

## 최종 결과물
`redmine_editor.html` — 단일 HTML 파일, 외부 의존성 없음

---

## 구현된 기능 전체 목록

### 에디터 입력 방식
- 마크다운 입력 → 즉시 렌더링 (단일 뷰, Notion 방식)
- `# ` `## ` `### ` → H1/H2/H3 (Space 키 트리거)
- `- ` `* ` → Bullet list (Space 키 트리거)
- `1. ` `2. ` ... → Numbered list (Space 키 트리거)
- `> ` → **Toggle 블록** (▶ 클릭으로 접기/펼치기, Space 키 트리거)
- ` ``` ` → Code block (Space 또는 Enter 트리거)
- `---` → 수평선 (Enter 트리거)
- `**text**` → Bold, `*text*` → Italic (input 이벤트)

### 단축키
- `Ctrl+B` / `⌘B` → Bold 토글
- `Ctrl+I` / `⌘I` → Italic
- `Ctrl+K` / `⌘K` → 링크 삽입 모달
- `Ctrl+Z` / `Ctrl+Y` → 실행취소 / 다시실행
- `Shift+Enter` → Code block / Toggle block / Toggle body에서 탈출 (다음 `<p>`로 이동)

### 슬래시(`/`) 커맨드
- `/` 입력 시 불투명 메뉴 팝업
- 입력 후 한글/영문 실시간 검색 (`/표` = `/table`, `/토글` = `/toggle` 등)
- ↑↓ 키 탐색, Enter 선택, Esc 닫기, 클릭으로도 선택 가능
- 항목: H1/H2/H3, Bullet, Numbered, Quote(인용), Toggle, Code block, Table, Divider, Attachment, Image

### 테이블
- 셀 클릭 시 상단에 컨텍스트 바 표시 (행 추가/삭제, 열 추가/삭제, 테이블 삭제)
- `Tab` / `Shift+Tab` 으로 셀 간 이동
- 헤더 보라색 배경, 짝수 행 연보라, 호버 강조

### 첨부 / 이미지
- `attachment:"파일명"` → 보라색 칩으로 렌더링
- `!파일명.png!` → 이미지 칩으로 렌더링
- `!https://...!` → `<img>` 인라인 렌더링
- 이미지 삽입 모달: URL 입력 시 실시간 미리보기

### Toggle 블록
- `> ` + Space 또는 슬래시 커맨드(`/toggle`)로 생성
- ▶ 아이콘 + 편집 가능한 title + 접히는 본문 영역
- title에서 Enter → body로 이동, `Shift+Enter` → 토글 블록 밖으로 탈출
- body에서 `Shift+Enter` → 토글 블록 밖으로 탈출
- Textile export 시 `> 제목\n내용` 형태로 변환

### Textile 가져오기 (Import)
- 모달에 Redmine Textile 원문 붙여넣기 → 실시간 미리보기 → 에디터 적용
- 지원 문법
  - `h1.`~`h6.` 헤딩
  - `* ** ***` / `# ## ###` 중첩 목록
  - `(/)` 체크마크 매크로 → 초록 ✓ 뱃지
  - `bq.` 인용
  - `<pre><code class="...">` — class 속성 포함 코드블록 정상 파싱
  - URL 인코딩된 이미지 경로 자동 디코딩 (`%EC%84%A4%EC%A0%95...` → 파일명 칩)
  - `attachment:"파일명"` 첨부 칩
  - `|_. |` 테이블

### Textile 내보내기 (Export)
- "Copy as Textile" 버튼 → 변환 결과 모달 + 클립보드 복사
- 중첩 목록 `## ###` / `** ***` 형태로 역변환
- 토글 블록 → `> 제목\n내용` 형태로 변환

### Backspace 탈출
- 빈 `li` (단일 항목) → 리스트 전체를 `<p>`로 교체
- 빈 `li` (다중 항목) → 해당 항목 제거 후 목록 뒤에 `<p>` 생성
- 빈 toggle-title → 토글 블록 전체를 `<p>`로 교체
- 빈 blockquote → `<p>`로 교체
- Code block 안 커서 맨 앞(offset 0) → 브라우저 블록 병합 차단

### Shift+Enter 탈출 힌트
- Code block, Toggle body에 포커스 시 우하단에 "Shift + Enter로 탈출" 텍스트를 흐릿하게 표시
- `escape-hint-wrapper` CSS 클래스로 구현, 포커스 시 자동 표시

---

## 버그 수정 이력

| 항목 | 원인 | 수정 방법 |
|---|---|---|
| 한글 입력 후 글자 사라짐 | `compositionend` 직후 Space keydown이 마크다운 변환을 트리거 | `justFinishedComposition` 플래그 + `setTimeout(0)` 으로 차단 |
| Code block 안 Backspace 시 블록 위로 이동 | 브라우저가 커서 맨 앞 Backspace를 블록 병합으로 처리 | offset 0일 때 `preventDefault`, 빈 블록은 `<p>` 교체 |
| Toggle block 탈출 불가 | toggle-title, toggle-body에 탈출 경로 없음 | `Shift+Enter`로 토글 블록 밖 다음 `<p>`로 이동 |
| `>` + Space가 blockquote로 변환 | 기존 로직이 blockquote 생성 | toggle-block(`div.toggle-block`) 생성으로 교체 |

---

## 실제 Textile 파일(`textile.txt`) 파싱 이슈 처리 내역

| 문법 | 처리 내용 |
|---|---|
| `## / ###` 중첩 번호목록 | 중첩 `<ol>` 렌더링 |
| `** / ***` 중첩 불릿 | 중첩 `<ul>` 렌더링 |
| `(/)` 체크마크 | 초록 ✓ 뱃지로 표시 |
| `<pre><code class="diff">` | class 속성 무시 후 내용 정상 추출 |
| `!/attachments/download/553400/%EC%84%A4...png!` | `decodeURIComponent`로 파일명 디코딩 후 이미지 칩 표시 |

---

## 다음 세션 인계 사항
- 결과물 파일: `redmine_editor.html` (단독 실행 가능)
- 모든 수정사항 반영 완료, 추가 요청사항 없이 마무리된 상태
- 이어서 작업 시 이 파일을 기준으로 수정 진행
