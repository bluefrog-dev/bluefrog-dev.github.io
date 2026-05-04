# Bluefrog 기업 랜딩 페이지

리워드 기반 제품으로 소비자와 비즈니스를 잇는 회사 **블루프로그(Bluefrog)** 의 공식 기업 사이트.

- **운영 도메인:** https://bluefrog.co.kr
- **호스팅:** GitHub Pages (`bluefrog-dev/bluefrog-dev.github.io`)
- **유형:** 정적 단일 페이지 (SPA-like, 라우팅 없음)

---

## 기술 스택

| 영역 | 사용 기술 |
|---|---|
| 마크업 | HTML5 단일 파일 (`index.html`) |
| 스타일 | Tailwind CSS 3.4 (정적 빌드) |
| 인터랙션 | Vanilla JS (프레임워크 없음) |
| 폰트 | Pretendard (서브셋), Nanum Pen Script (서브셋) — 자체 호스팅 |
| 이미지 | WebP + JPG 폴백 (`<picture>`) |
| 자동화 | GitHub Actions (Tailwind 자동 빌드) |
| 배포 | GitHub Pages (main 브랜치 자동 배포) |

---

## 디렉터리 구조

```
.
├── index.html              # 메인 페이지 (전체 콘텐츠)
├── styles.css              # 빌드된 Tailwind CSS (자동 생성)
├── src/
│   └── input.css           # Tailwind 입력 소스
├── tailwind.config.js
├── fonts/                  # 자체 호스팅 woff2 서브셋
│   ├── pretendard-subset.woff2   (133KB)
│   └── nanum-pen-subset.woff2    (5.4KB)
├── images/                 # WebP + JPG 페어
├── og-image.png            # OG 카드 (52KB, 1200x630)
├── llms.txt                # AI/LLM 크롤러용 구조화 정보
├── robots.txt              # AI 크롤러 명시적 허용 포함
├── sitemap.xml             # 4개 언어 hreflang 포함
├── CNAME                   # bluefrog.co.kr
└── .github/workflows/
    └── build-css.yml       # Tailwind 자동 빌드 액션
```

---

## 실행 / 빌드

```bash
pnpm install              # 또는 npm ci
pnpm run watch            # 개발 모드 (Tailwind --watch)
pnpm run build            # 프로덕션 빌드 (--minify)
```

> CSS는 main 브랜치 푸시 시 GitHub Actions가 자동으로 다시 빌드해 커밋합니다 (`build-css.yml`). 로컬에서 직접 빌드해도 무방.

---

## 최적화 포인트 기록

이 사이트는 **마케팅 페이지 = 빠르고, 검색·LLM에 잘 잡히고, 다국어 사용자에게 일관된 경험을 주는 곳**이라는 원칙에 따라 다음을 적용했습니다.

### 1. 다국어 (4개 언어 동시 지원)

- **지원 언어:** 한국어(기본), English, 日本語, Tiếng Việt
- **렌더링 방식:** 모든 텍스트 노드에 `data-ko / data-ja / data-vi / data-en` 속성으로 4개 번역 동시 보유 → Vanilla JS가 즉시 스왑 (페이지 리로드 없음)
- **언어 진입 경로 우선순위:**
  1. URL query param (`?lang=ja` 등) — Google 다국어 검색 결과에서 직접 진입할 때
  2. `localStorage.bf_lang` — 재방문 시 이전 선택 유지
  3. 기본값 `ko`
- **메타 동기화:** 언어 전환 시 `<title>`, `description`, `og:*`, `twitter:*`, `og:locale`, `<html lang>` 모두 갱신
- **표기 통일:** 마케팅 카피에서 "가맹점/제휴처" → **"제휴점"** 통일, 일본어 `加盟店` → `提携店`, 영어 `merchant` → `partner store`로 톤 일치

### 2. SEO

- **`<title>`, `<meta description>`, `keywords`, `robots`** — 4개 언어 모두 JS로 동기화
- **Canonical:** `https://bluefrog.co.kr/` 단일 정규 URL
- **hreflang:** ko / en / ja / vi / x-default 5종 명시 (HTML + sitemap.xml 양쪽)
- **JSON-LD 구조화 데이터:**
  - `Organization` — 회사 정보, 본사·서울·인천 3개 주소, 전화/이메일/언어/서비스 지역
  - `WebSite` — 사이트 정보
  - `WebApplication` × 2 — sosok, trepick 제품 정의
- **OpenGraph / Twitter Card:** 1200×630 OG 이미지, locale alternate 4종(ko/en/ja/vi)
- **sitemap.xml:** xhtml hreflang alternate 포함, 네임스페이스 정확
- **robots.txt:** 검색엔진 + 14종 AI 크롤러(GPTBot, ClaudeBot, PerplexityBot 등) 명시적 Allow
- **네이버 사이트 등록:** `naver-site-verification` 메타 적용

### 3. 성능 (Lighthouse 최적화)

#### 폰트
- **Pretendard 자체 호스팅 + 페이지 서브셋** — 사이트에 실제 사용된 글리프만 포함한 단일 woff2 파일(133KB)로 통합 (이전: CDN 92청크 분할 로딩)
- **Nanum Pen 자체 호스팅 서브셋** (5.4KB)
- **`rel="preload" as="font" fetchpriority="high"`** — 본문 폰트는 최우선 프리로드
- **`font-display: swap`** — FOIT 방지
- 모바일 FCP 회귀 방지 위해 폰트 CSS 우선순위 조정

#### 이미지
- **WebP + JPG `<picture>` 폴백** — 평균 60% 이상 절감 (예: press-kyeongin 215KB → 23KB)
- **`width`/`height` 명시** — CLS 0 보장
- **`loading="lazy"`, `decoding="async"`** — 첫 화면 외 모든 이미지에 적용
- **로고만 `fetchpriority="high"`, `decoding="sync"`** — LCP 후보 우선 처리

#### CSS / JS
- **Tailwind CSS 정적 빌드 + minify** (24KB)
- **CSS render-blocking 최소화** — 중복 preload 제거
- **인라인 SVG 아이콘** — 외부 아이콘 폰트/스프라이트 요청 0
- **스크롤 핸들러 `requestAnimationFrame` + early-return** — 강제 리플로우 제거, passive listener
- **CountUp 애니메이션은 IntersectionObserver 기반** — 화면 진입 시에만 실행
- **`paint-defer` 클래스로 reveal 애니메이션 단축** — 인터랙션 지연 방지

### 4. 접근성 (a11y)

- **시맨틱 HTML** — `<section>`, `<nav>`, `<header>`, `<footer>`, `<main>` 구조화
- **색 대비 보강** — 본문/보조 텍스트 모두 WCAG AA 이상
- **`aria-label`, `aria-hidden`, `aria-expanded`, `aria-selected`, `role="option"`** — 언어 토글 등 인터랙티브 요소에 적용
- **키보드 접근:** 언어 메뉴 ESC 닫힘, 선택 후 토글 버튼 포커스 복귀
- **`<a href="#main">` 등 앵커 네비게이션** — 스크린리더용 섹션 점프
- **`alt` 속성** — 모든 콘텐츠 이미지에 한국어 설명 제공

### 5. AI / LLM 친화

- **`llms.txt`** — Bluefrog 회사·제품·연락처·언론 정보를 LLM이 즉시 활용 가능한 구조화 마크다운으로 제공
- **JSON-LD `WebApplication` × 2** — sosok / trepick을 별도 엔티티로 노출
- **`robots.txt` AI 크롤러 명시 허용** — GPTBot, ClaudeBot, PerplexityBot, OAI-SearchBot, Google-Extended, ChatGPT-User 등 14종 화이트리스트
- **본문 카피의 LLM 친화성** — 제품 정의·지표·언론 보도가 자연어로 페이지에 직접 노출 (DOM-only 콘텐츠, 외부 fetch 없음)

### 6. 호환성 / 안정성

- **Single page, no router** — 라우팅 라이브러리 의존성 0
- **No external CDN at runtime** — 모든 폰트·아이콘·이미지 자체 호스팅 (네트워크 페일오버 위험 0)
- **모바일·데스크탑·태블릿 반응형** — Tailwind breakpoint 활용
- **카카오톡 OG 미리보기 호환** — og:image PNG 유지 (KakaoTalk WebP 미지원)

---

## 자동화 (GitHub Actions)

`.github/workflows/build-css.yml`:

- **트리거:** main 브랜치 push, 그리고 다음 파일 변경 시에만 실행
  - `index.html`, `src/input.css`, `tailwind.config.js`, `package.json`, `package-lock.json`
- **동작:**
  1. Node 20 + npm ci
  2. `npm run build` — Tailwind 다시 컴파일
  3. `styles.css` 변경되면 자동 커밋·푸시 (`[skip ci]`로 무한 루프 방지)
- **README, 이미지, sitemap.xml 등 푸시는 액션 트리거 안 함** → 비용·시간 절약

---

## 배포

- **방식:** GitHub Pages가 main 브랜치를 자동 배포 (1~2분)
- **도메인:** `CNAME` 파일로 `bluefrog.co.kr` 매핑, HTTPS 자동 (Let's Encrypt)
- **DNS:** A 레코드 → GitHub Pages IP

---

## 향후 검토 가능한 개선

- [ ] og:image 다국어 분리 (현재 한국어 1종만)
- [ ] Service Worker로 오프라인 캐싱
- [ ] `<noscript>` 폴백 콘텐츠 (현재 JS 비활성 시 언어 토글 작동 안 함)
- [ ] PageSpeed Insights 정기 측정 자동화

---

## 라이선스 / 문의

© Bluefrog Inc. — 본 저장소는 회사 공식 사이트 코드입니다.
문의: contact@bluefrog.co.kr
