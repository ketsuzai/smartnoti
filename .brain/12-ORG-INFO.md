# 🧠 프로젝트 브레인: 기관정보 관리 기능정의

> **최종 업데이트**: 2026-03-23
> **파일**: `src/pages/oper/operation-org-info.html`
> **연관 브레인**: `08-OPERATION-MANAGEMENT.md`, `07-ROLE-PERMISSION.md`

---

## 1. 개요

기관정보 관리 페이지는 **운영관리 컨텍스트** 안에서 현재 기관의 기본 정보를 조회하고 수정하는 페이지다.

- `super` + `operationContext` (운영관리 모드) → 전체 조회·수정 가능
- `org_admin` → 전체 조회·수정 가능
- `teacher` → 조회 전용 ("👁️ 조회 전용" 뱃지 표시, 편집 버튼 숨김)

```javascript
const canEdit = (userRole === 'super' || userRole === 'org_admin');
```

---

## 2. 섹션 구성

### 2.1 프로필 헤더
- 기관 로고 (이니셜 아바타)
- 기관명 (크게 표시)
- 유형 뱃지 (어린이집 / 유치원)
- 상태 뱃지 (이용중 / 이용종료)
- teacher 역할 시 "👁️ 조회 전용" 뱃지 추가 표시
- canEdit 시 [편집] 버튼 노출

### 2.2 기본 정보 (편집 가능)
| 필드 | 비고 |
|------|------|
| 기관명 | |
| 기관 유형 | 어린이집 / 유치원 |
| 대표 원장 | |
| 연락처 | |
| 이메일 | |
| 주소 | |
| 상세 주소 | |
| 설립일 | |
| 정원 | |

### 2.3 서비스 현황 (KPI 카드)
| 카드 | 내용 |
|------|------|
| 총 반 수 | |
| 총 원아 수 | |
| 총 교직원 수 | |
| 가입 학부모 수 | |

### 2.4 멤버십 정보
- 플랜명 (예: 프리미엄)
- 구독 시작일 / 만료일
- 결제 주기

### 2.5 운영 현황 통계
- 이번 달 알림장 발송 건수
- 이번 달 공지사항 건수
- 전체 상담 이력 건수
- 미처리 투약의뢰서 건수

### 2.6 교직원 목록 (멀티회원)
- 멀티회원(MultiMember) 개념: 한 기관에 소속된 여러 교직원 계정
- 테이블 컬럼: 이름 / 역할 / 담당 반 / 가입일 / 상태
- canEdit 시 행마다 [수정] / [삭제] 버튼 노출

---

## 3. 뷰/편집 모드 전환

```javascript
// 뷰 모드 → 편집 모드
function enterEditMode()   // 입력 필드 활성화, [저장] / [취소] 버튼 노출

// 편집 취소
function cancelEdit()      // 원래 값 복원, 뷰 모드로 복귀

// 저장
function saveEdit()        // mock: ORG_MOCK 갱신 + 뷰 모드 복귀 + 토스트

// 전체 페이지 재렌더링
function renderPage()      // 현재 모드에 따라 뷰/편집 UI 출력
```

---

## 4. 목 데이터 구조

```javascript
const ORG_MOCK = {
  id: 'ORG001',
  name: '해맑은 어린이집',
  type: '어린이집',         // '어린이집' | '유치원'
  director: '김원장',
  phone: '02-1234-5678',
  email: 'haemalk@example.com',
  address: '서울시 강남구 테헤란로 123',
  addressDetail: '2층',
  established: '2015-03-01',
  capacity: 60,
  status: 'active',         // 'active' | 'inactive'
  plan: '프리미엄',
  planStart: '2025-01-01',
  planEnd: '2026-12-31',
  planCycle: '연간',
  stats: {
    classes: 5,
    children: 47,
    teachers: 8,
    parents: 42
  },
  monthly: {
    noticeBoards: 124,
    announcements: 7,
    consultations: 23,
    medicines: 2
  },
  multiMembers: [
    { id: 'T001', name: '이교사', role: '담임교사', class: '햇님반', joinedAt: '2024-03-01', status: 'active' },
    // ...
  ]
};
```

---

## 5. 권한별 UI 차이

| 요소 | super/org_admin | teacher |
|------|:---:|:---:|
| 헤더 [편집] 버튼 | ✅ 표시 | ❌ 숨김 |
| "👁️ 조회 전용" 뱃지 | ❌ | ✅ 표시 |
| 기본 정보 편집 | ✅ | ❌ (텍스트만) |
| 교직원 [수정]/[삭제] | ✅ | ❌ |

---

## 6. 미결 사항

| 항목 | 내용 |
|------|------|
| 기관 로고 이미지 업로드 | 현재 이니셜 아바타만 구현, 실제 이미지 업로드 미구현 |
| 멤버십 플랜 변경 | 플랜 변경 기능 스펙 미정 |
| 교직원 신규 추가 | 목록에 추가 버튼 필요 여부 미정 |
