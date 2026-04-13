# 🧠 프로젝트 브레인: 공지사항 기능 정의

> **최종 업데이트**: 2026-04-13 (댓글 수정 기능·상대 날짜 표시·role badge 추가 — 공지사항·알림장·대시보드 댓글 시스템 통일)
> **작업 범위**: 운영관리 내 공지사항 (`operation-announcement.html`)
> **연관 브레인**: `07-ROLE-PERMISSION.md`, `06-DESIGN-SYSTEM.md`, `08-OPERATION-MANAGEMENT.md`

---

## ⚡ QUICK REF
> 빌드 시 핵심 요약. 상세 스펙은 아래 섹션 참조.

- **파일**: `src/pages/oper/operation-announcement.html` ✅ v2 완료 (2026-04-06)
- **역할**: 기관 전체 대상 공지사항 발행·조회·관리
- **반 필터**: `<select>` 드롭다운 → 칩 버튼 형태 (역할별 분기)
- **리스트 컬럼**: 핀고정 / 제목(+인디케이터) / 카테고리 / 대상 / 작성자 / 작성일 / 조회 / 상태 / 액션
- **상태**: `게시중` / `예약발행` 두 가지만 (임시저장·종료 제거)
- **임시보관함**: `drafts[]` 별도 배열, 리스트 미노출, 액션바 버튼→모달
- **드로어**: 너비 520px, 2-뷰 구조 (기본 콘텐츠 뷰 ↔ 읽음현황 뷰)
- **쓰기 모달**: 너비 640px, 카테고리 칩·예약발행·댓글허용·사진·첨부·예문·맞춤법·예전글 기능 추가

---

## 1. 데이터 모델

### 1.1 notices[] 필드

```javascript
{
  id          : 'n1',
  pinned      : true,
  title       : '봄 소풍 안내',
  category    : '행사',            // CATEGORIES 상수 참조
  target      : '전체',
  author      : '김원장',
  createdDate : '2026.03.20',      // 게시기간 대신 작성일 단일 컬럼
  views       : 42,
  status      : '게시중',          // '게시중' | '예약발행'
  reservedAt  : null,              // 예약발행 시각 (예: '2026-04-10T09:00')
  hasPhoto    : true,
  hasAttachment: false,
  commentCount: 3,
  commentAllowed: true,
  content     : '...',
  readStatus  : [                  // 읽음현황 목록
    { name:'박엄마', role:'학부모', read:true,  readAt:'2026.03.20 09:15' },
    { name:'이아빠', role:'학부모', read:false, readAt:null }
  ],
  comments    : [                  // 댓글 목록
    {
      id:'c1', author:'박엄마', initials:'박', isTeacher:false,
      text:'감사합니다!', time:'2026.03.20 10:00',
      replies:[
        { id:'r1', author:'김원장', initials:'김', isTeacher:true,
          text:'네, 꼭 참석해 주세요!', time:'2026.03.20 10:30' }
      ]
    }
  ]
}
```

### 1.2 drafts[] 필드

```javascript
// 임시보관함 전용 — notices[]에 포함되지 않음
let drafts = [
  {
    id           : 'd1',
    title        : '소풍 일정 안내 (초안)',
    target       : '전체',
    category     : '행사',
    savedAt      : '2026.04.05 14:30',
    content      : '...',
    pinned       : false,
    commentAllowed: true,
    reservedAt   : null
  }
];
```

### 1.3 상수

```javascript
const CATEGORIES = ['일반공지','행사','준비물','상담','교육','기타'];

const CAT_COLORS = {
  '일반공지': { bg:'#F3F4F6', text:'#374151' },
  '행사':     { bg:'#EFF6FF', text:'#1D4ED8' },
  '준비물':   { bg:'#ECFDF5', text:'#065F46' },
  '상담':     { bg:'#FFF7ED', text:'#C2410C' },
  '교육':     { bg:'#F5F3FF', text:'#6D28D9' },
  '기타':     { bg:'#FDF2F8', text:'#9D174D' }
};

// 예문 템플릿 (자주쓰는 예문 패널에 사용)
const SAMPLE_TEMPLATES = {
  '일반공지': [{ title:'...', content:'...' }, ...],
  '행사':     [...], ...
};

// 예전 글 불러오기 데이터 (작년 동월 기준)
const PAST_SAMPLES = {
  '일반공지': [{ id:'p1', title:'...', content:'...', date:'2025.04' }, ...],
  '행사':     [...], ...
};

const PAST_COUNTS = { '일반공지':5, '행사':3, '준비물':2, '상담':1, '교육':2, '기타':1 };
```

---

## 2. 반 칩 필터

### 2.1 UI

```
[전체] [햇님반] [달님반] [별님반] [꽃님반]  ← 역할별 분기
```

- `org_admin` / `super(contextRole=org_admin)`: 전체 반 칩 표시
- `teacher`: 담당 반만 표시 (`assignedClasses` 세션 기준)
- 기본 선택: `전체` (activeClassChip = '')

### 2.2 CSS

```css
.chip-filter-row    { display:flex; gap:8px; flex-wrap:wrap; }
.filter-chip        { height:32px; padding:0 14px; border-radius:20px;
                      font-size:13px; font-weight:700; border:1.5px solid #D1D5DB;
                      background:#fff; cursor:pointer; }
.filter-chip:hover  { border-color:var(--accent); color:var(--accent); }
.filter-chip.active { background:var(--accent); border-color:var(--accent); color:#fff; }
```

### 2.3 관련 JS 함수

| 함수 | 역할 |
|------|------|
| `renderClassChips()` | 역할 기반 칩 목록 렌더링 |
| `selectClassChip(name)` | activeClassChip 갱신 → applyFilter() |
| `getClassList()` | 역할별 반 목록 반환 |

---

## 3. 리스트 테이블

### 3.1 컬럼 구성 (10열)

| # | 컬럼 | 내용 |
|---|------|------|
| 1 | — | 핀고정 아이콘 (📌) |
| 2 | — | 체크박스 |
| 3 | 제목 | 텍스트 + 인디케이터 뱃지 |
| 4 | 카테고리 | 컬러 뱃지 |
| 5 | 대상 | 반 이름 또는 전체 |
| 6 | 작성자 | 이름 |
| 7 | 작성일 | `createdDate` |
| 8 | 조회 | 숫자 |
| 9 | 상태 | 게시중 / 예약발행 뱃지 |
| 10 | 액션 | 수정 / 삭제 버튼 |

> ⚠️ `게시기간` 컬럼 삭제됨 (v1 → v2). `작성일` 단일 컬럼으로 대체.

### 3.2 제목 인디케이터 뱃지

```html
<!-- 제목 셀 내부 -->
<span class="notice-title-text">제목텍스트</span>
<span class="title-indicator-badge">📷</span>   <!-- hasPhoto -->
<span class="title-indicator-badge">📎</span>   <!-- hasAttachment -->
<span class="title-indicator-badge">💬3</span>  <!-- commentCount > 0 -->
```

```css
.title-indicator-badge { font-size:11px; margin-left:4px; color:#6B7280; }
```

### 3.3 상태 뱃지

| 상태 | 색상 | 아이콘 |
|------|------|--------|
| `게시중` | 초록 | — |
| `예약발행` | 파란 | 🕐 + reservedAt 날짜 표시 |

> v1의 `임시저장`, `종료` 상태는 완전 제거됨.

---

## 4. 상세 드로어

### 4.1 기본 스펙

| 항목 | 값 |
|------|---|
| 너비 | 520px |
| 구조 | 2-뷰 스왑 (`#drawerContentView` / `#drawerReadView`) |
| 기본 뷰 | 콘텐츠 뷰 (View A) |

### 4.2 View A — 콘텐츠 뷰

**헤더**

```
[반 뱃지]  [작성자 아바타·이름·교사]    [👁 읽음현황 5/10]
[중요글 뱃지 (pinned 시)]  [댓글허용 뱃지]
[제목 (h3)]
```

**바디**

```
공지 내용 (.notice-body-box)
────────────────
사진 그리드 (.photo-grid, hasPhoto 시 표시)
첨부파일 목록 (.file-list, hasAttachment 시 표시)
────────────────
댓글 (N)
  [댓글 목록 — 아바타 / 이름 / 시각 / 텍스트 / [답변] 버튼]
    └ 답변 클릭 → .reply-input-row 표시
[댓글 작성 입력창 + 전송 버튼]
```

### 4.3 View B — 읽음현황 뷰

**구조**

```
[← 뒤로]
수신자 읽음 현황 (제목)
────────────────────────────────────
[이름 / 역할 / 읽음 여부 / 읽은 시각] 테이블
(.read-recipients-table)
────────────────────────────────────
[미확인 학부모 알림 재전송(N명)] CTA 버튼
(.read-resend-btn)
```

**리셋 규칙**

| 이벤트 | resendBtn.disabled |
|--------|--------------------|
| 재전송 CTA 클릭 | `true` (비활성) |
| `←` 뒤로 클릭 | 변경 없음 (비활성 유지) |
| 드로어 닫기 → 재오픈 (`openDrawer`) | `false` (재활성) |

### 4.4 관련 JS 함수

| 함수 | 역할 |
|------|------|
| `openDrawer(id)` | 드로어 열기 + resendBtn 초기화 |
| `closeDrawer()` | 드로어 닫기 |
| `showReadPanel()` | View B 전환 + 수신자 목록 렌더 |
| `showContentView()` | View A 복귀 |
| `handleResend()` | 재전송 처리 + 토스트 + 버튼 비활성 |
| `formatCommentDate(createdAt, editedAt)` | 상대 날짜 반환 (오늘/어제/날짜 + (수정됨)) |
| `renderComments(comments)` | 댓글 목록 렌더 — role badge + 수정 버튼 포함 |
| `toggleReplyInput(commentId)` | 답변 입력창 토글 |
| `submitComment()` | 댓글 제출 — ISO createdAt 저장 |
| `editComment(id)` | 인라인 수정 UI (textarea + 저장/취소) |
| `editReply(commentId, replyId)` | 답변 인라인 수정 |

---

## 5. 임시보관함

### 5.1 흐름

```
[📂 임시보관함 (N)] 버튼 클릭
  → #draftModal 오픈
  → 드래프트 목록 표시 (제목 / 저장시각 / [이어쓰기] 버튼)
  → [이어쓰기] 클릭 → draftSourceId 세팅 → openWriteModal() → 폼 채우기
  → 수정 후 저장 → notices[]에 추가 + drafts[]에서 해당 항목 제거
```

### 5.2 draftSourceId 라이프사이클

| 시점 | 값 |
|------|----|
| `loadDraft(id)` 호출 | `draftSourceId = id` |
| `clearWriteForm()` 호출 | **건드리지 않음** |
| `closeWriteModal()` 호출 | `draftSourceId = null` |
| `saveNotice()` 호출 | `draftSourceId = null` |

> ⚠️ `clearWriteForm()`은 `openWriteModal()` 내부에서 호출되므로, `loadDraft()`에서 `draftSourceId`를 세팅한 후 `openWriteModal()`을 호출하는 순서를 반드시 지켜야 한다.

### 5.3 관련 JS 함수

| 함수 | 역할 |
|------|------|
| `openDraftModal()` | 임시보관함 모달 오픈 |
| `closeDraftModal()` | 모달 닫기 |
| `renderDraftList()` | drafts[] 렌더 |
| `loadDraft(id)` | 선택 드래프트 → 쓰기 모달 프리필 |
| `saveToDraftBox()` | 현재 폼 → drafts[] 저장/덮어쓰기 |
| `updateDraftCount()` | 액션바 카운트 뱃지 갱신 |

---

## 6. 쓰기 모달

### 6.1 기본 스펙

| 항목 | 값 |
|------|---|
| 너비 | 640px (`.modal-box.wide`) |
| 트리거 | 액션바 `[+ 공지 작성]` 버튼 |

### 6.2 추가된 UI 요소

| 요소 | 설명 |
|------|------|
| 카테고리 칩 | `[일반공지][행사][준비물][상담][교육][기타]` 단일 선택, 기본값 `일반공지` |
| 예약발행 토글 | `.toggle-blue` + `datetime-local` 입력 (토글 ON 시 표시) |
| 댓글허용 토글 | `.toggle-blue`, 기본값 ON |
| 사진/동영상 업로드 | `.dropzone` + `.upload-preview-item` 미리보기 (`photoFiles[]`) |
| 첨부파일 업로드 | 동일 패턴 (`attachFiles[]`) |
| 자주쓰는 예문 | `#samplePanel` 토글, 카테고리 칩 + 예문 목록, 클릭 시 title/content 자동 채우기 |
| 맞춤법 검사 | 버튼 클릭 → 토스트 "맞춤법 검사 완료. 이상 없습니다." |
| 예전 글 불러오기 | collapsible `.past-section`, 작년 동월 기준, 카테고리 칩 필터 |

### 6.3 예전 글 불러오기 흐름

```
[📅 예전 글 불러오기] 헤더 클릭 → .past-section 펼침
  카테고리 칩 클릭 → 해당 카테고리 항목 목록 표시
  항목 클릭 → .past-preview-modal 중앙 모달 오픈
    (카테고리 뱃지 / 제목 / 전체 내용 표시)
    [불러오기] 버튼 클릭
      → 모달 닫기
      → activeCategory 칩 반영
      → mTitle / mContent 채우기
```

### 6.4 저장 시 상태 결정

```javascript
status = (mReserveToggle.checked && mReserveTime.value) ? '예약발행' : '게시중';
```

### 6.5 관련 JS 함수

| 함수 | 역할 |
|------|------|
| `openWriteModal(notice?)` | 모달 오픈 (수정 시 notice 전달) |
| `closeWriteModal()` | 모달 닫기 + draftSourceId 초기화 |
| `clearWriteForm()` | 폼 초기화 (draftSourceId 제외) |
| `saveNotice()` | 저장 처리 → notices[] 추가/수정 + draft 제거 |
| `onReserveToggleChange()` | 예약 시간 입력 행 표시/숨김 |
| `onPhotoFilesChange()` | 사진 파일 선택 처리 |
| `removeUploadItem(type, idx)` | 업로드 미리보기 항목 제거 |
| `toggleSamplePanel()` | 예문 패널 토글 |
| `selectSampleCat(cat)` | 예문 카테고리 선택 |
| `renderSampleList()` | 예문 목록 렌더 |
| `insertSampleText(idx)` | 예문 클릭 → 폼 채우기 |
| `checkSpelling()` | 맞춤법 검사 → 토스트 |
| `togglePastSection()` | 예전 글 섹션 펼침/닫힘 |
| `selectPastCat(cat)` | 예전 글 카테고리 선택 |
| `renderPastItems()` | 예전 글 목록 렌더 |
| `openPastPreview(id)` | 예전 글 중앙 미리보기 모달 오픈 |
| `closePastPreview()` | 미리보기 모달 닫기 |
| `loadPastNotice(id)` | 예전 글 불러오기 → 카테고리+내용 채우기 |

---

## 7. 상태 및 컬럼 정리 (v1 → v2 변경사항)

| 항목 | v1 | v2 |
|------|----|----|
| 반 필터 UI | `<select>` 드롭다운 | 칩 버튼 (.filter-chip) |
| 게시기간 컬럼 | O | 제거 (작성일로 대체) |
| 공지 상태 | 게시중 / 임시저장 / 종료 / 예약발행 | **게시중 / 예약발행** (2가지만) |
| 임시저장 | notices[]에 포함, 상태값으로 관리 | drafts[] 별도 배열 + 임시보관함 모달로 분리 |
| 상세 드로어 너비 | 420px | 520px |
| 쓰기 모달 너비 | 560px | 640px |
| 카테고리 | 없음 | 6가지 칩 선택 (일반공지/행사/준비물/상담/교육/기타) |
| 읽음현황 | 없음 | 드로어 내 View B 패널 |
| 댓글 | 없음 | 드로어 내 댓글 목록 + 작성 |
| 예약발행 | 없음 | 쓰기 모달 토글 + datetime-local |
| 파일 업로드 | 없음 | dropzone 사진/첨부파일 (mock) |
| 예문/맞춤법 | 없음 | 샘플 패널 + 토스트 |
| 예전 글 불러오기 | 없음 | collapsible 섹션 + 미리보기 모달 |

---

## 8. 역할별 분기

| 기능 | org_admin | teacher |
|------|-----------|---------|
| 칩 필터 | 전체 반 칩 표시 | 담당 반만 표시 |
| 공지 목록 | 전체 반 노출 | 담당 반 및 전체 대상만 |
| 작성/수정/삭제 | O | O (본인 작성 기준) |
| 읽음현황 | O | O |

---

## 9. CSS 주요 클래스 목록

| 클래스 | 용도 |
|--------|------|
| `.chip-filter-row` | 반 칩 필터 컨테이너 |
| `.filter-chip` / `.filter-chip.active` | 반 선택 칩 |
| `.cat-badge` | 카테고리 인라인 뱃지 (테이블) |
| `.cat-chip-row` / `.cat-chip` / `.cat-chip.active` | 쓰기 모달 카테고리 칩 |
| `.title-indicators` / `.title-indicator-badge` | 제목 인디케이터 |
| `.toggle-blue` | 파란 토글 (예약발행, 댓글허용) |
| `.reserve-time-row.visible` | 예약 시간 입력 행 표시 상태 |
| `.drawer-author-block` | 드로어 헤더 작성자 블록 |
| `.read-status-btn` | 읽음현황 버튼 |
| `.drawer-meta-row` | 핀고정/댓글허용 뱃지 행 |
| `.photo-grid` | 드로어 사진 3열 그리드 |
| `.file-list` | 드로어 첨부파일 목록 |
| `.comment-list` | 댓글 목록 |
| `.comment-avatar.teacher` | 교사 아바타 (accent 배경) |
| `.comment-role-badge` / `.crb-teacher` / `.crb-parent` | 댓글 역할 뱃지 |
| `.comment-edit-btn` | 수정 버튼 (본인 댓글만 표시) |
| `.comment-edit-area` | 인라인 수정 textarea |
| `.comment-edit-ctrl` / `.comment-edit-save` / `.comment-edit-cancel` | 수정 저장/취소 컨트롤 |
| `.comment-edited` | `(수정됨)` 인디케이터 |
| `.comment-reply-wrap` / `.reply-btn` | 답변 영역 |
| `.read-panel-back` | 읽음현황 뒤로 버튼 |
| `.read-recipients-table` | 읽음현황 수신자 테이블 |
| `.read-resend-btn` | 재전송 CTA 버튼 |
| `.read-stat-box` | 읽음 통계 박스 |
| `.dropzone` | 파일 업로드 영역 |
| `.upload-preview-item` | 업로드 미리보기 항목 |
| `.sample-panel.visible` | 예문 패널 표시 상태 |
| `.past-section.open` | 예전 글 섹션 펼침 상태 |
| `.past-item` | 예전 글 목록 항목 |
| `.past-preview-modal-box` | 예전 글 중앙 미리보기 모달 |
| `.draft-modal-box` | 임시보관함 모달 |
| `.draft-item` | 드래프트 목록 항목 |
| `.modal-box.wide` | 쓰기 모달 640px 확장형 |
