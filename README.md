# 🏫 키드키즈 스마트알림장 — 운영관리 시스템

> 유아교육기관(어린이집/유치원) 알림장 서비스의 운영관리 백오피스 프로토타입

---

## 📋 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **프로젝트** | 키드키즈 스마트알림장 운영관리 |
| **버전** | v2.4 |
| **기술 스택** | Vanilla HTML5 + CSS3 + JavaScript (MPA 구조) |
| **외부 라이브러리** | Chart.js 4.4.1, Pretendard Font, JetBrains Mono |
| **데이터** | 프론트엔드 모킹 데이터 (프로토타입) |

---

## 📂 프로젝트 구조

```
📦 프로젝트 루트
├── 📄 index.html                               ← 로그인 페이지 (진입점)
├── 📄 README.md                                ← 이 파일
├── 📄 CLAUDE.md                                ← AI 작업 지침 (Claude용)
├── 📁 src/
│   └── 📁 pages/
│       ├── 📄 dashboard.html                   ← 대시보드 [🔒 수정 금지]
│       ├── 📄 member-list-admin.html           ← 통합회원 조회 [🔒 수정 금지]
│       ├── 📄 member-approval.html             ← 가입/승인 관리 [🔒 수정 금지]
│       ├── 📄 member-status-admin.html         ← 회원 상태 관리 [🔒 수정 금지]
│       ├── 📄 invitation-admin.html            ← 초대장 관리 [🔒 수정 금지]
│       ├── 📄 org-admin.html                   ← 기관관리 [🔒 수정 금지]
│       └── 📁 oper/                            ← 운영관리 섹션 (신규 파일 전용)
│           ├── 📄 operation-dashboard.html     ← 운영관리 대시보드 (기관·교사 공용) [🔲 작업중]
│           ├── 📄 operation-org-selector.html  ← 기관 선택 팝업 [🔲 작업중]
│           ├── 📄 operation-child.html         ← 원아관리 [🔲 작업중]
│           └── 📄 operation-notice-board.html  ← 알림장 [🔲 작업중]
└── 📁 .brain/                                  ← 프로젝트 브레인 (지식베이스)
    ├── 📄 00-PROJECT-OVERVIEW.md               ← 프로젝트 개요
    ├── 📄 01-MEMBER-MANAGEMENT.md              ← 회원 관리 기능정의
    ├── 📄 02-ORG-MANAGEMENT.md                 ← 기관 관리 기능정의
    ├── 📄 03-INVITATION-MANAGEMENT.md          ← 초대장 관리 기능정의
    ├── 📄 04-DASHBOARD.md                      ← 대시보드 기능정의
    ├── 📄 05-DATABASE-SCHEMA.md                ← DB 스키마 정의
    ├── 📄 06-DESIGN-SYSTEM.md                  ← 디자인 시스템 가이드
    ├── 📄 07-ROLE-PERMISSION.md                ← 역할별 권한 매트릭스
    ├── 📄 08-OPERATION-MANAGEMENT.md           ← 운영관리 기능정의
    ├── 📄 09-ROADMAP.md                        ← 작업 로드맵
    ├── 📄 10-CHILD-MANAGEMENT.md               ← 원아관리 기능정의
    └── 📄 11-NOTICE-BOARD.md                   ← 알림장 기능정의
```

---

## 🔗 페이지 URI 맵

### 통합관리자 전용

| 경로 | 설명 | 비고 |
|------|------|------|
| `/index.html` | 관리자 로그인 | 테스트: admin / 1234 |
| `/src/pages/dashboard.html` | 대시보드 | 로그인 후 진입 |
| `/src/pages/member-list-admin.html` | 통합회원 조회 | |
| `/src/pages/member-approval.html` | 가입/승인 관리 | |
| `/src/pages/member-status-admin.html` | 회원 상태 관리 | |
| `/src/pages/invitation-admin.html` | 초대장 관리 | |
| `/src/pages/org-admin.html` | 기관 조회 | |
| `/src/pages/org-admin.html#approval` | 기관 승인 관리 | URL 해시 탭 |
| `/src/pages/org-admin.html#status` | 기관 상태 관리 | URL 해시 탭 |

### 운영관리 (`src/pages/oper/`)

| 경로 | 설명 | 상태 |
|------|------|------|
| `/src/pages/oper/operation-dashboard.html` | 운영관리 대시보드 (기관·교사 공용, 로그인 후 진입) | 🔲 작업중 |
| `/src/pages/oper/operation-org-selector.html` | 기관 선택 팝업 | 🔲 작업중 |
| `/src/pages/oper/operation-child.html` | 원아관리 | 🔲 작업중 |
| `/src/pages/oper/operation-notice-board.html` | 알림장 | 🔲 작업중 |
| `/src/pages/oper/operation-org-info.html` | 기관정보 관리 | 🔲 미시작 |
| `/src/pages/oper/operation-class.html` | 반 관리 | 🔲 미시작 |
| `/src/pages/oper/operation-announcement.html` | 공지사항 | 🔲 미시작 |
| `/src/pages/oper/operation-album.html` | 앨범 | 🔲 미시작 |
| `/src/pages/oper/operation-schedule.html` | 일정 관리 | 🔲 미시작 |
| `/src/pages/oper/operation-consulting.html` | 상담 관리 | 🔲 미시작 |
| `/src/pages/oper/operation-medicine.html` | 투약의뢰서 관리 | 🔲 미시작 |

---

## ✅ 구현 완료 기능

### 1. 로그인 (`index.html`)
- 프리미엄 다크 테마 로그인 UI
- 테스트 계정 인증 (admin/1234, org_admin/1234, teacher/1234)
- 에러 토스트 알림

### 2. 대시보드 (`dashboard.html`)
- 5개 KPI 카드 (활성 기관, 교사, 학부모, 접속자, 게시글)
- DAU/MAU 라인 차트 / 메뉴별 사용량 바 차트
- 운영 To-do 카드, 공지사항, 기관·교사 활동 랭킹
- 디바이스 분포 도넛 차트 / 기간 필터

### 3. 통합회원 조회 (`member-list-admin.html`)
- 전체 회원 테이블 (교사/학부모/기관장)
- 다중 필터 (유형, 상태, 가입경로, 보호자, 검색) + 필터 칩 UI

### 4. 가입/승인 관리 (`member-approval.html`)
- 가입 승인 KPI / 승인·거절·취소 액션
- 일괄 처리 (체크박스) / 상세 패널 (우측 슬라이드)

### 5. 초대장 관리 (`invitation-admin.html`)
- 학부모/교직원 서브탭 / 기관 선택 → 반별 초대장 관리
- 초대 링크 생성·복사·중지·재개·삭제 / KPI 카드

### 6. 회원 상태 관리 (`member-status-admin.html`)
- 상태별 탭 / 상태 변경 모달 / 일괄 상태 변경

### 7. 기관관리 (`org-admin.html`)
- 3탭: 기관 조회 / 기관 승인 / 기관 상태
- 기관 상세 패널 / 멤버십 등급 시스템

---

## 🚧 미구현 / 준비중 기능

| 기능 | 상태 | 설명 |
|------|------|------|
| 운영관리 화면 전체 | 🔲 작업중 | operation-*.html 순차 제작 |
| 기관·교사 대시보드 | 🔲 작업중 | org-dashboard.html |
| 통계관리 | 🔒 준비중 | 사이드바에 "준비중" 표시 |
| 서버 연동 | ❌ 미구현 | 현재 모든 데이터는 프론트엔드 하드코딩 |
| 실제 인증 | ❌ 미구현 | 클라이언트 사이드 테스트 계정만 |
| 반응형 모바일 | ⚠️ 부분 | 데스크탑 우선 설계 |

---

## 👥 관리자 계층 구조

```
통합관리자 (super)          — 다크 사이드바 #1A1D2E, 액센트 #4361EE
기관관리자 (org_admin)      — 딥네이비 사이드바 #0C2D48, 액센트 #0EA5E9
교사관리자 (teacher)        — 딥네이비 사이드바 #0C2D48, 액센트 #0EA5E9
```

---

## 🧠 프로젝트 브레인

`.brain/` 디렉토리에 프로젝트 지식 문서가 관리됩니다:

- **00-PROJECT-OVERVIEW**: 전체 개요, 수정 금지 목록, 사이드바 구조
- **01–04**: 회원/기관/초대/대시보드 기능정의
- **05-DATABASE-SCHEMA**: 전체 DB 테이블 스키마
- **06-DESIGN-SYSTEM**: 색상, 그림자, 라운딩, 컴포넌트 패턴
- **07-ROLE-PERMISSION**: 역할별 권한 매트릭스, 반 필터 정책, 세션 구조
- **08-OPERATION-MANAGEMENT**: 운영관리 기능정의, 기관 선택 팝업 스펙
- **09-ROADMAP**: Phase별 작업 로드맵, 미결 사항
- **10-CHILD-MANAGEMENT**: 원아관리 스키마, 화면 스펙, 모킹 데이터
- **11-NOTICE-BOARD**: 알림장 스키마, 체크필드, AI 기능, 화면 스펙

> AI 협업 시 `CLAUDE.md`를 기준으로 작업하며, `.brain/`에서 상세 기획을 참조합니다.

---

© 2026 키드키즈 알림장 Admin Panel. All Rights Reserved.
