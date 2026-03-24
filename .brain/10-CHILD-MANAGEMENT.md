# 🧠 프로젝트 브레인: 원아관리 기능정의

> **최종 업데이트**: 2026-03-24 (operation-child.html 완료)
> **작업 범위**: 운영관리 내 원아관리 (`operation-child.html`)
> **연관 브레인**: `07-ROLE-PERMISSION.md`, `08-OPERATION-MANAGEMENT.md`, `05-DATABASE-SCHEMA.md`

---

## ⚡ QUICK REF
> 빌드 시 핵심 요약. 상세 스펙은 아래 섹션 참조.

- **파일**: `src/pages/oper/operation-child.html` ✅ 완료
- **역할**: 기관 내 원아 목록 조회·등록·수정·상태관리 + 보호자 연결 관리
- **권한**: super/org_admin → CRUD 전체; teacher → 담당 반 원아 조회·수정만
- **상태값**: `enrolled`재원중 / `graduated`졸업 / `withdrawn`퇴소 / `on_leave`휴원중 / `pending`등록대기
- **핵심 필드**: id, name, birth, gender, class_id, status, primary_parent_id, parents[]
- **UI 구조**: 반 탭 필터 + 검색/상태 필터바 → 테이블 목록 → 우측 드로어(상세/보호자/수정)
- **반 연동**: `sessionStorage.getItem('currentClass')` 로 반 관리에서 진입 가능
- **보호자**: 주계정 1명 + 가족계정 N명, parents[] 배열로 관리

---

## 1. 원아관리 개요

### 1.1 개념 정의

- **원아(Children)**는 어린이집/유치원에 등록된 아동 개체다.
- 원아는 **학부모(parent) 계정에 종속**되는 개념이다.
- 학부모는 가입 시 **원하는 만큼 자녀를 생성**할 수 있다 (1:N 관계).
- 하나의 원아 레코드에는 **주계정 학부모** 1명과 **가족계정 학부모** N명이 연결될 수 있다.
- 원아는 특정 **반(class)**에 배정되며, 반 배정은 교사·기관관리자가 수행한다.

### 1.2 비즈니스 규칙

```
학부모 계정 1명 ──has many──→ 자녀(children) N명
자녀 1명 ──belongs to──→ 반(class) 1개
자녀 1명 ──has many──→ 보호자(parents) N명 (주계정 + 가족계정)
반 1개 ──belongs to──→ 기관(organization) 1개
```

| 규칙 | 내용 |
|------|------|
| 원아 생성 | 학부모 가입 시 자녀 생성 (또는 교사/기관관리자가 직접 등록) |
| 반 배정 | 교사 또는 기관관리자가 원아를 특정 반에 배정 |
| 원아 수정 | 교사관리자, 기관관리자, 통합관리자(운영관리 컨텍스트) 수정 가능 |
| 보호자 필터링 | 원아 목록에서 학부모 계정 기준으로 필터링 가능 |
| 복수 보호자 | 주계정(엄마/아빠 등) + 가족계정(조부모, 다른 부모 등) |
| 상태 관리 | 재원중, 졸업, 퇴소 등 상태 변경 가능 |

---

## 2. 원아(children) 테이블 스키마

### 2.1 기본 정보

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `id` | text (PK) | ✅ | 원아 고유 ID (예: `CHD001`) |
| `org_id` | text (FK) | ✅ | 소속 기관 ID |
| `class_id` | text (FK) | | 배정된 반 ID (미배정 시 null) |
| `class_name` | text | | 배정된 반명 (표시용) |

### 2.2 아동 정보

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `name` | text | ✅ | 아동 이름 |
| `birth` | text | ✅ | 생년월일 (YYYY-MM-DD) |
| `age` | text | | 만 나이 (표시용, 계산 생성) |
| `gender` | text | ✅ | '남' \| '여' |
| `profile_color` | text | | 아바타 배경색 (HEX, 랜덤 지정) |
| `photo_url` | text | | 아동 사진 URL (선택) |

### 2.3 상태 정보

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `status` | text | ✅ | 'enrolled' \| 'graduated' \| 'withdrawn' \| 'on_leave' \| 'pending' |
| `status_label` | text | ✅ | 재원중, 졸업, 퇴소, 휴원중, 등록대기 |
| `enroll_date` | text | ✅ | 입원일 (YYYY-MM-DD) |
| `graduate_date` | text | | 졸업/퇴소일 |
| `expected_graduate` | text | | 예상 졸업년도 (YYYY) |

### 2.4 보호자 연결 정보

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `primary_parent_id` | text (FK) | ✅ | 주계정 학부모 ID (members 테이블) |
| `primary_parent_name` | text | ✅ | 주계정 학부모 이름 |
| `primary_parent_phone` | text | | 주계정 학부모 연락처 |
| `primary_parent_relation` | text | ✅ | 주계정 관계: '엄마' \| '아빠' \| '기타' |
| `parents` | array | | 연결된 전체 보호자 목록 (하단 상세) |
| `parent_count` | number | | 연결된 보호자 수 |

### 2.5 parents 배열 내부 구조

```javascript
parents: [
  {
    member_id: 'MBR005',     // 학부모 회원 ID
    name: '김민지',           // 이름
    phone: '010-1234-5678',  // 연락처
    relation: '엄마',         // 관계: '엄마' | '아빠' | '할머니' | '할아버지' | '기타'
    account_type: 'primary', // 'primary' | 'family'
    app_connected: true,     // 앱 연결(가입) 여부
    last_access: '2026-03-17' // 최근 접속일
  },
  {
    member_id: 'MBR006',
    name: '이준혁',
    phone: '010-9876-5432',
    relation: '아빠',
    account_type: 'family',
    app_connected: false,
    last_access: null
  }
]
```

### 2.6 등록 경로 및 기타

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `reg_route` | text | ✅ | 등록 경로: '학부모직접' \| '교사등록' \| '기관등록' |
| `memo` | text | | 특이사항 메모 (교사용) |
| `allergy` | array | | 알레르기 정보 (문자열 배열) |
| `created_at` | text | ✅ | 레코드 생성일시 |
| `updated_at` | text | | 최종 수정일시 |
| `updated_by` | text | | 최종 수정자 (회원 ID) |

---

## 3. 원아관리 화면 구성

### 3.1 화면 레이아웃

```
┌─────────────────────────────────────────────────────────┐
│ 상단바 (기관명, 반 필터, 브레드크럼)                        │
├──────────────┬──────────────────────────────────────────┤
│              │ [원아관리] 상단                             │
│  사이드바     │  ┌── KPI 카드 ──────────────────────────┐ │
│              │  │ 전체원아 | 재원중 | 졸업/퇴소 | 미연결  │ │
│              │  └──────────────────────────────────────┘ │
│              │  ┌── 필터/검색 바 ─────────────────────┐  │
│              │  │ 반 | 상태 | 성별 | 연령 | 보호자검색  │  │
│              │  └────────────────────────────────────┘  │
│              │  ┌── 액션 바 ──────────────────────────┐  │
│              │  │ [원아 등록] [엑셀내보내기] 체크된N명▼  │  │
│              │  └────────────────────────────────────┘  │
│              │  ┌── 원아 목록 테이블 ─────────────────┐  │
│              │  │ 체크│상태│사진│이름│성별│나이│반│       │  │
│              │  │     보호자│앱연결│입원일│비고         │  │
│              │  └────────────────────────────────────┘  │
└──────────────┴──────────────────────────────────────────┘
```

### 3.2 KPI 카드 (4개)

| 카드 | 필드 | 색상 |
|------|------|------|
| 전체 원아 | 전체 children 수 | 블루 |
| 재원중 | status='enrolled' | 그린 |
| 졸업/퇴소 | status='graduated' or 'withdrawn' | 그레이 |
| 앱 미연결 | parents 중 app_connected=false 인 원아 수 | 오렌지 |

---

## 4. 필터 시스템

### 4.1 필터 항목

| 필터명 | 타입 | 옵션 | 기본값 |
|--------|------|------|--------|
| **반 필터** | select | 전체 + 반 목록 | 전체 (기관관리자) / 담당반 (교사) |
| **재원 상태** | select | 전체, 재원중, 졸업, 퇴소, 휴원중, 등록대기 | 전체 |
| **성별** | select | 전체, 남, 여 | 전체 |
| **연령(만)** | select | 전체, 만0세~만7세 | 전체 |
| **보호자 검색** | text | 보호자 이름 또는 전화번호 입력 | 빈값 |
| **원아 검색** | text | 원아 이름 입력 | 빈값 |

### 4.2 역할별 반 필터 정책

| 역할 | 반 필터 표시 | 선택 가능 범위 |
|------|------------|--------------|
| 통합관리자 (운영관리 컨텍스트) | 기관 전체 반 | 모든 반 선택 가능 |
| 기관관리자 | 기관 전체 반 | 모든 반 선택 가능 |
| 교사관리자 | 기관 전체 반 (비담당 반 disabled/회색) | 담당 반만 선택 가능 |

> 📌 교사관리자는 비담당 반에 속한 원아를 볼 수 없음 (데이터 자체 필터링)

---

## 5. 원아 목록 테이블

### 5.1 테이블 컬럼 정의

| # | 컬럼명 | 필드 | 너비 | 비고 |
|---|--------|------|------|------|
| 1 | ☑ | checkbox | 40px | 일괄 선택 |
| 2 | 상태 | `status` | 80px | 색상 뱃지 |
| 3 | 아동명 | `name` + `profile_color` | 140px | 아바타 + 이름 |
| 4 | 성별 | `gender` | 50px | 남/여 아이콘 |
| 5 | 만 나이 | `age` | 60px | 계산 표시 |
| 6 | 생년월일 | `birth` | 100px | |
| 7 | 반 | `class_name` | 100px | 뱃지 스타일 |
| 8 | 주보호자 | `primary_parent_name` | 100px | |
| 9 | 보호자 수 | `parent_count` | 70px | N명 링크 |
| 10 | 앱 연결 | app_connected 여부 | 80px | ✅/❌ |
| 11 | 입원일 | `enroll_date` | 100px | |
| 12 | 액션 | - | 80px | 상세/수정 버튼 |

### 5.2 상태 뱃지 색상

| 상태 | label | 색상 | 배경 |
|------|-------|------|------|
| `enrolled` | 재원중 | `#059669` | `#D1FAE5` |
| `graduated` | 졸업 | `#6B7280` | `#F3F4F6` |
| `withdrawn` | 퇴소 | `#DC2626` | `#FEE2E2` |
| `on_leave` | 휴원중 | `#D97706` | `#FEF3C7` |
| `pending` | 등록대기 | `#7C3AED` | `#EDE9FE` |

---

## 6. 원아 상세/수정 패널 (슬라이드 패널)

### 6.1 패널 구성 탭

| 탭 | 내용 |
|----|------|
| 기본정보 | 아동 이름, 생년월일, 성별, 반, 입원일, 예상졸업년도 |
| 보호자 정보 | 연결된 보호자 목록 (주계정/가족계정), 앱 연결 여부, 연락처 |
| 건강/특이사항 | 알레르기, 특이사항 메모 |
| 활동 이력 | 입퇴원 이력, 반 이동 이력, 수정 이력 |

### 6.2 수정 가능 필드 (역할별)

| 필드 | 통합관리자 | 기관관리자 | 교사관리자 |
|------|:---------:|:---------:|:---------:|
| 이름 | ✅ | ✅ | ✅ |
| 생년월일 | ✅ | ✅ | ✅ |
| 성별 | ✅ | ✅ | ✅ |
| 반 배정 | ✅ | ✅ | ✅ |
| 입원일/졸업일 | ✅ | ✅ | ✅ |
| 상태 변경 | ✅ | ✅ | ✅ |
| 알레르기/메모 | ✅ | ✅ | ✅ |
| 보호자 연결/수정 | ✅ | ✅ | ❌ |
| 원아 삭제 | ✅ | ✅ | ❌ |

---

## 7. 원아 등록 모달

### 7.1 등록 필드

```
필수 항목 (*)
├── * 아동 이름
├── * 생년월일 (date picker)
├── * 성별 (남/여 라디오)
├── * 반 선택 (select)
├── * 입원일 (date picker)
└── * 주보호자 연결
    └── 학부모 검색 (이름/전화번호)
    └── 검색 결과에서 선택 → 연결

선택 항목
├── 예상 졸업년도
├── 알레르기 입력 (태그 입력)
└── 특이사항 메모
```

### 7.2 보호자 연결 방식

1. **기존 회원 검색**: 이름 또는 전화번호로 members 테이블에서 학부모(type='parent') 검색
2. **신규 등록 후 연결**: 보호자 정보 입력 → 임시 계정 생성 → 나중에 학부모 앱에서 연동
3. **미연결 등록**: 보호자 연결 없이 원아만 먼저 등록 가능 (status: pending)

---

## 8. 보호자 필터링 상세

### 8.1 보호자 기준 필터링

원아 목록에서 **특정 보호자의 자녀 전체**를 조회 가능:

```
보호자 검색 입력 → "김민지 (010-1234-5678)" 선택
→ 해당 학부모와 연결된 모든 자녀 필터링
```

- 검색 입력: 보호자 이름 또는 전화번호
- 자동완성: 입력 시 학부모 목록 드롭다운 표시
- 선택 시 해당 보호자의 자녀만 목록에 표시

### 8.2 활용 시나리오

| 시나리오 | 방법 |
|----------|------|
| 형제자매 한번에 확인 | 부모 계정으로 검색 → 연결된 자녀 모두 표시 |
| 특정 학부모 연락처 확인 | 아동명 검색 → 보호자 정보 탭에서 확인 |
| 앱 미연결 보호자 파악 | '앱 미연결' 상태 필터 → 연락 필요 목록 추출 |

---

## 9. 모킹 데이터 예시

```javascript
const MOCK_CHILDREN = [
  {
    id: 'CHD001',
    org_id: 'ORG001',
    class_id: 'CLS001',
    class_name: '햇님반',
    name: '김도현',
    birth: '2021-03-15',
    age: '만 5세',
    gender: '남',
    profile_color: '#4361EE',
    status: 'enrolled',
    status_label: '재원중',
    enroll_date: '2024-03-02',
    expected_graduate: '2027',
    primary_parent_id: 'MBR010',
    primary_parent_name: '김민지',
    primary_parent_phone: '010-1234-5678',
    primary_parent_relation: '엄마',
    parent_count: 2,
    parents: [
      { member_id: 'MBR010', name: '김민지', phone: '010-1234-5678', relation: '엄마', account_type: 'primary', app_connected: true, last_access: '2026-03-17' },
      { member_id: 'MBR011', name: '이준혁', phone: '010-9876-5432', relation: '아빠', account_type: 'family', app_connected: true, last_access: '2026-03-10' }
    ],
    reg_route: '학부모직접',
    allergy: ['땅콩', '갑각류'],
    memo: '활발하고 에너지 넘침. 낮잠 30분 필수.',
    created_at: '2024-03-02T09:00:00',
    updated_at: '2026-03-15T14:30:00',
    updated_by: 'teacher_001'
  },
  {
    id: 'CHD002',
    org_id: 'ORG001',
    class_id: 'CLS001',
    class_name: '햇님반',
    name: '박서연',
    birth: '2021-07-20',
    age: '만 4세',
    gender: '여',
    profile_color: '#EC4899',
    status: 'enrolled',
    status_label: '재원중',
    enroll_date: '2024-03-02',
    expected_graduate: '2027',
    primary_parent_id: 'MBR012',
    primary_parent_name: '박지영',
    primary_parent_phone: '010-2222-3333',
    primary_parent_relation: '엄마',
    parent_count: 1,
    parents: [
      { member_id: 'MBR012', name: '박지영', phone: '010-2222-3333', relation: '엄마', account_type: 'primary', app_connected: false, last_access: null }
    ],
    reg_route: '교사등록',
    allergy: [],
    memo: '',
    created_at: '2024-03-02T09:10:00',
    updated_at: '2024-03-02T09:10:00',
    updated_by: 'teacher_001'
  },
  {
    id: 'CHD003',
    org_id: 'ORG001',
    class_id: 'CLS002',
    class_name: '달님반',
    name: '이채원',
    birth: '2020-11-05',
    age: '만 5세',
    gender: '여',
    profile_color: '#F59E0B',
    status: 'enrolled',
    status_label: '재원중',
    enroll_date: '2023-03-03',
    expected_graduate: '2026',
    primary_parent_id: 'MBR013',
    primary_parent_name: '이수진',
    primary_parent_phone: '010-4444-5555',
    primary_parent_relation: '엄마',
    parent_count: 2,
    parents: [
      { member_id: 'MBR013', name: '이수진', phone: '010-4444-5555', relation: '엄마', account_type: 'primary', app_connected: true, last_access: '2026-03-18' },
      { member_id: 'MBR014', name: '이정훈', phone: '010-6666-7777', relation: '아빠', account_type: 'family', app_connected: false, last_access: null }
    ],
    reg_route: '학부모직접',
    allergy: ['유제품'],
    memo: '',
    created_at: '2023-03-03T10:00:00',
    updated_at: '2026-01-10T11:00:00',
    updated_by: 'org_admin_001'
  },
  {
    id: 'CHD004',
    org_id: 'ORG001',
    class_id: 'CLS003',
    class_name: '별빛반',
    name: '최민준',
    birth: '2019-05-12',
    age: '만 6세',
    gender: '남',
    profile_color: '#10B981',
    status: 'enrolled',
    status_label: '재원중',
    enroll_date: '2022-03-04',
    expected_graduate: '2026',
    primary_parent_id: 'MBR015',
    primary_parent_name: '최영희',
    primary_parent_phone: '010-8888-9999',
    primary_parent_relation: '엄마',
    parent_count: 3,
    parents: [
      { member_id: 'MBR015', name: '최영희', phone: '010-8888-9999', relation: '엄마', account_type: 'primary', app_connected: true, last_access: '2026-03-16' },
      { member_id: 'MBR016', name: '최재호', phone: '010-1111-2222', relation: '아빠', account_type: 'family', app_connected: true, last_access: '2026-03-12' },
      { member_id: 'MBR017', name: '김순자', phone: '010-3333-4444', relation: '할머니', account_type: 'family', app_connected: false, last_access: null }
    ],
    reg_route: '학부모직접',
    allergy: [],
    memo: '형제: 최민서(CHD005)',
    created_at: '2022-03-04T09:00:00',
    updated_at: '2026-02-20T16:00:00',
    updated_by: 'teacher_002'
  },
  {
    id: 'CHD005',
    org_id: 'ORG001',
    class_id: 'CLS001',
    class_name: '햇님반',
    name: '최민서',
    birth: '2021-09-01',
    age: '만 4세',
    gender: '여',
    profile_color: '#8B5CF6',
    status: 'enrolled',
    status_label: '재원중',
    enroll_date: '2024-09-02',
    expected_graduate: '2027',
    primary_parent_id: 'MBR015',
    primary_parent_name: '최영희',
    primary_parent_phone: '010-8888-9999',
    primary_parent_relation: '엄마',
    parent_count: 2,
    parents: [
      { member_id: 'MBR015', name: '최영희', phone: '010-8888-9999', relation: '엄마', account_type: 'primary', app_connected: true, last_access: '2026-03-16' },
      { member_id: 'MBR016', name: '최재호', phone: '010-1111-2222', relation: '아빠', account_type: 'family', app_connected: true, last_access: '2026-03-12' }
    ],
    reg_route: '학부모직접',
    allergy: [],
    memo: '형제: 최민준(CHD004) - 같은 보호자',
    created_at: '2024-09-02T09:00:00',
    updated_at: '2026-03-01T10:00:00',
    updated_by: 'teacher_001'
  },
  {
    id: 'CHD006',
    org_id: 'ORG001',
    class_id: 'CLS002',
    class_name: '달님반',
    name: '정하은',
    birth: '2020-02-14',
    age: '만 6세',
    gender: '여',
    profile_color: '#EF4444',
    status: 'withdrawn',
    status_label: '퇴소',
    enroll_date: '2023-03-03',
    graduate_date: '2026-02-28',
    expected_graduate: null,
    primary_parent_id: 'MBR018',
    primary_parent_name: '정미영',
    primary_parent_phone: '010-5555-6666',
    primary_parent_relation: '엄마',
    parent_count: 1,
    parents: [
      { member_id: 'MBR018', name: '정미영', phone: '010-5555-6666', relation: '엄마', account_type: 'primary', app_connected: false, last_access: '2026-02-20' }
    ],
    reg_route: '학부모직접',
    allergy: [],
    memo: '이사로 인한 퇴소',
    created_at: '2023-03-03T09:30:00',
    updated_at: '2026-02-28T17:00:00',
    updated_by: 'org_admin_001'
  }
];
```

---

## 10. 관계도 업데이트 (ERD 추가)

```
organizations ──1:N──→ children (org_id)
classes ──1:N──→ children (class_id)
members (parent) ──1:N──→ children (primary_parent_id)
children ──1:N──→ child_parents (연결 테이블, parents 배열로 표현)
members (parent) ──N:M──→ children (via child_parents)
```

---

## 11. 반 관리 메뉴 위치 변경

> 📌 **결정 사항 (2026-03-18)**: 반 관리는 콘텐츠 관리가 아닌 **기관관리** 그룹으로 이동

### 변경 전 (콘텐츠 관리 그룹)
```
콘텐츠 관리
├── 반 관리  ← 여기 있었음
├── 원아 관리
├── 알림장
...
```

### 변경 후 (기관관리 그룹)
```
기관관리
├── 기관정보 관리
└── 반 관리  ← 여기로 이동

원아 및 콘텐츠
├── 원아 관리
├── 알림장
├── 공지사항
├── 앨범
├── 일정 관리
├── 상담 관리
└── 투약의뢰서 관리
```

> 이유: 반은 기관 운영 구조에 해당하며, 콘텐츠(알림장, 앨범 등)와는 성격이 다름

---

## 12. operation-child.html 구현 스펙

### 12.1 파일 위치
- `src/pages/operation-child.html`

### 12.2 구현 목표
- 원아 목록 테이블 (정렬, 필터, 검색)
- KPI 카드 4개
- 반 필터 (역할별 분기)
- 보호자 기준 필터링
- 원아 상세/수정 슬라이드 패널
- 원아 등록 모달
- 상태 변경 모달

### 12.3 역할별 UI 분기 로직
```javascript
const userRole = sessionStorage.getItem('userRole');
const opContext = JSON.parse(sessionStorage.getItem('operationContext') || 'null');

// 현재 기관 컨텍스트 결정
const currentOrgId = opContext?.orgId          // 운영관리 컨텍스트 (통합관리자)
  ?? sessionStorage.getItem('orgId');           // 기관관리자/교사관리자 본인 기관

// 담당 반 목록 (교사만)
const assignedClasses = userRole === 'teacher'
  ? JSON.parse(sessionStorage.getItem('assignedClasses') || '[]')
  : null;
```
