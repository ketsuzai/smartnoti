Read the brain file for the requested page from `.brain/` directory, then build the corresponding HTML file.

## Steps

1. **브레인 체크** : 해당 페이지의 `.brain/*.md` 파일을 읽는다.
   - 컴포넌트 구조, 사용자 흐름, 엣지 케이스, 권한 규칙, 디자인 방향을 모두 파악한다
   - 미결 사항(미결 사항 섹션)이 있으면 빌드 전에 사용자에게 확인한다

2. **스키마 체크** : `.brain/db-schema.md` 를 읽는다.
   - 이 페이지에서 사용하는 테이블과 필드를 파악한다
   - 목업 데이터의 구조가 스키마와 일치하는지 확인한다
   - 브레인과 스키마 간 불일치가 있으면 빌드 전에 사용자에게 보고한다

3. **기존 파일 확인** : HTML 파일이 이미 존재하면 읽고 업데이트한다 (덮어쓰지 않는다)

4. **빌드** : 아래 규칙을 준수한다
   - Vanilla HTML/CSS/JS only (프레임워크 없음)
   - 외부 라이브러리는 CDN만 허용
   - 단독 실행 가능 (다른 파일에 의존하지 않음)
   - 역할별 뷰가 있으면 프로토타입 역할 전환 바 포함
   - 목업 데이터는 db-schema.md 구조를 따른다
   - 디자인은 브레인의 디자인 방향을 따른다
   - `<head>`에 Figma 캡처 스크립트를 항상 포함한다:
     `<script src="https://mcp.figma.com/mcp/html-to-design/capture.js" async></script>`
     (로컬 서버로 열면 Figma 캡처 툴바가 나타나 직접 Figma로 변환 가능)

## Usage

User says: `/project:build [page-name]`

Example: `/project:build dashboard` → reads `.brain/dashboard-main.md` → builds/updates `dashboard.html`

If the page name is ambiguous, list matching brain files and ask which one to use.
