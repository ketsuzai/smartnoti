# CLAUDE.md — 키드키즈 스마트알림장 운영관리 시스템

> Claude가 이 프로젝트에서 작업할 때 반드시 읽어야 하는 지침서.
> 상세 기획/스키마는 `.brain/` 폴더를 참조한다.

---

## ⚠️ 협업 규칙 (최우선 기준)

### 수정 금지 파일 (특별 지시 없이 절대 수정 불가)

| 파일명 | 설명 |
|--------|------|
| `src/pages/member-approval.html` | 가입/승인 관리 |
| `src/pages/invitation-admin.html` | 초대장 관리 |
| `src/pages/member-status-admin.html` | 회원 상태 관리 |
| `src/pages/member-list-admin.html` | 통합회원 조회 |
| `src/pages/org-admin.html` | 기관관리 |
| `src/pages/dashboard.html` | 대시보드 |

> 수정이 필요한 상황이면 **작업 전 반드시 사용자에게 확인 요청**.

### 작업 범위
- **운영관리 섹션** 신규 화면 제작이 주 작업 범위.
- 신규 파일은 `src/pages/oper/operation-*.html` 패턴으로 생성.
- 매 작업 완료 후 `.brain/` 해당 파일과 `README.md`를 업데이트한다.

---

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **프로젝트명** | 키드키즈 스마트알림장 운영관리 시스템 |
| **버전** | v2.4 |
| **도메인** | 유아교육기관(어린이집/유치원) 알림장 서비스 운영관리 백오피스 |
| **기술 스택** | Vanilla HTML5 / CSS3 / JS (MPA), Chart.js 4.4.1 |
| **폰트** | Pretendard, JetBrains Mono |
| **데이터** | 프론트엔드 모킹 (프로토타입) |

---

## 관리자 계층 구조

| 역할 코드 | 역할명 | 사이드바 테마 | 테스트 계정 |
|-----------|--------|-------------|-----------|
| `super` | 통합관리자 | 다크 `#1A1D2E` | admin / 1234 |
| `org_admin` | 기관관리자 | 딥네이비 `#0C2D48` | org_admin / 1234 |
| `teacher` | 교사관리자 | 딥네이비 `#0C2D48` | teacher / 1234 |

---

## 현재 파일 목록

### 통합관리자 전용 (`src/pages/`)

| 파일 | 설명 | 상태 |
|------|------|------|
| `index.html` | 로그인 (진입점) | ✅ 완료 |
| `src/pages/dashboard.html` | 대시보드 | ✅ 완료 (수정 금지) |
| `src/pages/member-list-admin.html` | 통합회원 조회 | ✅ 완료 (수정 금지) |
| `src/pages/member-approval.html` | 가입/승인 관리 | ✅ 완료 (수정 금지) |
| `src/pages/invitation-admin.html` | 초대장 관리 | ✅ 완료 (수정 금지) |
| `src/pages/member-status-admin.html` | 회원 상태 관리 | ✅ 완료 (수정 금지) |
| `src/pages/org-admin.html` | 기관관리 | ✅ 완료 (수정 금지) |

### 운영관리 섹션 (`src/pages/oper/`)

| 파일 | 설명 | 상태 |
|------|------|------|
| `src/pages/oper/operation-dashboard.html` | 운영관리 대시보드 (기관·교사 공용) | 🔲 미시작 |
| `src/pages/oper/operation-org-selector.html` | 기관 선택 팝업 | 🔲 미시작 |
| `src/pages/oper/operation-notice-board.html` | 알림장 | ✅ 완료 |
| `src/pages/oper/operation-invitation.html` | 초대 관리 | ✅ 완료 |
| `src/pages/oper/operation-org-info.html` | 기관정보 관리 | ✅ 완료 |
| `src/pages/oper/operation-child.html` | 원아관리 | 🔲 미시작 |
| `src/pages/oper/operation-class.html` | 반 관리 | ✅ 완료 |
| `src/pages/oper/operation-announcement.html` | 공지사항 | 🔲 미시작 |
| `src/pages/oper/operation-album.html` | 앨범 | 🔲 미시작 |
| `src/pages/oper/operation-schedule.html` | 일정 관리 | 🔲 미시작 |
| `src/pages/oper/operation-consulting.html` | 상담 관리 | 🔲 미시작 |
| `src/pages/oper/operation-medicine.html` | 투약의뢰서 관리 | 🔲 미시작 |

---

## 운영관리 로드맵 (Phase 순서)

```
Phase 1. 로그인 개선        — index.html 역할별 분기 (수정 시 확인 필요)
Phase 2. 운영관리 기반      — 기관 선택 팝업 + sessionStorage 컨텍스트
Phase 3. 운영관리 서브메뉴  — operation-*.html 각 페이지
Phase 4. 통계관리           — 기존 dashboard.html 활용
Phase 5. 기관·교사 전용 UI  — 하늘색 테마 분기
```

### Phase 3 서브메뉴 우선순위

| 우선순위 | 파일 | 상태 |
|---------|------|------|
| 🔴 높음 | `operation-org-info.html` | 🔲 미시작 |
| 🔴 높음 | `operation-class.html` | 🔲 미시작 |
| 🔴 높음 | `operation-child.html` | 🔲 작업 중 |
| 🟡 중간 | `operation-notice-board.html` | 🔲 작업 중 |
| 🟡 중간 | `operation-announcement.html` | 🔲 미시작 |
| 🟡 중간 | `operation-album.html` | 🔲 미시작 |
| 🟢 낮음 | `operation-schedule.html` | 🔲 미시작 |
| 🟢 낮음 | `operation-consulting.html` | 🔲 미시작 |
| 🟢 낮음 | `operation-medicine.html` | 🔲 미시작 |

---

## 핵심 설계 원칙

### sessionStorage 구조

```javascript
// 로그인 시 저장
sessionStorage.setItem('userRole', 'super'); // 'super' | 'org_admin' | 'teacher'
sessionStorage.setItem('orgId', 'ORG001');   // 기관관리자·교사관리자 전용

// 통합관리자 운영관리 진입 시 저장
sessionStorage.setItem('operationContext', JSON.stringify({
  orgId: 'ORG001', orgName: '해맑은 어린이집', orgType: '어린이집'
}));

// 교사 담당 반 목록
sessionStorage.setItem('assignedClasses', JSON.stringify([
  { id: 'CLS001', name: '햇님반' }
]));
```

### 로그인 후 리다이렉트

| 역할 | 이동 대상 |
|------|---------|
| 통합관리자 | `src/pages/dashboard.html` |
| 기관관리자 | `src/pages/oper/operation-dashboard.html` |
| 교사관리자 | `src/pages/oper/operation-dashboard.html` |

### 운영관리 컨텍스트 UI
- 상단바: `🏫 OO어린이집 · 운영관리 모드  [기관 변경]  [컨텍스트 종료]`
- 사이드바: 하늘색 테마(`#0C2D48`)로 전환

---

## 디자인 시스템 핵심

### 색상 변수

```css
--bg-base: #F4F5F7;
--bg-surface: #FFFFFF;
--bg-sidebar: #1A1D2E;          /* 통합관리자 */
--bg-sidebar-op: #0C2D48;       /* 운영관리/기관·교사 */
--accent-primary: #4361EE;      /* 통합관리자 액센트 */
--accent-op: #0EA5E9;           /* 운영관리 액센트 */
--text-primary: #1A1D2E;
--text-secondary: #6B7280;
--border-light: #E5E7EB;
```

### 시멘틱 색상 (상태 뱃지용)

| 의미 | 색상 | 배경 | 텍스트 |
|------|------|------|--------|
| 성공/활성/승인 | `#10B981` | `#D1FAE5` | `#065F46` |
| 정보/재원중 | `#3B82F6` | `#DBEAFE` | `#1E40AF` |
| 경고/대기 | `#F59E0B` | `#FEF3C7` | `#92400E` |
| 위험/거절 | `#EF4444` | `#FEE2E2` | `#991B1B` |
| 특수/졸업 | `#8B5CF6` | `#EDE9FE` | `#5B21B6` |

### 공통 레이아웃

```
사이드바: 240px 고정
상단바: 60px 고정
카드: border-radius 12~16px, shadow-md
```

### 공통 컴포넌트 클래스

```html
<!-- 상태 뱃지 -->
<span class="sbadge"><span class="sdot"></span>상태명</span>

<!-- 필터 셀렉트 -->
<div class="fsel-wrap"><select class="fsel">...</select></div>

<!-- 검색 -->
<div class="search-wrap">
  <input class="search-inp"><button class="search-btn">🔍</button>
</div>
```

---

## .brain/ 파일 인덱스

| 파일 | 내용 |
|------|------|
| `00-PROJECT-OVERVIEW.md` | 전체 개요, 수정 금지 목록, 화면 목록, 사이드바 구조 |
| `01-MEMBER-MANAGEMENT.md` | 회원 관리 기능정의 |
| `02-ORG-MANAGEMENT.md` | 기관 관리 기능정의 |
| `03-INVITATION-MANAGEMENT.md` | 초대장 관리 기능정의 |
| `04-DASHBOARD.md` | 대시보드 KPI·차트 |
| `05-DATABASE-SCHEMA.md` | 전체 DB 스키마 |
| `06-DESIGN-SYSTEM.md` | 색상·그림자·라운딩·컴포넌트 패턴 상세 |
| `07-ROLE-PERMISSION.md` | 역할별 권한 매트릭스, 세션 분기 로직 |
| `08-OPERATION-MANAGEMENT.md` | 운영관리 기능정의, 기관 선택 팝업 스펙 |
| `09-ROADMAP.md` | Phase별 작업 로드맵, 미결 사항 |
| `10-CHILD-MANAGEMENT.md` | 원아관리 스키마, 화면 스펙, 모킹 데이터 |
| `11-NOTICE-BOARD.md` | 알림장 스키마, 체크필드, AI 기능, 화면 스펙 |
