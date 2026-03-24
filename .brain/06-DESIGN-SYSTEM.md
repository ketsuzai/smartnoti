# 🧠 프로젝트 브레인: 디자인 시스템 & 공통 스타일 가이드

> **최종 업데이트**: 2026-03-18

---

## 1. 색상 시스템 (CSS Variables)

### 1.1 기본 색상
| 변수명 | 값 | 용도 |
|--------|------|------|
| `--bg-base` | #F4F5F7 | 페이지 배경 |
| `--bg-surface` | #FFFFFF | 카드/패널 배경 |
| `--bg-sidebar` | #1A1D2E | 사이드바 배경 |
| `--bg-sidebar-hover` | #252840 | 사이드바 호버 |
| `--bg-sidebar-active` | #2E3250 | 사이드바 활성 |

### 1.2 액센트 색상
| 변수명 | 값 | 용도 |
|--------|------|------|
| `--accent-primary` / `--accent` | #4361EE | 주요 액센트 (버튼, 링크) |
| `--accent-primary-light` / `--accent-light` | #EEF1FD | 액센트 라이트 배경 |
| `--accent-secondary` | #3A0CA3 | 보조 액센트 |

### 1.3 텍스트 색상
| 변수명 | 값 | 용도 |
|--------|------|------|
| `--text-primary` | #1A1D2E | 주요 텍스트 |
| `--text-secondary` | #6B7280 | 보조 텍스트 |
| `--text-muted` | #9CA3AF | 비활성 텍스트 |
| `--text-sidebar` | #C5C9D6 | 사이드바 텍스트 |

### 1.4 보더 색상
| 변수명 | 값 | 용도 |
|--------|------|------|
| `--border-light` | #E5E7EB | 기본 보더 |
| `--border-medium` | #D1D5DB | 강조 보더 |

### 1.5 시멘틱 색상
| 색상 | 변수 | 배경 | 텍스트 | 용도 |
|------|------|------|--------|------|
| 초록 | `--green` #10B981 | `--green-bg` #D1FAE5 | `--green-text` #065F46 | 성공, 활성, 승인 |
| 파랑 | `--blue` #3B82F6 | `--blue-bg` #DBEAFE | `--blue-text` #1E40AF | 정보, 재원중 |
| 황색 | `--amber` #F59E0B | `--amber-bg` #FEF3C7 | `--amber-text` #92400E | 경고, 대기 |
| 빨강 | `--red` #EF4444 | `--red-bg` #FEE2E2 | `--red-text` #991B1B | 위험, 거절, 탈퇴 |
| 보라 | `--purple` #8B5CF6 | `--purple-bg` #EDE9FE | `--purple-text` #5B21B6 | 특수, 졸업 |
| 시안 | `--cyan` #06B6D4 | `--cyan-bg` #CFFAFE | `--cyan-text` #164E63 | 보조 정보 |

---

## 2. 그림자 시스템
| 변수명 | 값 | 용도 |
|--------|------|------|
| `--shadow-sm` | 0 1px 2px rgba(0,0,0,0.05) | 미세한 그림자 |
| `--shadow-md` | 0 4px 12px rgba(0,0,0,0.08) | 중간 그림자 |
| `--shadow-lg` | 0 8px 28~32px rgba(0,0,0,0.10~12) | 큰 그림자 |
| `--shadow-modal` | 0 20px 60px rgba(0,0,0,0.18) | 모달 그림자 |

---

## 3. 테두리 라운딩
| 변수명 | 값 | 용도 |
|--------|------|------|
| `--radius-sm` | 6px | 작은 요소 (태그, 뱃지) |
| `--radius-md` | 8~10px | 버튼, 입력필드 |
| `--radius-lg` | 12~14px | 카드 |
| `--radius-xl` | 16~18px | 큰 카드, 모달 |

---

## 4. 레이아웃 상수
| 변수명 | 값 |
|--------|------|
| `--sidebar-width` | 240px |
| `--header-height` | 60px |

---

## 5. 폰트
| 폰트 | 용도 |
|------|------|
| Pretendard (300~800) | 메인 UI 폰트 |
| JetBrains Mono (400, 500) | 코드, 날짜, 숫자 |

---

## 6. 공통 UI 컴포넌트 패턴

### 6.1 상태 뱃지 (Status Badge)
```html
<span class="sbadge" style="background:var(--bg);color:var(--text)">
  <span class="sdot" style="background:var(--dot)"></span>
  상태명
</span>
```

### 6.2 필터 셀렉트
```html
<div class="fsel-wrap">
  <select class="fsel">...</select>
</div>
```

### 6.3 검색 입력
```html
<div class="search-wrap">
  <input class="search-inp" placeholder="검색...">
  <button class="search-btn">🔍</button>
</div>
```

### 6.4 토스트 알림
```javascript
function showToast(msg) { /* 하단 우측, 2.6초 자동 소멸 */ }
```

### 6.5 확인 모달
```html
<div class="cm-overlay" id="cmOverlay">
  <div class="cm-box">
    <div class="cm-title">제목</div>
    <div class="cm-msg">메시지</div>
    <div class="cm-btns">
      <button class="cm-btn cm-cancel">취소</button>
      <button class="cm-btn" id="cmOk">확인</button>
    </div>
  </div>
</div>
```

### 6.6 사이드 디테일 패널
- 우측에서 슬라이드인
- overlay(반투명 배경) + panel 조합
- 닫기 버튼 + ESC 키 닫기
