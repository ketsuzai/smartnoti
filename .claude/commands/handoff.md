Generate a developer handoff document by aggregating all brain files and schema into a single structured spec.

## Steps

1. Read all `.brain/*.md` files
2. Read `.brain/db-schema.md`
3. List all HTML prototype files in the project root
4. Generate `.handoff/spec.md` with the following structure:

---

## Output Structure

### 1. 프로젝트 개요
- 서비스 설명
- 사용자 유형 및 역할
- 기술 스택 권장사항 (프론트엔드, 백엔드, DB, 인프라)

### 2. 페이지 목록
For each brain file:
- 페이지명 및 파일명
- 핵심 기능 요약 (3줄 이내)
- 연결된 프로토타입 HTML 파일

### 3. 기능 상세 스펙
Full content of each brain file, formatted cleanly

### 4. DB 스키마
Full content of db-schema.md

### 5. 미결 사항
Aggregate all "미결 사항" sections from all brain files

### 6. 프로토타입 파일 목록
List of all HTML files with one-line descriptions

---

## Rules

- Write in Korean
- Keep it developer-friendly — precise, no ambiguity
- Flag any brain files that are missing DB schema coverage
- Create `.handoff/` directory if it doesn't exist

## Usage

User says: `/project:handoff`
