# 🧠 프로젝트 브레인: 운영관리 대시보드

> **최종 업데이트**: 2026-03-31
> **파일**: `src/pages/oper/operation-dashboard.html`
> **연관 브레인**: `08-OPERATION-MANAGEMENT.md`, `11-NOTICE-BOARD.md`, `10-CHILD-MANAGEMENT.md`

---

## ⚡ QUICK REF
> 운영관리 대시보드 핵심 요약

- **파일**: `src/pages/oper/operation-dashboard.html` — 완료
- **역할별 뷰**: 교사(teacher) / 기관관리자(org_admin) / 통합관리자(super) 3가지 분기
- **교사 뷰 핵심**: "오늘의 현황" 미니테이블(5명) + 전체보기 모달, 반 필터 연동 데이터
- **교차 페이지 연동**: sessionStorage pre-fill 키를 통한 알림장/투약의뢰서 페이지 이동
- **미결**: 출석 상태 변경 전용 페이지 없음 (toast만 표시)
- **2026-03-31 개선**: 하단 주간차트 삭제, 최근활동+공지사항 나란히, 더보기 토글, 전 교직원 활동(원장포함), 오늘의 현황 명칭변경, 반필터 원아데이터 연동, 할일 클릭액션, 공지 작성하기 버튼 액션

---

## 1. 개요

운영관리 대시보드는 기관관리자·교사관리자 공용 메인 화면이다.
역할에 따라 표시되는 KPI, 카드, 기능이 달라진다.

---

## 2. 교사 뷰 — 원아 출석현황

### 2.1 설계 결정 (2026-03-24)

| 검토 옵션 | 결정 |
|----------|------|
| 알림장 하위 서브메뉴로 출석표 추가 | ❌ 기각 — 투약정보도 포함되어 알림장 하위로 분류 부적절 |
| 별도 사이드바 메뉴 항목 추가 | ❌ 기각 |
| 대시보드 내 전체보기 모달로 구현 | ✅ 채택 |

> **기각 사유**: "알림장 하위로 가기에는 투약정보도 있어서 좀 그렇다" — 사용자 피드백

### 2.2 미니테이블 (메인 카드 내)

- 대시보드 메인 카드(`mainCard`)에 렌더
- **상단**: 출석/결석/조퇴/외출 4개 요약 카드 (색상 코드)
- **하단**: `CHILDREN_CLS001.slice(0, 5)` 으로 5명 프리뷰 테이블
- 컬럼: 원아명 / 출석 상태 / 알림장 (작성완료/미작성) / 투약 (있음/없음)
- "+N명 더 보기 →" 클릭 시 전체보기 모달 열기

### 2.3 전체보기 모달 (`attOverlay`)

- **트리거**: "전체 보기" 버튼 또는 "+N명 더 보기" 클릭
- **UI 구조**:
  ```
  ┌─────────────────────────────────────────┐
  │ 🧒 원아 출석 현황  [햇님반 · 오늘]  [✕] │
  ├─────────────────────────────────────────┤
  │ 요약 카드 6개 (출석/결석/조퇴/외출/     │
  │              투약예정/알림장미작성)       │
  ├─────────────────────────────────────────┤
  │ 필터: [전체] [알림장 미작성] [💊투약있음] │
  │       [결석·조퇴·외출]                   │
  ├─────────────────────────────────────────┤
  │ 스크롤 테이블                            │
  │  # | 원아명 | 출석 상태 | 알림장 | 투약  │
  │  1   김지우   출석       작성완료  —      │
  │  2   이서준   출석       미작성    💊3회  │
  │  ...                                     │
  ├─────────────────────────────────────────┤
  │ 항목을 클릭하면 해당 원아의 기능으로      │
  │ 바로 이동합니다                          │
  └─────────────────────────────────────────┘
  ```
- **닫기**: 오버레이 배경 클릭, ✕ 버튼, ESC 키
- **애니메이션**: opacity 0→1 + translateY(12px→0) 슬라이드업
- **크기**: `min(900px, 90vw)` × `max-height: 88vh`

### 2.4 셀 클릭 동작 (교차 페이지 연동)

| 셀 | 클릭 동작 | sessionStorage 키 | 이동 대상 |
|----|----------|-------------------|----------|
| 알림장 (미작성) | 작성 화면으로 이동 | `noticePrefill` | `operation-notice-board.html` |
| 알림장 (작성완료) | 해당 알림장 보기 | `noticePrefill` | `operation-notice-board.html` |
| 투약 (있음) | 투약의뢰서 조회 | `medicinePrefill` | `operation-medicine.html` |
| 출석 상태 | toast 표시 (연동 예정) | — | — (전용 페이지 없음) |

### 2.5 sessionStorage pre-fill 구조

```javascript
// 알림장 이동 시
sessionStorage.setItem('noticePrefill', JSON.stringify({
  childId: 'CHD001',
  childName: '김지우',
  mode: 'view' | 'write'  // 작성완료 → view, 미작성 → write
}));

// 투약의뢰서 이동 시
sessionStorage.setItem('medicinePrefill', JSON.stringify({
  childId: 'CHD002',
  childName: '이서준'
}));
```

> ⚠️ **주의**: `operation-notice-board.html`과 `operation-medicine.html`에서 이 pre-fill 키를 아직 수신 처리하지 않음. 추후 연동 필요.

---

## 3. 목업 데이터

### 3.1 CHILDREN_CLS001 (18명)

```javascript
const CHILDREN_CLS001 = [
  { id:'CHD001', name:'김지우',  attend:'출석', noticeWritten:true,  medicine:null },
  { id:'CHD002', name:'이서준',  attend:'출석', noticeWritten:false, medicine:{count:3, time:'오전 10시'} },
  { id:'CHD003', name:'박하은',  attend:'결석', noticeWritten:false, medicine:null },
  // ... (총 18명)
];
```

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | string | 원아 ID (CHD001~CHD018) |
| `name` | string | 원아 이름 |
| `attend` | string | 출석/결석/조퇴/외출 |
| `noticeWritten` | boolean | 당일 알림장 작성 여부 |
| `medicine` | object\|null | 투약의뢰 정보 (`{count, time}`) |

---

## 4. 미결 사항

| # | 항목 | 내용 |
|---|------|------|
| 1 | 출석 상태 변경 | 전용 출석 관리 페이지 없음 — 현재 toast만 표시 |
| 2 | pre-fill 수신 처리 | 알림장·투약의뢰서 페이지에서 sessionStorage 키 읽어서 자동 이동 미구현 |
| 3 | 기관관리자 뷰 | 대시보드 기관관리자 전용 KPI·차트 상세 미정의 |
| 4 | 통합관리자 뷰 | 운영관리 모드에서의 대시보드 표시 내용 미정의 |
