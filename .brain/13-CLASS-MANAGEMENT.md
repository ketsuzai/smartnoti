# 🧠 프로젝트 브레인: 반 관리 기능정의

> **최종 업데이트**: 2026-03-23
> **파일**: `src/pages/oper/operation-class.html`
> **연관 브레인**: `08-OPERATION-MANAGEMENT.md`, `07-ROLE-PERMISSION.md`, `05-DATABASE-SCHEMA.md`, `10-CHILD-MANAGEMENT.md`

---

## 1. 개요

운영관리 내 반(Class) 관리 페이지. 기관에 속한 반 목록을 조회·생성·수정·삭제하며, 담당 교사 배정과 정원 현황을 관리한다.

---

## 2. 권한 매트릭스

| 역할 | 반 생성 | 반 수정 | 반 삭제 | 조회 |
|------|:---:|:---:|:---:|:---:|
| super (운영관리 컨텍스트) | ✅ | ✅ 전체 반 | ✅ | ✅ |
| org_admin | ✅ | ✅ 전체 반 | ✅ | ✅ |
| teacher | ❌ | ✅ **담당 반만** | ❌ | ✅ |

```javascript
const canCreate  = userRole !== 'teacher';
const canDelete  = userRole !== 'teacher';
const canEdit    = (clsId) => userRole !== 'teacher' || assignedClasses.some(c => c.id === clsId);
```

- teacher: [+ 반 추가] 버튼 미표시, 삭제 버튼 미표시
- teacher: 담당 반 카드에만 [수정] 버튼 표시, 담당 반에 "✓ 담당 반" 배지 + 좌측 강조 테두리

---

## 3. 화면 구성

### 3.1 KPI 카드 (4개)
| 카드 | 색상 | 내용 |
|------|------|------|
| 전체 반 | 액센트 | CLASSES 전체 수, 비운영 포함 안내 |
| 운영중 반 | 초록 | status === 'active' 수 |
| 총 원아 수 | 보라 | childCount 합계 |
| 평균 충원율 | 앰버 | (총 원아 / 총 정원) × 100% |

### 3.2 필터 / 액션바
- 상태 필터 (전체 / 운영중 / 비운영)
- 학년도 필터 (전체 / 2026년도 / 2025년도)
- 텍스트 검색 (반 이름, 교사명)
- 검색 결과 건수 표시
- [+ 반 추가] 버튼 (super/org_admin만)

### 3.3 반 카드 그리드 (3열)
각 카드 표시 항목:
- 상단 컬러바 (반마다 고유 색상)
- 반 이름 + 운영 상태 뱃지
- 연령 그룹 + 학년도
- 담임교사 + 보조교사 (있을 시)
- 원아 수 / 정원 진행바 (충원율에 따라 색상 분기: 90%↑ 빨강 / 70%↑ 앰버 / 나머지 초록)
- 생성일
- [수정] / [삭제] 버튼 (권한 분기)

### 3.4 우측 드로어 (클릭 시)
- 반 기본 정보 (이름 / 연령 / 학년도 / 생성일)
- 정원 현황 카드 (진행바 + 잔여 정원)
- 담당 교사 섹션 (담임 + 보조 카드)
- 소속 원아 목록 (5명 미리보기 + "원아관리로 이동" 링크)
- 하단 [수정] / [삭제] 버튼 (권한 분기)
- 권한 없을 시: "조회 전용 — 수정 권한 없음" 안내

---

## 4. 반 추가/수정 모달

| 필드 | 타입 | 필수 |
|------|------|:---:|
| 반 이름 | text input | ✅ |
| 연령 그룹 | select (만 0세~7세) | ✅ |
| 학년도 | select (2024~2026) | ✅ |
| 정원 | number input (1~50) | ✅ |
| 담임 교사 | select | — |
| 보조 교사 | select | — |
| 운영 상태 | toggle (운영중/비운영) | — |

---

## 5. 삭제 모달

- 삭제 대상 반 정보 요약 표시 (이름 / 연령 / 담임 / 원아 수)
- 소속 원아 있을 시 경고: "삭제 시 원아의 반 배정이 해제됩니다"

---

## 6. 데이터 모델

### classes 테이블 (DB 스키마 §5 + 추가 필드)

| 필드 | 타입 | 설명 |
|------|------|------|
| id | text (PK) | 반 고유 ID |
| org_id | text (FK) | 소속 기관 ID |
| name | text | 반 이름 |
| age_group | text | 만 0세 ~ 만 7세 |
| year | number | 학년도 (2026) |
| teacher | text | 담임 교사명 |
| co_teacher | text | 보조 교사명 (선택) |
| child_count | number | 현재 원아 수 |
| capacity | number | 정원 |
| status | text | 'active' \| 'inactive' |
| color | text | 카드 색상 (HEX) |
| created_at | text | 생성일 |

> 기존 DB 스키마의 `status` 값 `'none'` / `'stopped'`는 초대장 전용. 반 관리에서는 `'active'` / `'inactive'`만 사용.

---

## 7. 상태 간 이동

```
미운영(inactive) ← 토글 OFF → 운영중(active)
                → 삭제 → (삭제됨)
```

---

## 8. 드로어 → 원아관리 연동

드로어 하단 "원아관리로 이동" 클릭 시:
```javascript
sessionStorage.setItem('currentClass', clsId);
window.location.href = 'operation-child.html';
```
→ operation-child.html에서 `currentClass` 세션 값 읽어 해당 반 자동 필터 적용 예정

---

## 9. 미결 사항

| 항목 | 내용 |
|------|------|
| 반 색상 자동 배정 | 신규 반 추가 시 고유 색상 풀에서 순환 배정 (현재 8색 풀) |
| 반 복사 기능 | 이전 학년도 반 구조 복사 기능 필요 여부 미결 |
| 교사 목록 연동 | 담임/보조 셀렉트박스가 현재 하드코딩 → org_members에서 동적 로드 필요 |
