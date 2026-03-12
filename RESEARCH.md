# Next.js 웹 슬라이드 프레젠테이션 구현 리서치

> Vercel 배포를 전제로 한 Next.js (App Router) 기반 웹 프레젠테이션 구현 방법 조사
> 결과물은 단일 HTML 파일로 출력

---

## 0. 디자인 스타일 가이드 (샘플 분석 기준)

> `sample/` 폴더의 8개 스크린샷을 분석하여 도출한 디자인 시스템

### 컬러 팔레트

| 토큰 | 값 | 용도 |
|------|-----|------|
| `--bg-primary` | `#0a0a0a` | 슬라이드 배경 (순수 블랙에 가까운 다크) |
| `--bg-card` | `#141414` | 카드/박스 배경 |
| `--bg-card-hover` | `#1a1a1a` | 카드 호버/강조 배경 |
| `--border-card` | `#262626` | 카드 테두리 (미세한 경계선) |
| `--accent` | `#4ECCA3` | **민트 그린 액센트** — 강조 텍스트, 키워드, 라벨, 구분선 |
| `--accent-dim` | `#3BA88A` | 민트 그린 어두운 변형 (호버, 보조) |
| `--text-primary` | `#FFFFFF` | **순수 화이트** — 메인 제목, 헤딩 텍스트 |
| `--text-secondary` | `#B0B0B0` | 중립 라이트 그레이 — 본문, 설명 텍스트 |
| `--text-muted` | `#787878` | 중립 그레이 — 서브타이틀, 라벨, 작은 텍스트 |
| `--text-code` | `#E0E0E0` | 코드 블록 내 텍스트 (밝은 그레이) |

### 타이포그래피

```css
/* 한글 폰트 최적화 */
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&family=Inter:wght@400;600;700&family=JetBrains+Mono:wght@400;500&display=swap');

:root {
  --font-body: 'Noto Sans KR', 'Inter', sans-serif;
  --font-code: 'JetBrains Mono', 'Noto Sans KR', monospace;
}
```

| 요소 | 폰트 | 크기 | 두께 | 색상 |
|------|------|------|------|------|
| 슬라이드 대제목 | Noto Sans KR | 3rem~4rem (48~64px) | 700 (bold) | `--text-primary` (화이트) |
| 강조 키워드 | Noto Sans KR | 대제목과 동일 | 700 | `--accent` (민트 그린) |
| 섹션 라벨 (> BEFORE 등) | Inter | 0.875rem (14px) | 600 | `--accent` (민트 그린), letter-spacing: 0.1em |
| 본문/설명 | Noto Sans KR | 1.125rem (18px) | 400 | `--text-secondary` (라이트 그레이) |
| 코드 블록 | JetBrains Mono | 0.95rem (15px) | 400 | `--text-code` |
| 카드 내 텍스트 | Noto Sans KR | 1rem (16px) | 400 | `--text-primary` |
| 카드 라벨 (@viewer_01 등) | Inter | 0.75rem (12px) | 400 | `--text-muted` |
| 넘버링 (01, 02, 03) | Inter | 0.875rem (14px) | 600 | `--accent` (민트 그린) |

### 레이아웃 원칙

- **콘텐츠 max-width: 680px** — 화면 중앙에 좁은 폭으로 집중
- **수직 중앙 정렬** — 모든 슬라이드 콘텐츠가 뷰포트 정중앙
- **여유로운 여백** — 요소 간 간격 넉넉하게 (2rem~3rem)
- **16:9 비율 고려 없음** — 풀스크린 뷰포트를 꽉 채우는 방식

### 컴포넌트 스타일

#### 카드/박스

```css
.card {
  background: var(--bg-card);           /* #141414 */
  border: 1px solid var(--border-card); /* #262626 */
  border-radius: 12px;
  padding: 1.5rem 2rem;
}
```

#### 플로우 다이어그램 (→ 화살표 연결)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  카드 제목    │  →  │  카드 제목    │  →  │  카드 제목    │
│  설명 텍스트  │     │  설명 텍스트  │     │  설명 텍스트  │
└─────────────┘     └─────────────┘     └─────────────┘
```

- 카드: `--bg-card` 배경 + `--border-card` 테두리 + `border-radius: 12px`
- 카드 제목: `--accent` (민트 그린) 색상
- 화살표 (→): `--text-muted` 색상, 카드 사이에 위치

#### 코드 블록

```css
.code-block {
  background: #0d0d0d;
  border: 1px solid var(--border-card);
  border-radius: 12px;
  overflow: hidden;
}
/* macOS 창 크롬 (빨강/노랑/초록 dot) */
.code-block-header {
  padding: 12px 16px;
  display: flex;
  gap: 8px;
}
.dot { width: 12px; height: 12px; border-radius: 50%; }
.dot--red { background: #ff5f57; }
.dot--yellow { background: #febc2e; }
.dot--green { background: #28c840; }
```

#### 넘버링 리스트

```css
.numbered-item {
  background: var(--bg-card);
  border: 1px solid var(--border-card);
  border-radius: 8px;
  padding: 1rem 1.5rem;
  display: flex;
  align-items: center;
  gap: 1rem;
}
.numbered-item__number {
  color: var(--accent);  /* 민트 그린 */
  font-family: 'Inter', sans-serif;
  font-weight: 600;
}
```

#### 인용/질문 카드

```css
.quote-card {
  background: var(--bg-card);
  border-left: 3px solid var(--accent);  /* 민트 그린 왼쪽 보더 */
  border-radius: 0 8px 8px 0;
  padding: 1rem 1.5rem;
}
.quote-card__label {
  color: var(--text-muted);
  font-size: 0.75rem;
}
```

#### 구분선 (제목 아래 짧은 바)

```css
.divider {
  width: 40px;
  height: 3px;
  background: var(--accent);  /* 민트 그린 */
  margin: 1rem auto;
}
```

### 배경 효과

- 미세한 **그리드 패턴** — 배경에 희미한 격자선 (`rgba(255,255,255,0.02)` 수준)
- 그리드 간격: 약 60~80px
- 순수 CSS `background-image: linear-gradient(...)` 또는 `repeating-linear-gradient`로 구현

```css
.slide-bg {
  background-color: var(--bg-primary);
  background-image:
    linear-gradient(rgba(255,255,255,0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255,255,255,0.03) 1px, transparent 1px);
  background-size: 70px 70px;
}
```

---

## 1. 슬라이드 레이아웃 & DOM 구조

### 추천구조

#### 파일 구조 (단일 HTML 출력 기준)

최종 결과물은 **단일 HTML 파일**이므로, 개발은 Next.js App Router 구조로 하되 `output: 'export'`로 정적 빌드합니다.

```
src/
├── app/
│   ├── layout.tsx              # 루트 레이아웃 (풀스크린 리셋)
│   ├── page.tsx                # 프레젠테이션 진입점
│   └── globals.css             # 글로벌 스타일 + CSS 변수
├── components/
│   ├── SlideContainer.tsx      # 슬라이드 전체 래퍼 (JS 전환 엔진)
│   ├── Slide.tsx               # 개별 슬라이드 래퍼
│   ├── slides/                 # 슬라이드별 컴포넌트
│   └── ui/
│       ├── SlideNumber.tsx
│       └── ProgressBar.tsx
├── hooks/
│   └── useSlideNavigation.ts
└── styles/
    └── slide.module.css
```

#### CSS 전략

| 레벨 | 방식 | 이유 |
|------|------|------|
| 슬라이드 배치 | **JS fixed + absolute** | 스크롤 스냅 대신 프로그래밍 방식으로 정밀 제어 |
| 내부 콘텐츠 | **Flexbox** (`flex-direction: column; align-items: center`) | 수직 중앙 정렬 + 680px 폭 제한 |
| 오버레이 (번호, 프로그레스바) | `position: fixed` | 슬라이드 전환과 독립 |

### 핵심패턴

#### 뷰포트 풀스크린 리셋

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html, body {
  height: 100%;
  overflow: hidden;  /* 바디 스크롤 완전 차단 */
  background: var(--bg-primary);
}
```

#### 슬라이드 컨테이너 (JS fixed+absolute 전환 방식)

스크롤 스냅 대신, 모든 슬라이드를 `position: fixed`로 겹쳐놓고 JS로 활성 슬라이드만 표시합니다.

```css
.slide {
  position: fixed;
  inset: 0;                    /* 위아래좌우 여백 0 — 브라우저 꽉 채움 */
  width: 100vw;
  height: 100vh;
  height: 100dvh;              /* 모바일 주소창 대응 */
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: var(--bg-primary);
  opacity: 0;
  pointer-events: none;
  transition: opacity 500ms ease-in-out;
}

.slide--active {
  opacity: 1;
  pointer-events: auto;
  z-index: 1;
}
```

#### 콘텐츠 폭 제한 (680px)

```css
.slide-content {
  max-width: 680px;
  width: 100%;
  padding: 0 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
}
```

#### 전체 조합

```tsx
// SlideContainer.tsx
'use client';
import { useState, useEffect } from 'react';

export function SlideContainer({ slides }: { slides: React.ReactNode[] }) {
  const [current, setCurrent] = useState(0);

  return (
    <div className="presentation">
      {slides.map((slide, i) => (
        <section
          key={i}
          className={`slide slide-bg ${i === current ? 'slide--active' : ''}`}
          aria-hidden={i !== current}
        >
          <div className="slide-content">
            {slide}
          </div>
        </section>
      ))}
    </div>
  );
}
```

### 주의사항

- **스크롤 스냅 사용하지 않음**: JS `fixed+absolute` 방식으로 전환. 스크롤 이벤트 버그, Safari 스냅 불일치 문제 회피
- **`100dvh` 폴백 필수**: `height: 100vh; height: 100dvh;` 순서로 선언 (iOS Safari 주소창 대응)
- **`overflow: hidden` 필수**: `html`, `body` 모두에 적용. 누락 시 슬라이드 바깥 스크롤 발생
- **`"use client"` 필수**: 이벤트 핸들러, `useState`, `useEffect` 사용 컴포넌트에 선언
- **Vercel 배포**: `output: 'export'`로 정적 빌드 → CDN 엣지 서빙

---

## 2. 키보드 & 마우스 네비게이션

### 추천구조

Context + 커스텀 훅 분리 아키텍처:

```
SlideProvider (Context)        ← currentSlide, totalSlides, goToSlide, next, prev
├── useSlideNavigation()       ← 키보드 이벤트 (ArrowLeft/Right, Space, PageUp/Down)
├── useSwipeGesture()          ← 터치/스와이프 제스처
├── useHashSync()              ← URL hash 동기화 (#slide-1)
├── ProgressBar                ← 진행 표시바 (골드 액센트)
└── SlideCounter               ← 슬라이드 번호 (아이보리 muted)
```

### 핵심패턴

#### SlideProvider (Context)

```tsx
'use client';
import { createContext, useContext, useState, useCallback, ReactNode } from 'react';

interface SlideContextType {
  currentSlide: number;
  totalSlides: number;
  goToSlide: (index: number) => void;
  nextSlide: () => void;
  prevSlide: () => void;
}

const SlideContext = createContext<SlideContextType | null>(null);

export function SlideProvider({ children, totalSlides }: { children: ReactNode; totalSlides: number }) {
  const [currentSlide, setCurrentSlide] = useState(0);
  const goToSlide = useCallback((i: number) => setCurrentSlide(Math.max(0, Math.min(i, totalSlides - 1))), [totalSlides]);
  const nextSlide = useCallback(() => goToSlide(currentSlide + 1), [currentSlide, goToSlide]);
  const prevSlide = useCallback(() => goToSlide(currentSlide - 1), [currentSlide, goToSlide]);

  return (
    <SlideContext.Provider value={{ currentSlide, totalSlides, goToSlide, nextSlide, prevSlide }}>
      {children}
    </SlideContext.Provider>
  );
}

export const useSlideContext = () => {
  const ctx = useContext(SlideContext);
  if (!ctx) throw new Error('useSlideContext must be used within SlideProvider');
  return ctx;
};
```

#### 키보드 이벤트 처리

```tsx
'use client';
import { useEffect, useCallback } from 'react';
import { useSlideContext } from './SlideProvider';

export function useSlideNavigation() {
  const { nextSlide, prevSlide } = useSlideContext();

  const handleKeyDown = useCallback((e: KeyboardEvent) => {
    const target = e.target as HTMLElement;
    if (target.tagName === 'INPUT' || target.tagName === 'TEXTAREA' || target.isContentEditable) return;

    switch (e.key) {
      case 'ArrowRight': case 'ArrowDown': case ' ': case 'PageDown':
        e.preventDefault(); nextSlide(); break;
      case 'ArrowLeft': case 'ArrowUp': case 'PageUp':
        e.preventDefault(); prevSlide(); break;
    }
  }, [nextSlide, prevSlide]);

  useEffect(() => {
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [handleKeyDown]);
}
```

#### URL Hash 동기화

```tsx
'use client';
import { useEffect } from 'react';
import { useSlideContext } from './SlideProvider';

export function useHashSync() {
  const { currentSlide, goToSlide } = useSlideContext();

  // 슬라이드 변경 → hash 업데이트 (replaceState로 히스토리 오염 방지)
  useEffect(() => {
    window.history.replaceState(null, '', `#slide-${currentSlide + 1}`);
  }, [currentSlide]);

  // 초기 로드 시 hash 파싱
  useEffect(() => {
    const match = window.location.hash.match(/^#slide-(\d+)$/);
    if (match) goToSlide(parseInt(match[1], 10) - 1);
  }, [goToSlide]);

  // 뒤로/앞으로 버튼 대응
  useEffect(() => {
    const handler = () => {
      const match = window.location.hash.match(/^#slide-(\d+)$/);
      if (match) goToSlide(parseInt(match[1], 10) - 1);
    };
    window.addEventListener('hashchange', handler);
    return () => window.removeEventListener('hashchange', handler);
  }, [goToSlide]);
}
```

#### 터치/스와이프 제스처 (의존성 없음)

```tsx
'use client';
import { useRef, useEffect, useCallback } from 'react';
import { useSlideContext } from './SlideProvider';

export function useSwipeGesture(ref: React.RefObject<HTMLElement | null>, threshold = 50) {
  const { nextSlide, prevSlide } = useSlideContext();
  const touchStart = useRef<{ x: number; y: number; time: number } | null>(null);

  const handleTouchStart = useCallback((e: TouchEvent) => {
    const t = e.touches[0];
    touchStart.current = { x: t.clientX, y: t.clientY, time: Date.now() };
  }, []);

  const handleTouchEnd = useCallback((e: TouchEvent) => {
    if (!touchStart.current) return;
    const t = e.changedTouches[0];
    const dx = t.clientX - touchStart.current.x;
    const dy = t.clientY - touchStart.current.y;
    if (Date.now() - touchStart.current.time > 500) return;
    if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > threshold) {
      dx < 0 ? nextSlide() : prevSlide();
    }
    touchStart.current = null;
  }, [nextSlide, prevSlide, threshold]);

  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    el.addEventListener('touchstart', handleTouchStart, { passive: true });
    el.addEventListener('touchend', handleTouchEnd, { passive: true });
    return () => {
      el.removeEventListener('touchstart', handleTouchStart);
      el.removeEventListener('touchend', handleTouchEnd);
    };
  }, [ref, handleTouchStart, handleTouchEnd]);
}
```

> **대안**: `react-swipeable` (~2KB gzipped) — `trackMouse: true`로 마우스 드래그도 지원

#### Progress Bar (골드 액센트)

```tsx
'use client';
import { useSlideContext } from './SlideProvider';

export function ProgressBar() {
  const { currentSlide, totalSlides } = useSlideContext();
  const progress = ((currentSlide + 1) / totalSlides) * 100;

  return (
    <div role="progressbar" aria-valuenow={currentSlide + 1} aria-valuemin={1} aria-valuemax={totalSlides}
         style={{ position: 'fixed', bottom: 0, left: 0, width: '100%', height: '3px', background: 'rgba(78,204,163,0.15)', zIndex: 50 }}>
      <div style={{ height: '100%', width: `${progress}%`, background: 'var(--accent)', transition: 'width 300ms ease-in-out' }} />
    </div>
  );
}
```

### 주의사항

- **SSR 안전**: 모든 `window`/`document` 접근은 `useEffect` 내부에서만
- **접근성 (WAI-ARIA Carousel)**: 컨테이너 `aria-roledescription="carousel"`, 각 슬라이드 `aria-roledescription="slide"`, 비활성 슬라이드 `aria-hidden="true"`
- **Space 키 충돌**: 버튼/링크 위에서 Space 누르면 기본 동작과 충돌 → `e.target` 검사 필수
- **replaceState vs pushState**: `replaceState` 추천 (30장 슬라이드 → 뒤로가기 30번 방지)
- **passive 리스너**: `touchstart`에 `{ passive: true }` 사용 시 `preventDefault()` 불가 → 스크롤 방지가 필요하면 `{ passive: false }`로 변경
- **hash fragment**: 서버로 전송되지 않음 → Vercel 미들웨어/서버 함수에서 처리 불가, 순수 클라이언트 처리

---

## 3. 슬라이드 전환 애니메이션

### 추천구조

#### 기술 선택: CSS Transitions (fixed+absolute 방식에 최적)

스크롤 스냅이 아닌 JS 전환 방식이므로, CSS Transitions로 `opacity`/`transform`을 제어하는 것이 가장 효율적입니다.

| 기준 | CSS Transitions | CSS Animations | Web Animations API |
|------|----------------|----------------|-------------------|
| 적합한 경우 | 단순 상태 전환 (fade) | 복잡한 키프레임 (zoom) | 동적 제어 필요 시 |
| 제어 수준 | 낮음 (시작/끝만) | 중간 (키프레임) | 높음 (play/pause/reverse) |
| 코드 복잡도 | 낮음 | 중간 | 높음 |

**결론**: CSS Transitions으로 fade 전환을 기본으로 사용. 방향성 slide나 zoom이 필요하면 CSS Animations 보조.

#### 상태 관리

```typescript
interface SlideState {
  currentIndex: number;
  previousIndex: number | null;   // 전환 중 이전 슬라이드 동시 렌더링용
  direction: 'forward' | 'backward';
  isAnimating: boolean;           // 이벤트 잠금 플래그
}
```

`useReducer`로 통합 관리하여 `isAnimating` 상태일 때 모든 네비게이션 액션 무시.

### 핵심패턴

#### Fade 전환 (기본 — fixed+absolute 방식)

```css
.slide {
  position: fixed;
  inset: 0;
  opacity: 0;
  pointer-events: none;
  transition: opacity 500ms ease-in-out;
}
.slide--active {
  opacity: 1;
  pointer-events: auto;
  z-index: 1;
}
```

#### Slide 전환 (방향별)

```css
.slide--slide {
  transform: translateX(100%);
  transition: transform 500ms cubic-bezier(0.25, 0.46, 0.45, 0.94);
  will-change: transform;
}
.slide--slide.slide--active { transform: translateX(0); }
.slide--slide.slide--exiting-forward { transform: translateX(-100%); }
.slide--slide.slide--exiting-backward { transform: translateX(100%); }
```

#### Zoom 전환

```css
@keyframes zoomIn {
  from { opacity: 0; transform: scale(0.85); }
  to   { opacity: 1; transform: scale(1); }
}
@keyframes zoomOut {
  from { opacity: 1; transform: scale(1); }
  to   { opacity: 0; transform: scale(1.15); }
}
.slide--zoom.slide--active  { animation: zoomIn 500ms ease forwards; }
.slide--zoom.slide--exiting { animation: zoomOut 500ms ease forwards; }
```

#### 이벤트 잠금 (useReducer)

```typescript
function slideReducer(state: SlideState, action: SlideAction): SlideState {
  switch (action.type) {
    case 'NEXT':
      if (state.isAnimating) return state;  // 잠금
      return { previousIndex: state.currentIndex, currentIndex: state.currentIndex + 1, direction: 'forward', isAnimating: true };
    case 'ANIMATION_END':
      return { ...state, isAnimating: false, previousIndex: null };
    // ...
  }
}
```

#### 전환 래퍼 (이전/다음 동시 렌더링)

fixed+absolute 방식에서는 모든 슬라이드가 이미 DOM에 있으므로, 클래스 토글만으로 전환됩니다.

```tsx
{slides.map((slide, i) => (
  <section
    key={i}
    className={`slide slide-bg ${i === current ? 'slide--active' : ''}`}
    onTransitionEnd={() => { if (i === current) onAnimationEnd(); }}
  >
    <div className="slide-content">{slide}</div>
  </section>
))}
```

#### GPU 가속 동적 관리

```typescript
// 애니메이션 직전에 will-change 적용, 완료 후 해제
element.style.willChange = 'transform, opacity';  // 전환 시작
// onTransitionEnd:
element.style.willChange = 'auto';                 // 전환 완료
```

### 주의사항

- **compositor-only 속성만 애니메이션**: `transform`, `opacity`만 사용. `top/left/width/height`는 layout 트리거 → 60fps 불가
- **`will-change` 남용 금지**: MDN 권고 — 성능 문제 있을 때만 사용, 스타일시트에 고정하지 말 것. 과도 사용 시 GPU 메모리 부족
- **`prefers-reduced-motion`**: 반드시 존중. `0ms`가 아닌 `0.01ms` 설정 (`transitionend` 이벤트 정상 발생 보장)

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

- **View Transitions API**: Chrome 111+, Safari 18+, Firefox 144+에서 SPA 지원 안정화. Progressive enhancement로 검토 가능
- **Framer Motion**: ~30KB 추가. 네이티브 CSS로 충분하면 불필요

---

## 4. 슬라이드 내 콘텐츠 애니메이션 (빌드 효과)

### 추천구조

두 가지 레벨의 콘텐츠 애니메이션을 지원합니다:

| 레벨 | 방식 | 트리거 | 용도 |
|------|------|--------|------|
| **자동 Stagger** | CSS-only (`nth-child` 딜레이) | 슬라이드 전환 시 자동 | 모든 슬라이드 기본 적용 — 자식 요소가 순차적으로 빠르게 등장 |
| **수동 빌드 스텝** | JS 상태 (`stepIndex`) + Framer Motion | 키보드/클릭으로 수동 진행 | 발표자가 하나씩 클릭해서 보여주는 고급 시나리오 |

---

### 자동 Stagger 애니메이션 (CSS-only, 기본 적용)

슬라이드 전환 시 `.slide-content`의 **직계 자식 요소들이 100ms 간격으로 순차 등장**합니다.
JS 없이 CSS만으로 동작하며, 모든 슬라이드에 자동 적용됩니다.

#### CSS 구현

```css
/* 초기 상태: 모든 자식 요소 숨김 + 살짝 아래 배치 */
.slide-content > * {
  opacity: 0;
  transform: translateY(18px);
  transition:
    opacity 400ms ease-out,
    transform 400ms ease-out;
}

/* 활성 슬라이드: 자식 요소 등장 */
.slide--active .slide-content > * {
  opacity: 1;
  transform: translateY(0);
}

/* 자식 요소별 순차 딜레이 (nth-child 기반, 100ms 간격) */
.slide--active .slide-content > *:nth-child(1) { transition-delay: 100ms; }
.slide--active .slide-content > *:nth-child(2) { transition-delay: 200ms; }
.slide--active .slide-content > *:nth-child(3) { transition-delay: 300ms; }
.slide--active .slide-content > *:nth-child(4) { transition-delay: 400ms; }
.slide--active .slide-content > *:nth-child(5) { transition-delay: 500ms; }
.slide--active .slide-content > *:nth-child(6) { transition-delay: 600ms; }
.slide--active .slide-content > *:nth-child(7) { transition-delay: 700ms; }
.slide--active .slide-content > *:nth-child(8) { transition-delay: 800ms; }

/* 비활성 슬라이드: 딜레이 없이 즉시 숨김 (빠른 퇴장) */
.slide:not(.slide--active) .slide-content > * {
  transition-delay: 0ms;
  transition-duration: 200ms;
}
```

#### 동작 원리

```
슬라이드 전환 시 타임라인:

  0ms     슬라이드 fade-in 시작 (opacity 0→1, 500ms)
  100ms   ┌ 자식 1: label        ── opacity 0→1, translateY 18→0 (400ms)
  200ms   ├ 자식 2: title        ── opacity 0→1, translateY 18→0 (400ms)
  300ms   ├ 자식 3: divider      ── opacity 0→1, translateY 18→0 (400ms)
  400ms   ├ 자식 4: card/flow    ── opacity 0→1, translateY 18→0 (400ms)
  500ms   └ 자식 5: text-muted   ── opacity 0→1, translateY 18→0 (400ms)
  ~900ms  모든 애니메이션 완료
```

#### 이벤트 잠금 (자식 수에 따른 동적 계산)

```javascript
// 마지막 자식의 stagger 완료 시점까지 입력 차단
var childCount = slides[targetIndex].querySelector('.slide-content').children.length;
var unlockDelay = Math.max(520, childCount * 100 + 400 + 50);
setTimeout(function () { isAnimating = false; }, unlockDelay);
```

#### 설계 포인트

- **등장**: `100ms` 간격 stagger + `400ms` duration + `translateY(18px)` → 빠르고 경쾌한 느낌
- **퇴장**: 딜레이 0, `200ms` duration → 빠르게 사라져서 다음 슬라이드에 시선 집중
- **compositor-only**: `opacity` + `transform`만 애니메이션하여 GPU 가속, 60fps 보장
- **자식 8개까지 커버**: 슬라이드 내 직계 자식이 8개 이상일 경우 `nth-child` 규칙 추가 필요

---

### 수동 빌드 스텝 (JS 기반, 고급 시나리오)

발표자가 키보드를 눌러 하나씩 콘텐츠를 드러내는 방식입니다.
자동 stagger와 별개로, 특정 슬라이드에만 선택적으로 적용합니다.

#### 2차원 상태 모델

```typescript
interface DeckState {
  slideIndex: number;
  stepIndex: number;       // 현재 슬라이드 내 빌드 스텝
  maxSteps: number[];      // 각 슬라이드별 최대 스텝 수
  mode: 'slideshow' | 'presenter';
}
```

**핵심 동작**: "앞으로" → 스텝 소진 후 다음 슬라이드. "뒤로" → 이전 스텝 or 이전 슬라이드 마지막 스텝.

```typescript
case 'NEXT': {
  const currentMax = state.maxSteps[state.slideIndex] ?? 0;
  if (state.stepIndex < currentMax)
    return { ...state, stepIndex: state.stepIndex + 1 };
  if (state.slideIndex < state.maxSteps.length - 1)
    return { ...state, slideIndex: state.slideIndex + 1, stepIndex: 0 };
  return state;
}
```

### 핵심패턴 (수동 빌드 스텝)

#### BuildStep 컴포넌트

```tsx
'use client';
import { useContext } from 'react';
import { motion, type Variants } from 'framer-motion';
import { DeckContext } from '@/context/DeckContext';

const variants: Record<string, Variants> = {
  fade:      { hidden: { opacity: 0 },           visible: { opacity: 1, transition: { duration: 0.4 } } },
  'fade-up': { hidden: { opacity: 0, y: 30 },    visible: { opacity: 1, y: 0, transition: { duration: 0.5 } } },
  'fade-left': { hidden: { opacity: 0, x: -40 }, visible: { opacity: 1, x: 0, transition: { duration: 0.5 } } },
};

interface BuildStepProps { step: number; animation?: 'fade' | 'fade-up' | 'fade-left'; children: React.ReactNode; }

export function BuildStep({ step, animation = 'fade', children }: BuildStepProps) {
  const { state } = useContext(DeckContext);
  const isVisible = state.stepIndex >= step;

  return (
    <motion.div data-step={step} variants={variants[animation]}
                initial="hidden" animate={isVisible ? 'visible' : 'hidden'} aria-hidden={!isVisible}>
      {children}
    </motion.div>
  );
}
```

#### 순차 등장 리스트 (staggerChildren)

```tsx
const containerVariants = {
  hidden: {},
  visible: { transition: { staggerChildren: 0.12, delayChildren: 0.1 } },
};
const itemVariants = {
  hidden: { opacity: 0, x: -20 },
  visible: { opacity: 1, x: 0, transition: { duration: 0.4 } },
};

export function StaggerList({ step, items }: { step: number; items: string[] }) {
  const { state } = useContext(DeckContext);
  return (
    <motion.ul variants={containerVariants} initial="hidden" animate={state.stepIndex >= step ? 'visible' : 'hidden'}>
      {items.map((item, i) => <motion.li key={i} variants={itemVariants}>{item}</motion.li>)}
    </motion.ul>
  );
}
```

#### 코드 블록 줄 하이라이트 (Shiki)

```tsx
'use client';
import { useEffect, useState, useContext } from 'react';
import { codeToHtml } from 'shiki';
import { transformerNotationHighlight } from '@shikijs/transformers';
import { DeckContext } from '@/context/DeckContext';

interface CodeBlockProps {
  code: string;
  lang?: string;
  highlightLines?: Record<number, number[]>;  // step → 강조할 줄 번호 배열
}

export function CodeBlock({ code, lang = 'typescript', highlightLines }: CodeBlockProps) {
  const { state } = useContext(DeckContext);
  const [html, setHtml] = useState('');
  const activeLines = highlightLines?.[state.stepIndex] ?? [];

  useEffect(() => {
    codeToHtml(code, {
      lang, theme: 'github-dark',
      transformers: [transformerNotationHighlight()],
      decorations: activeLines.map((line) => ({
        start: { line: line - 1, character: 0 },
        end: { line: line - 1, character: Infinity },
        properties: { class: 'highlighted-line' },
      })),
    }).then(setHtml);
  }, [code, lang, activeLines]);

  return <div className="code-block" dangerouslySetInnerHTML={{ __html: html }} />;
}
```

```css
.highlighted-line {
  background-color: rgba(78, 204, 163, 0.10);       /* 민트 그린 하이라이트 */
  border-left: 3px solid var(--accent);              /* 민트 그린 보더 */
  display: inline-block;
  width: 100%;
  transition: background-color 0.3s ease;
}
```

사용 예시:

```tsx
<CodeBlock code={sampleCode} highlightLines={{ 1: [1], 2: [2, 3], 3: [4] }} />
```

#### 화자 노트 (Speaker Notes)

```tsx
export function SpeakerNotes({ children }: { children: React.ReactNode }) {
  const { state } = useContext(DeckContext);
  if (state.mode !== 'presenter') return null;

  return (
    <aside className="speaker-notes" role="note" aria-label="Speaker notes">
      <h3>Notes</h3>
      <div>{children}</div>
    </aside>
  );
}
```

```css
.speaker-notes {
  position: fixed;
  bottom: 0; left: 0; right: 0;
  height: 200px;
  background: #0d0d0d;
  color: var(--text-secondary);
  padding: 1rem 2rem;
  overflow-y: auto;
  border-top: 1px solid var(--border-card);
  z-index: 50;
  font-family: var(--font-body);
}
```

#### 탭 간 동기화 (localStorage)

```typescript
// 발표자 창 → 슬라이드쇼 창 동기화
useEffect(() => {
  localStorage.setItem('deck-sync', JSON.stringify({ slideIndex, stepIndex, ts: Date.now() }));
}, [slideIndex, stepIndex]);

useEffect(() => {
  const handler = (e: StorageEvent) => {
    if (e.key === 'deck-sync' && e.newValue) {
      const { slideIndex: s, stepIndex: st } = JSON.parse(e.newValue);
      onSync(s, st);
    }
  };
  window.addEventListener('storage', handler);
  return () => window.removeEventListener('storage', handler);
}, [onSync]);
```

#### 슬라이드 사용 예시

```tsx
<Slide>
  <h1>주요 변경사항</h1>
  <BuildStep step={1} animation="fade-up"><p>성능 30% 향상</p></BuildStep>
  <BuildStep step={2} animation="fade-up"><p>번들 크기 50% 감소</p></BuildStep>
  <BuildStep step={3}>
    <CodeBlock code={sampleCode} highlightLines={{ 3: [2, 3], 4: [5] }} />
  </BuildStep>
  <SpeakerNotes>
    성능 개선 수치를 먼저 언급하고, 코드 예시로 구체적인 변경점을 설명합니다.
  </SpeakerNotes>
</Slide>
```

### 주의사항

#### 성능

| 항목 | 권장사항 |
|------|----------|
| Framer Motion | ~30KB gzipped. 단순 애니메이션이면 CSS-only 방식 고려 |
| Shiki | WASM 기반, 초기 로드 무거움 → RSC 사전 렌더링 또는 dynamic import |
| 슬라이드 마운트 | fixed 방식은 모든 슬라이드가 DOM에 존재. 슬라이드 수 30장 이하에 적합 |
| Vercel 배포 | `output: 'export'`로 Static Export 가능. Edge Runtime 불필요 |

#### 접근성

- 숨겨진 BuildStep은 `aria-hidden="true"` 필수
- `visibility: hidden` 또는 `aria-hidden` 사용 (`display: none`은 스크린 리더에서 완전 제거)
- `prefers-reduced-motion` 미디어 쿼리 반드시 존중
- `@media print`에서 모든 빌드 스텝을 visible 상태로 표시

#### 코드 하이라이트 라이브러리 비교

| 라이브러리 | 장점 | 단점 |
|-----------|------|------|
| **Shiki** (추천) | VS Code 동일 문법, 빌드 시 렌더링 가능 (JS 0KB), 줄 강조 transformer 내장 | WASM 초기 로드 비용 |
| react-syntax-highlighter | 설정 간단, 생태계 성숙 | 런타임 JS 필요, 테마 커스텀 제한적 |
| react-shiki | React 전용 컴포넌트/훅 | 비교적 신규 패키지 |

---

## 5. 적용된 수정사항 요약

| # | 항목 | 변경 내용 |
|---|------|----------|
| 1 | **뷰포트 풀스크린** | 위아래 여백 없이 브라우저 화면 꽉 채움. 스크롤 스냅 → JS `fixed+absolute` 전환 방식 |
| 2 | **다크/민트 그린 프리미엄 컬러** | 액센트: 민트 그린(`#4ECCA3`). 메인 텍스트: 순수 화이트(`#FFFFFF`), 보조 텍스트: 라이트 그레이(`#B0B0B0`) — 샘플 스크린샷에서 추출한 실제 색상 |
| 3 | **콘텐츠 폭 축소** | `max-width: 1000px` → `680px`. 내부 컴포넌트도 비율에 맞게 축소 |
| 4 | **한글 폰트 최적화** | Google Fonts에서 Noto Sans KR (wght 400;700) 추가. 한글 본문은 Noto Sans KR, 영문/코드는 Inter + JetBrains Mono 유지. `font-family: 'Noto Sans KR', 'Inter', sans-serif` |

---

## 참고 자료

- [next-mdx-deck (GitHub)](https://github.com/whoisryosuke/next-mdx-deck)
- [slide-presentation — Next.js 15 + Tailwind CSS (GitHub)](https://github.com/theoklitosBam7/slide-presentation)
- [reveal.js Fragments](https://revealjs.com/fragments/)
- [Shiki Transformers (@shikijs/transformers)](https://shiki.style/packages/transformers)
- [Shiki Decorations API](https://shiki.matsu.io/guide/decorations)
- [web.dev — The large, small, and dynamic viewport units](https://web.dev/blog/viewport-units)
- [CSS aspect-ratio (Can I Use)](https://caniuse.com/mdn-css_properties_aspect-ratio)
- [MDN — CSS Scroll Snap](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll_snap)
- [Framer Motion stagger](https://www.framer.com/motion/stagger/)
- [Google Fonts — Noto Sans KR](https://fonts.google.com/noto/specimen/Noto+Sans+KR)