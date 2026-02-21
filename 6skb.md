---
name: sixshop-pro
description: 식스샵 프로 웹빌더 개발 지식베이스. 커스텀 블록 개발 시 블록메이커 코드(template, style, script, json 설정패널)를 작성할 때 사용.
---


# [Block Maker AI 완전 지식 베이스]

## 1. 블록 메이커 핵심 개념

> 블록(Block)은 웹사이트 페이지를 구성하는 디자인 단위입니다.
  • 하나의 블록은 <data>, <template>, <style>, <script> 태그로 구성됩니다.
  • 각 블록은 독립적인 컨텍스트(Context)를 가지며, 사용자가 설정한 값들이 property 객체에 저장됩니다.
  • 블록은 CSR(Client-Side Rendering) 방식으로 렌더링되며, 각 배치 위치마다 별도의 설정을 가질 수 있습니다.

---

## 2. 블록 구조 - 4가지 핵심 태그

> <data> 태그 (선택사항)
  • 블록 외부 데이터를 가져올 때 사용합니다.
  • 지원 값: $page, $customer, $cart, $legal, $social
  - <data value="$page" /> <data value="$customer" />

> <template> 태그
  • HTML 템플릿을 정의하며, Handlebars.js 문법을 지원합니다.
  • 블록 컨테이너 내부에 렌더링됩니다.
  - <template> <h1>{{property.title}}</h1> <p>{{property.description}}</p> </template>

> <style> 태그
  • CSS 스타일을 정의하며, Handlebars.js 문법을 지원합니다.
  • 블록 컨테이너 선택자로 자동 격리되어 다른 블록에 영향을 주지 않습니다.
  - <style> h1 { color: {{property.textColor}}; font-size: {{property.fontSize}}px; } </style>

> <script> 태그
  • JavaScript 코드를 작성하여 블록의 동작을 제어합니다.
  • bm 객체를 통해 블록 기능에 접근합니다.
  • 자동으로 IIFE로 래핑되어 격리됩니다.
  - <script> const container = bm.container; const context = bm.context; container.addEventListener('click', e => { e.preventDefault(); if (e.target.matches('button')) { alert('버튼 클릭!'); } }, true); </script>

---

## 3. Handlebars.js 핵심 문법

> 기본 값 접근
  - {{property.myValue}} {{property.nested.value}}
  - HTML 이스케이프 방지
  - {{{property.htmlContent}}}

> 조건문
  - {{#if property.showContent}} <p>표시됩니다</p> {{else}} <p>숨겨집니다</p> {{/if}} {{#if (eq property.status "active")}} <p>활성 상태</p> {{/if}} {{#if (and property.flagA property.flagB)}} <p>둘 다 참</p> {{/if}}

> 반복문
  - {{#each property.items}} <div> {{#if @first}}<strong>첫 항목</strong>{{/if}} {{@index}}: {{name}} {{#if @last}}<strong>마지막</strong>{{/if}} </div> {{else}} <p>항목이 없습니다</p> {{/each}}

> 주요 헬퍼 함수
  • 조건: and, or, eq, gt, gte, lt, lte, isEmpty, isNil
  • 숫자: add, subtract, multiply, divide, ceil, floor, round
  • 문자열: toUpper, toLower, capitalize, trim, split, join
  • 배열: first, last, slice, size, filter, orderBy
  • 날짜: datetime
  • 변환: toJSON, fromJSON, toNumber, toString

---

## 4. bm 객체 - 블록 제어 API

> bm.container
  • 블록 컨테이너의 DOM 객체
  • 이벤트 바인딩 및 DOM 조작에 사용
  - const container = bm.container; container.addEventListener('click', handler, true);

> bm.cntext
  • 블록의 컨텍스트 객체 (property, data 등 포함)
  - const context = bm.context; console.log(context.property.myValue);

> bm.apply()
  • 컨텍스트 변경사항을 확정하고 블록을 재렌더링
  - context.counter = (context.counter || 0) + 1; bm.apply();

> bm.config(key[, value])
  • 블록 설정 값을 읽거나 변경
  - const page = bm.config('property:board.posts.page'); bm.config('property:board.posts.page', page + 1); bm.apply();

> bm.onContextChange
  • 컨텍스트 변경 시 실행되는 콜백
  - bm.onContextChange = () => { console.log('컨텍스트가 변경되었습니다'); };

> bm.onDestroy
  • 블록 제거 시 정리 작업 수행
  - const interval = setInterval(() => {...}, 1000); bm.onDestroy = () => { clearInterval(interval); };

> bm.do(action[, data])
  • 시스템 액션 실행 (로그아웃, 게시글 작성 등)
  - bm.do('logout'); await bm.do('board:createPost', { params: { boardId: 1, title: '제목', contents: '내용', accessType: 'OPEN' } });

> bm.call(request)
  • HTTP 요청 (axios 스타일)
  - bm.call({ method: 'GET', url: 'https://api.example.com/data' }).then(response => { bm.context.data = response.data; bm.apply(); });

> bm.localStorage / bm.sessionStorage
  • 블록 스코프 스토리지 (자동 JSON 변환)
  - bm.localStorage.setItem('key', {name: 'value'}); const data = bm.localStorage.getItem('key'); bm.localStorage.removeItem('key');

---

## 5. 블록 세팅(Setting) 시스템
- 블록 세팅은 사용자가 에디터에서 블록을 커스터마이징할 수 있게 해주는 설정 스키마입니다.

> 기본 타입
  • TEXT: 한 줄 텍스트
  • TEXTAREA: 여러 줄 텍스트
  • RICH_TEXT: HTML 포맷 텍스트 (CKEditor)
  • CHECKBOX: 참/거짓 선택
  • RADIO: 여러 옵션 중 하나 선택 (최대 5개)
  • SELECT: 드롭다운 선택 (최대 100개)
  • RANGE: 슬라이더 숫자 입력
  • COLOR_PICKER: 색상 선택 (Hex + Alpha)
  • COLOR_SCHEME: 색상 조합 선택 (CSS 변수 제공)
  • IMAGE_PICKER: 이미지 업로드
  • VIDEO_PICKER: 비디오 업로드

> 구조화된 타입
  • LIST: 반복 가능한 항목 목록 (최대 100개)
  • LINK: 내부/외부 링크 설정
  • MENU: 3단계 메뉴 구조 (id 필드 생략, property.menus로 접근)
  • PRODUCT: 상품 선택 (id는 "products" 고정)
  • BOARD: 게시판 선택 (id는 "board" 고정)
  • REVIEW: 리뷰 선택 (id는 "reviews" 고정)

> UI 구성 타입
  • TITLE: 섹션 제목
  • DESCRIPTION: 설명 텍스트
  • TAB: 탭 그룹 (최대 2개 권장)

> 조건부 표시 (isVisible)
  - { "id": "customColor", "type": "COLOR_PICKER", "isVisible": "property.useCustomColor === true" }

> Setting ID 규칙
  • a-z, A-Z, 0-9, _, . 만 사용 가능 (하이픈, 공백 불가)
  • 점(.)으로 중첩 구조 표현: a.b.c → {a: {b: {c: ...}}}
  • 예약어 금지: class, case, default, for 등
  • 타입별 필수 ID: products(PRODUCT), board(BOARD), reviews(REVIEW), colorSchemeId(COLOR_SCHEME)

---

## 6. CSS 스타일링 규칙

> CSS 변수 사용 원칙
  • 사용자 설정값은 Handlebars.js로 직접 삽입 (CSS 변수 사용 금지)
    /* 올바른 방법 */ .container { padding: {{property.padding}}px; font-size: {{property.fontSize}}px; } /* 잘못된 방법 */ .container { padding: var(--padding); font-size: var(--font-size); }

> 시스템 제공 CSS 변수 (예외)
  • 타이포그래피 변수 (항상 사용 가능)
    var(--font-family-heading) var(--font-weight-heading) var(--font-family-body) var(--font-weight-body)
  • 색상 조합 변수 (COLOR_SCHEME 사용 시)
    var(--color-text-100) ~ var(--color-text-3) var(--color-background-100) ~ var(--color-background-0) var(--color-accent-100) ~ var(--color-accent-5) var(--color-border-100)

> 필수 타이포그래피 적용
  /* 헤딩 요소 */ h1, h2, h3 { font-family: var(--font-family-heading); font-weight: var(--font-weight-heading); } /* 본문 요소 */ p, li, span { font-family: var(--font-family-body); font-weight: var(--font-weight-body); }

> 전폭 컨테이너 안전 영역
  • 내부 콘텐츠 보호를 위한 좌우 패딩 필수
  • 배경 이미지는 CSS background-image 사용 (img 태그 금지)
  - .full-width-container { padding-left: {{property.paddingHorizontal}}px; padding-right: {{property.paddingHorizontal}}px; background-image: url('{{property.bgImage}}'); }

---

## 7. 반응형 디자인 가이드

> 브레이크포인트 표준
  • 모바일: < 768px
  • 태블릿: 768px ~ 1024px
  • 데스크톱: > 1024px

> 반응형 설정 구조
  • 데스크톱/모바일 별도 설정 필수
    { "id": "fontSize", "label": "폰트 크기 (데스크톱)", "type": "RANGE", "min": 16, "max": 48, "step": 2, "unit": "px" }, { "id": "fontSizeMobile", "label": "폰트 크기 (모바일)", "type": "RANGE", "min": 14, "max": 32, "step": 2, "unit": "px" }

> 모바일 최적화
  • 다단 레이아웃 → 단일 컬럼 스택
  • 애니메이션 복잡도 감소
  • 호버 인터랙션 → 터치 친화적 변환
  • 폰트 크기 적절히 스케일링
  @media (max-width: 768px) { .grid { grid-template-columns: 1fr; } .font-size { font-size: {{property.fontSizeMobile}}px; } }

---

## 8. 성능 및 보안 규칙

> 절대 금지 사항
  • eval() 사용 (보안 위험)
  • 직접 네트워크 요청: fetch(), XMLHttpRequest, navigator.sendBeacon()
  • 전역 객체 직접 접근: document, window, localStorage
  • 인라인 스크립트 주입: {{{property.unsafeHTML}}}
  • HTTP 프로토콜 사용 (HTTPS만 허용)

> 권장 사항
  • 이미지 lazy loading 사용
    <img loading="lazy" src="..." />
  • IntersectionObserver 활용 (스크롤 이벤트 대신)
  • CSS transform/opacity로 애니메이션 최적화
  • 리소스 정리 필수 (bm.onDestroy)
    const observer = new IntersectionObserver(...); bm.onDestroy = () => { observer.disconnect(); };

> 이벤트 바인딩 패턴
  • 상위 컨테이너에 이벤트 위임
    /* 올바른 방법 */ bm.container.addEventListener('click', e => { if (e.target.matches('button')) { // 처리 } }, true); /* 잘못된 방법 */ const button = bm.container.querySelector('button'); button.addEventListener('click', handler);

---

## 9. 데이터 타입 상세 스키마

> $page (페이지 정보)
  - { "name": "HOME", // HOME, CUSTOM, PRODUCT_LIST, PRODUCT_DETAIL, CART, CHECKOUT, ORDER_COMPLETED, BOARD, POST, LOGIN, SIGNUP 등 "path": "/" // URL 경로 }

> $customer (고객 정보)
  - { "id": "...", "name": { "full": "김블록" } }

> $cart (장바구니)
  - { "count": 1, "total": { "items": { "price": { "original": 40000, "sale": 32000 }, "discounted": 8000 } } }

> LINK 타입
  - { "id": null, "type": "url", "label": "https://example.org", "value": "https://example.org" }

> MENU 타입
  - [ { "id": "1", "name": "메뉴명", "link": { /* LINK 객체 */ }, "shouldOpenInNewTab": false, "childMenus": [ /* 재귀 구조 */ ] } ]

> PRODUCT 타입
  - { "data": [ { "id": "...", "product": {"id": "..."}, "availability": "in-stock", "slug": "...", "name": "상품명", "summary": "요약", "labels": [{"id": "...", "name": "인기"}], "images": [{"url": "..."}], "price": {"original": 50000, "sale": 42500}, "discounts": [...], "review": { "publicReviewAverageRating": 4.5, "publicReviewCount": 10 } } ] }

> BOARD 타입
  - { "id": 1, "name": "게시판명", "commentStatus": "ENABLED", "posts": { "data": [ { "id": 1, "title": "제목", "contents": "<p>내용</p>", "displayName": "김**", "isNotice": false, "status": "PUBLIC", "accessType": "OPEN", "authorType": "ADMIN", "commentsCount": 3, "thumbnails": [...], "createdDate": "2023-11-02T02:24:16.388791" } ], "count": 35 } }

> REVIEW 타입
  - { "data": [ { "id": 1, "channel": "WEBSITE", "authorName": "김**", "productName": "상품명", "rating": 5, "content": "리뷰 내용", "images": [{"url": "...", "type": "IMAGE"}], "commentsCount": 3, "displayDate": "2025-01-30T10:34:01.351826" } ], "count": 1 }

---

## 10. 프리뷰 지원 및 라이프사이클

> 자동 프리뷰 반영
  • <template>, <style>의 property 값 변경은 자동 반영

> 스크립트 프리뷰 지원
  • bm.onContextChange로 property 변경 감지 및 재처리
    const container = bm.container; const context = bm.context; // 초기 렌더링 function render() { const p = container.querySelector('p'); p.innerHTML = context.property.content; } render(); // property 변경 시 재렌더링 bm.onContextChange = () => { render(); };

> 라이프사이클 순서
  1. <template>, <style> 렌더링
  2. <script> 실행
  3. property 변경 시:
    → <template>, <style> 재렌더링
    → bm.onContextChange 실행
  4. 블록 제거 시:
    → bm.onDestroy 실행

---

## 11. 디자인 시스템 - 테마

> Fashion-Minimal
  • 색상: 모노크롬 + 골드 액센트
  • 타이포그래피: 굵기 대비 + 자간 강조
  • 형태: 날카로운 모서리 (0-4px), 플랫
  • 키워드: Luxury, Minimal, Artistic, Fashion, Soft-Gallery
  • 색상: 따뜻한 중성색 (#B08968, #F5F5F0)
  • 타이포그래피: 명확한 계층, 중간 굵기 대비
  • 형태: 곡선 강조 (8-24px), 부드러운 그림자
  • 키워드: Gallery, Warm, Friendly, Curved

> Dynamic-Premium
  • 색상: 고대비 흑백 + 유연한 비비드 액센트
  • 타이포그래피: 극단적 굵기 대비 (900 vs 300)
  • 형태: 강한 기하학 + 동적 애니메이션
  • 키워드: Premium, Dynamic, High-Impact, Brutalist

> Cyber-Glass
  • 색상: 다크 그라디언트 + 네온 (#00FF88, #FF0080)
  • 타이포그래피: 극단적 굵기 대비
  • 형태: 글라스모피즘 + 사이버 요소
  • 효과: 텍스트 글로우, blur(10px)
  • 키워드: Cyber, Future, Tech, Glass, Neon

> Liquid-3D
  • 색상: 비비드 그라디언트 (보라/핑크/블루)
  • 타이포그래피: 부드럽고 중간 굵기 대비
  • 형태: 유기적 형태 + 3D 변환 + 다층 blur
  • 애니메이션: 3D 변환 20초 무한 반복
  • 키워드: Liquid, 3D, Fluid, Organic, Premium

> Elegant-Sparkle
  • 색상: 럭셔리 그라디언트 (딥 퍼플/로즈 골드)
  • 타이포그래피: 초경량 굵기 (200-300)
  • 형태: 스파클 애니메이션 + 글로우 효과
  • 키워드: Elegant, Luxury, Sparkle, Jewelry

---

## 12. 디자인 제약사항

> 절대 금지
  • 동시 그라디언트 효과 최대 2개
  • 폰트 패밀리 최대 3개
  • 동시 사용 색상 최대 5개
  • 애니메이션 지속시간 1초 초과 금지
  • 복잡한 전역 스크롤 이벤트 바인딩 (IntersectionObserver 사용)

> 레이아웃 갭 요구사항
  • 카드 레이아웃: 최소 16px 갭 (0px 금지)
  • 데이터 그리드: 0px 포함 모든 갭 허용

> 금지된 테마-패턴 조합
  • Fashion-Minimal + Fluid-Glass-Card
  • Elegant-Sparkle + 텍스트 중심 스크롤 패턴

---

## 13. 샘플 에셋 생성 규칙

> URL 구조
  - https://ss3-prod-static-files.s3.ap-northeast-2.amazonaws.com/block-image-library/{{category}}/image{{number}}.jpg

> 이미지 카테고리
  • beauty-care (1-37): beauty, makeup, skincare
  • fashion-basic (1-15): men's fashion, simple, casual
  • fashion-classic (1-23): women's fashion, elegant
  • fashion-colorful (1-16): colorful, vivid, swimming
  • fashion-retro (1-18): retro, vintage, natural
  • fashion-urban (1-24): modern, urban
  • food-dessert (1-21): dessert, cake, bakery
  • food-fresh (1-24): salad, fruit, vegetables
  • health (1-25): yoga, wellness
  • interior-furniture (1-50): furniture, mid-century
  • jewellery (1-41): jewelry, accessory
  • kids (1-22): kids products
  • lifestyle (1-53): kitchen, home, daily life
  • pets (1-18): dog, cat
  • sports (1-9): exercise, tennis
  • video (1-5): video, movie

> 동적 이미지 번호 선택
  - image_number = (현재_초 % (max - min + 1)) + min

> 필수 속성
  - <img loading="lazy" onerror="this.src='https://ss3-prod-static-files.s3.ap-northeast-2.amazonaws.com/block-image-library/lifestyle/image1.jpg'" />

> 비디오 사용
  • VIDEO_PICKER + IMAGE_PICKER (썸네일) 세트 제공
  • video{{N}}.mp4 + video{{N}}-thumbnail.jpg

---

## 14. QA 규칙 요약

> R001: 핵심 태그 구조
  • <template>, <style>, <script> 각 1개씩만
  • 속성 금지 (단, <data value="..."> 예외)

> R002: Property 사용 일치
  • 사용된 property는 settings에 정의 필수
  • 정의된 property는 코드에서 참조 필수

> R005: eval() 금지
  • 보안 위험으로 절대 사용 금지

> R006: DOM 바인딩
  • 이벤트는 상위 컨테이너에 위임
  • 재렌더링 고려한 바인딩

> R007: 전역 환경 접근 금지
  • document, window, localStorage 직접 사용 금지
  • bm.container, bm.localStorage 사용

> R008: Context 변경 추적
  • property 읽는 로직은 bm.onContextChange에서 재실행

> R009: 리소스 정리
  • setInterval, addEventListener 등은 bm.onDestroy에서 정리

> R012: XSS 위험
  • {{{...}}}는 신뢰된 콘텐츠만 사용

> R013: HTTPS 필수
  • HTTP 링크 금지, HTTPS만 허용

> R014: 네이밍 일관성
  • camelCase 권장, 명확한 의미 전달

> R017: 뷰 렌더링
  • 가능하면 Handlebars로 렌더링, JavaScript는 필요시만

> R018: 네트워크 요청 금지
  • fetch, XMLHttpRequest 금지
  • bm.call()만 사용

---

## 15. 실전 패턴 라이브러리

> Hero-Three-Column
  • 3단 레이아웃 (30%-40%-30%)
  • 중앙 포커스 포인트
  • 용도: 패션 랜딩, 제품 런칭

> Scroll-Sticky-Text
  • 고정 텍스트 + 스크롤 이미지
  • IntersectionObserver 기반
  • 용도: 포트폴리오, 스토리텔링

> Arch-Slider
  • 아치형 프레임 (border-radius: 50% 50% 0 0)
  • 자동 슬라이드 + 네비게이션
  • 용도: 제품 갤러리, 이미지 컬렉션

> Gallery-Grid
  • 반응형 그리드 (auto-fit minmax(350px, 1fr))
  • 호버 오버레이
  • 용도: 포트폴리오, 제품 갤러리

> Product-Showcase
  • 설명(1fr) + 제품(2fr) 그리드
  • 호버 효과 + 빠른 보기
  • 용도: 이커머스 디스플레이

> Location-Map-Block
  • 카카오맵 HTML 이미지 추출
  • TEXTAREA 설정으로 HTML 입력
  • 네이버맵 지원 불가
  • 용도: 회사 위치, 매장 안내

> Typing-Animation-Text
  • 타자기 효과 애니메이션
  • 동적 라인 생성 (제한 없음)
  • CSS 깜빡이는 커서
  • 용도: 히어로 헤드라인, 브랜드 슬로건

---

## 16. 고급 기법 및 팁

> 동적 카운터 구현
  <template> <p>카운터: {{counter}}</p> <button>증가</button> </template> <script> const context = bm.context; const container = bm.container; container.addEventListener('click', e => { e.preventDefault(); if (e.target.matches('button')) { context.counter = (context.counter || 0) + 1; bm.apply(); } }, true); </script>

> 페이지네이션 구현
  <template> {{#each property.board.posts.data}} <div>{{title}}</div> {{/each}} <button>다음 페이지</button> </template> <script> bm.container.addEventListener('click', e => { e.preventDefault(); if (e.target.matches('button')) { const page = bm.config('property:board.posts.page'); bm.config('property:board.posts.page', page + 1); bm.apply(); } }, true); </script>

> 조건부 렌더링 최적화
  {{#if (and property.showSection (gt property.items.length 0))}} <section> {{#each property.items}} <div>{{name}}</div> {{/each}} </section> {{/if}}

> IntersectionObserver 활용
  const observer = new IntersectionObserver((entries) => { entries.forEach(entry => { if (entry.isIntersecting) { entry.target.classList.add('visible'); } }); }, { threshold: 0.1 }); const elements = bm.container.querySelectorAll('.animate'); elements.forEach(el => observer.observe(el)); bm.onDestroy = () => { observer.disconnect(); };

> 외부 API 호출
  bm.call({ method: 'GET', url: 'https://api.example.com/products' }).then(response => { bm.context.products = response.data; bm.apply(); }).catch(error => { console.error('API 호출 실패:', error); });

---

## 17. 디버깅 및 문제 해결

> 일반적인 오류
  1. Property 값이 반영되지 않음
    → bm.apply() 호출 확인
    → bm.onContextChange 구현 확인

  2. 이벤트가 작동하지 않음
    → 이벤트 위임 패턴 사용 확인
    → e.preventDefault() 호출 확인

  3. 스타일이 적용되지 않음
    → CSS 선택자 특이성 확인
    → Handlebars 문법 오류 확인

  4. 메모리 누수
    → bm.onDestroy에서 정리 확인
    → 타이머, 옵저버 해제 확인

> 디버깅 팁
  // Context 값 확인 console.log('Context:', bm.context); // Property 값 확인 console.log('Property:', bm.context.property); // 변경 감지 확인 bm.onContextChange = () => { console.log('Context changed:', bm.context); }; // 이벤트 확인 bm.container.addEventListener('click', e => { console.log('Clicked:', e.target); }, true);

---

## 18. 베스트 프랙티스

> 코드 구조
  • 간단하고 유지보수 가능하게 작성
  • 주석으로 주요 로직 설명
  • HTML/CSS 우선, JavaScript는 필요시만

> 성능 최적화
  • 이미지 lazy loading 활용
  • CSS transform/opacity로 애니메이션
  • IntersectionObserver로 스크롤 감지
  • 불필요한 재렌더링 최소화

> 접근성
  • 시맨틱 HTML 사용
  • alt 텍스트 제공
  • 키보드 네비게이션 지원
  • 적절한 색상 대비

> 반응형 디자인
  • 모바일 우선 접근
  • 터치 친화적 인터랙션
  • 적절한 폰트 스케일링
  • 유연한 레이아웃

> 보안
  • 사용자 입력 검증
  • XSS 방지 ({{...}} 사용)
  • HTTPS 링크만 사용
  • bm.call()로만 HTTP 요청

---

## 19. 체크리스트

> 블록 생성 전
  ☐ 요구사항 명확히 이해
  ☐ 적절한 테마 선택
  ☐ 필요한 설정 항목 정의
  ☐ 샘플 데이터 준비

> 코드 작성 중
  ☐ 태그 구조 올바른지 확인
  ☐ Property 정의와 사용 일치
  ☐ CSS 변수 올바르게 사용
  ☐ 이벤트 위임 패턴 적용
  ☐ 반응형 설정 구현

> 코드 완성 후
  ☐ QA 규칙 준수 확인
  ☐ 프리뷰 정상 작동 확인
  ☐ 리소스 정리 구현 확인
  ☐ 모바일 반응형 테스트
  ☐ 접근성 검토
  ☐ 성능 최적화 확인

> 제출 전
  ☐ 주석 추가
  ☐ 불필요한 코드 제거
  ☐ 설정 설명 명확한지 확인
  ☐ 기본값 적절한지 확인

---

## 20. 핵심 요약

> 블록 = 4개 태그
  • <data> (선택): 외부 데이터 주입
  • <template>: HTML + Handlebars
  • <style>: CSS + Handlebars
  • <script>: JavaScript + bm API

> 핵심 객체
  • bm.container: DOM 컨테이너
  • bm.context: 블록 데이터
  • bm.apply(): 변경 적용
  • bm.onContextChange: 변경 감지
  • bm.onDestroy: 정리 작업

> 설정 시스템
  • 15개 기본 타입 + 5개 구조화 타입
  • ID 규칙: a-z, A-Z, 0-9, _, .
  • 타입별 예약 ID 준수
  • 조건부 표시 지원

> 스타일링
  • 사용자 설정 = Handlebars 직접 삽입
  • 시스템 변수 = CSS 변수 사용
  • 타이포그래피 변수 필수 적용
  • 전폭 컨테이너 안전 영역 필수

> 금지 사항
  • eval()
  • 직접 네트워크 요청
  • 전역 객체 직접 접근
  • HTTP 프로토콜
  • 인라인 스크립트 주입

> 권장 사항
  • 이벤트 위임 패턴
  • IntersectionObserver 활용
  • 리소스 정리 구현
  • 반응형 설정 분리
  • 간단하고 유지보수 가능한 코드

---

[사고 프로세스]

# FILE: 00_README.md

# Sixshop Pro AI Copilot Pack (RAG + SOP + Prompts + Lints + Eval)
  이 패키지는 “다른 AI”가 식스샵 프로/블록메이커에서
  - 가능/불가능 판단
  - 설계(설정 패널/IA/반응형)
  - 개발(블록 코드)
  - QA 검증 및 자동 수정을 Sixshop Block Maker GPT 수준으로 수행하도록 만드는 운영체계입니다.

## 핵심 구성
- 01_SOP.md : 운영 규격(SOP) + 트랙 분리(클라이언트/마켓)
- 02_ARCHITECTURE.md : 시스템 아키텍처(5단계 + 역할 분리 + 수정루프)
- 03_PROMPTS.md : 역할별 프롬프트 4종 + 수리(Repair) 프롬프트
- 04_SETTING_RULES.md : 설정 패널 절대 원칙(당신의 원칙을 규칙화)
- 05_LINT_RULES.md : 코드/설정 정적검사 규칙(자동 체크 목록)
- 06_RECIPES.md : 레시피 10종(패턴 문서 템플릿 + 예시)
- 07_EVAL_SCHEMA.md : 평가세트 스키마(노션/시트) + 점수 기준
- 08_OUTPUT_FORMAT.md : “항상 이 포맷으로 출력” 강제 템플릿
- 09_RAG_INGESTION.md : RAG 문서 구성/청킹/메타데이터 가이드

## 운영 원칙
- 생성보다 “검증→수정”이 더 중요합니다.
- “클라이언트 납품용”은 운영자 실수 방지 최우선입니다.
- “마켓 납품용”은 범용성과 확장성 최우선입니다.
- 두 트랙은 QA 기준이 다릅니다(문서에 명시).



# FILE: 01_SOP.md

# SOP: 의뢰 처리 표준 운영 절차

## 트랙 분리
### 1) 클라이언트 납품용 (client_delivery)
목표: 운영자(비개발자)가 안정적으로 운용, 실수 방지 최우선
- 기본 노출: 콘텐츠 탭만
- 디자인/개발자 탭: developerMode를 켜야만 옵션 노출
- Desktop/Mobile 분리: 텍스트/정렬/레이아웃/패딩은 원칙적으로 분리
- description: “무엇/변화/권장/복구” 4문장 강제
- 고급 효과/애니메이션: 보수적으로

### 2) 마켓플레이스 납품용 (marketplace)
목표: 범용 적용(테마/다양한 사이트), 재사용성/확장성 최우선
- 옵션 범위는 넓게 제공 가능하나 developerMode 보호는 유지
- 기본값은 보수적(깨짐 방지)
- 문서화/레시피 준수(유지보수성)

---

## 5단계 고정 파이프라인
1) 입력(고객 의뢰) 수집
2) 가능/불가능 판정 + 대안 설계
3) 세팅/IA 설계 + 기본값/권장값 확정
4) 블록 코드 생성
5) QA 린트 통과 → 최종 산출

---

## 역할 분리(에이전트)
- Judge: 가능/불가능 판정관
- Architect: 설계자(IA/세팅/반응형/운영자 UX)
- Builder: 구현자(코드 작성)
- Reviewer: QA 리뷰어(룰 기반)
- Repair: 수정자(Reviewer 지적을 바탕으로 최소 수정)

---

## 수정 루프(중요)
- Reviewer가 Fail을 내면 Repair가 수정
- 최대 N회 반복 후에도 Fail이면 “부분완성 + 불가 사유 + 대안”으로 종료

권장 N:
- client_delivery: N=2 (보수적, 빠른 종료)
- marketplace: N=3 (범용성 때문에 수정 기회 1회 추가)

---

## 최종 산출물 구성
- 가능/불가능 판정 결과 + 근거
- 설계 요약(IA/반응형/운영자 UX)
- 설정 스키마 표(type/id/label/description)
- settings + property JSON
- 블록 코드(template/style/script)
- QA 체크 결과(통과/수정내역)



# FILE: 02_ARCHITECTURE.md

# 목표 아키텍처(한 장 요약)

## 파이프라인(고정)
입력(의뢰) → 판정/설계 → 코드 생성 → QA 검증 → 최종 산출

## 지식베이스(3층)
1) 정책/규칙: 블록 구조, 렌더링, 라이프사이클, 금지 패턴, 보안/성능 규칙
2) 레시피: 자주 쓰는 UI/설정/반응형/효과 패턴(조립 가능한 단위)
3) 사례: 실제 의뢰→정답 판정→대안→설계/코드/주의점(학습·평가의 핵심)

## 에이전트 구조
- Judge: “가능/불가능/부분가능” + 대안
- Architect: 설정 패널(탭/그룹/설명) + Desktop/Mobile + 기본값(최적값)
- Builder: 코드 생성(템플릿/스타일 우선, JS 최소화)
- Reviewer: 린트/룰 기반 Fail 판정 + 수정 지시
- Repair: Fail 지시를 반영하여 최소 수정

## 시스템 설계 포인트
- 생성 모델은 바뀌어도 됨
- 품질은 “정답 사례 + 린트 + 수정 루프”가 유지함
- 특히 ‘불가 판정 정확도’와 ‘QA 치명 위반률 0’이 핵심 KPI



# FILE: 03_PROMPTS.md

# 역할별 프롬프트(복붙용)

## 공통 시스템 규칙(모든 역할 공통)
- 클라이언트 납품용/마켓 납품용을 반드시 구분한다.
- 답변은 반드시 08_OUTPUT_FORMAT.md 포맷을 따른다.
- 설정 패널 절대 원칙(탭 3개, 그룹핑, D/M 분리, 개발자 모드, description 템플릿)을 위반하면 실패로 간주한다.
- 코드 금지 패턴을 사용하면 실패로 간주한다(04,05 문서 참조).
- 불확실하면 “추정”이라 말하고, 안전한 대안을 제시한다.

---

## 1) JUDGE 프롬프트
당신은 식스샵 프로 웹빌더/블록메이커의 제약을 고려해 “가능/불가능/부분가능”을 판단하는 전문가다.
입력된 요구사항을 다음 순서로 처리하라:

1) 트랙(track): client_delivery 또는 marketplace를 먼저 선언
2) 판정: possible / partial / impossible 중 하나
3) 근거: 제약/리스크 1~5개
4) 대안: 불가/부분가능이면 현실적 대안을 1~3개 제시
5) 구현 전략 힌트: 필요한 레시피 ID(06_RECIPES.md 참조) 1~5개

주의:
- 가능 판정을 남발하지 말고, 애매하면 partial로 처리
- 운영자 UX 리스크(설정 복잡도, 실수 가능성)도 제약으로 포함

---

## 2) ARCHITECT 프롬프트
당신은 세팅/IA/반응형/운영자 UX 설계자다.
반드시 04_SETTING_RULES.md 규칙을 준수하여 settings/property를 설계하라.

필수:
- TAB 3개: 콘텐츠/디자인/개발자
- 그룹핑 TITLE/DESCRIPTION: 콘텐츠 → 레이아웃 → 디자인 → 효과&특수
- 콘텐츠 탭은 운영자용만(제목/본문/이미지/버튼/링크/리스트)
- 디자인/개발자 탭의 모든 옵션은 developerMode가 true일 때만 노출
- 모바일 UX가 달라지는 항목은 Desktop/Mobile 2트랙
- 줄바꿈 필요 텍스트는 TEXTAREA + template에서 pre-line 처리

출력:
- setting table(type/id/label/description)
- settings + property JSON(기본값 포함)
- 권장값(최적값)을 property 기본값으로 반영

---

## 3) BUILDER 프롬프트
당신은 블록 코드를 작성하는 구현자다.
원칙:
- 가능한 한 템플릿/스타일(Handlebars)로 렌더링하고 JS는 최소화
- 스크립트에서 property 기반 UI를 바꾼다면 반드시 onContextChange로 동기화
- DOM 이벤트는 컨테이너 단에서 위임(리렌더로 DOM 교체됨)
- 외부 네트워크 호출은 금지(필요하면 제공된 안전 호출만 사용)

출력:
- <style>, <template>, <script> 세 태그만 사용(각 1개)
- settings/property에서 정의한 값만 참조(불일치 금지)

---

## 4) REVIEWER 프롬프트
당신은 QA 리뷰어다. 05_LINT_RULES.md의 규칙을 체크한다.
출력은 다음 형식으로만 작성:
- verdict: PASS or FAIL
- failedRules: [규칙ID...]
- issues: 사람이 이해할 수 있는 수정 지시(최소 수정 지향)
- riskLevel: critical/major/minor

주의:
- client_delivery 트랙은 설정 노출/설명/복구값 누락도 FAIL로 처리
- marketplace 트랙은 범용성/기본값 보수성에 대한 경고를 추가할 수 있음

---

## 5) REPAIR 프롬프트
당신은 수정자다. REVIEWER의 issues만 반영하여 “최소 수정”으로 PASS를 만들라.
수정 원칙:
- 기능 추가/변경 금지(요구사항 바꾸지 말 것)
- 규칙 위반만 고치기
- settings/property/코드 간 불일치를 최우선 해결
- 수정 전/후 diff를 간단히 요약

출력:
- 수정된 settings JSON(필요 시)
- 수정된 코드(필요 시)
- diff 요약

---

# FILE: 04_SETTING_RULES.md

## 설정 패널 설계 원칙
> 목표: 운영자(비개발자)가 **안전하게** 다루면서도, 제작자가 **확장/디버깅** 할 수 있는 세팅 패널을 일관된 규칙으로 설계한다.  
> 범위: 헤더 한정이 아닌 **웹사이트/쇼핑몰 전체 공통 블록**(섹션/배너/상품카드/푸터/팝업 등)


## 0) 탭 구조 (고정)

  ### ✅ 탭 3개 고정
  - `콘텐츠(Content)` : 운영자(비개발자)용. **안전하고 자주 바꾸는 것만** 노출.
  - `디자인(Design)` : 시각 스타일(레이아웃·타이포·색·간격·카드·버튼·미디어·효과 등), 기본은 숨김.
  - `개발자(Developer)` : 확장/호환/성능/디버그/문제 해결, 기본은 숨김.
  > 운영자가 실수로 조작하는 것을 방지하기 위해 '디자인과 개발자'탭에 속한 설정들은 '개발자 옵션보기'항목에 체크했을 때만 하위 옵션들을 보여준다.

  ### ✅ 고급 옵션 노출 방식 (표준화)
  아래 둘 중 하나를 팀 표준으로 고정한다.

    #### 권장안 A (추천): “고급 옵션 보기” 1개 토글
    - `개발자` 하나로 디자인/개발자 고급 옵션을 일괄 노출

    #### 권장안 B: 탭별 토글 2개
    - 디자인 탭: `디자인`
    - 개발자 탭: `개발자`

  > 어떤 방식을 쓰든 **토글 UI/문구/경고 톤을 통일**한다.

---

## 1) 라벨/설명 규칙 (운영자 친화)

### label 규칙 (필수)
- 기능이 한눈에 이해되는 **명사형**으로 작성  
  예) `좌우 여백(모바일)`, `버튼 텍스트`, `배경색`

### description 규칙 (타입별 최소 강제)
**모든 setting 공통(필수 2문장)**
- "이 값은 OO을(를) 변경합니다."
- "문제가 생기면 XX로 되돌리세요."

**운영자 영향이 큰 옵션(권장 1문장 추가)**
- "권장값: XX."

**TEXTAREA에만(필수)**
- "엔터로 줄바꿈이 가능해요."

> 설명을 “모든 항목에 4문장 강제”하면 UI가 과밀해지고 유지비가 커질 수 있으므로, 위처럼 **필수/권장/타입 전용**으로 분리한다.

---

## 2) Desktop/Mobile 2트랙 분리 규칙

### ✅ 분리 “권장/강제” 기준
**분리 강제(추천)**
- 줄바꿈/가독성이 중요한 텍스트(히어로/배너 카피 등)
- 레이아웃이 확실히 달라지는 옵션(열 수, 정렬, 미디어 위치)
- 간격/패딩(모바일 타이트)

**분리 비권장**
- 브랜드 컬러/폰트 패밀리 같은 전역 성격
- 메뉴명 등 짧은 라벨(관리 부담이 큼)

### ✅ 네이밍 규칙 (안전한 방식)
점(`.`) 경로는 시스템에 따라 경로 파싱이 불안정할 수 있어 아래 중 하나로 고정한다.

- 안전형: `titleDesktop`, `titleMobile`
- 객체형(지원 시): `title: { desktop, mobile }`

---

## 3) 줄바꿈(엔터) 처리 규칙

### ✅ 텍스트 줄바꿈이 필요한 경우
- 입력: `TEXTAREA` 사용
- 출력: 템플릿에 `white-space: pre-line;` 적용

### 제목(title)은 예외 허용
- 기본은 `TEXT`로 두고(줄바꿈 불가), “길면 자동 줄바꿈”으로 처리
- 정말 줄바꿈이 필요하면:
  - `titleAllowLineBreak` 토글 + 조건부로 `TEXTAREA` 노출(가능한 경우)

---

## 4) 그룹핑 규칙(IA 고정)
설정 패널은 아래로 고정한다(TITLE, DESCRIPTION으로 강제):
1) 콘텐츠(Content)
2) 디자인(Design)
3) 개발자(Developer)

### 4-1) “섹션의 모든 시각 요소”를 제어하려면 꼭 넣어야 하는 그룹들
> 비개발자 친화 + 자유도 높게 만들려면, 아래 그룹은 사실상 필수
1) 레이아웃
   - 최대 폭, 컬럼 수, 정렬, 미디어 위치
2) 타이포
   - 제목/본문 크기 단계(S/M/L), 강조 방식(굵기 정도 선택)
3) 색/배경
   - 스킴 선택 + 표면 톤(100/200) + 포인트 사용 여부
4) 간격
   - 섹션 paddingY, 요소 gap
5) 카드/컨테이너
   - 라운드, 보더, 그림자(토글), 내부 padding
6) 버튼
   - 스타일(채움/테두리), 크기(S/M), 라운드, 아이콘 표시 여부
7) 미디어
   - 비율, 크롭 방식(object-fit cover 고정), 오버레이
8) 효과(선택)
   - 아주 제한적으로(토글 + 강도 하나)

## 4) IA(정보 구조) 고정: “기능 단위 그룹”

### ✅ 콘텐츠 탭 그룹(추천)
1) 텍스트(제목/본문/라벨)
2) 미디어(이미지/아이콘/배경)
3) 링크/CTA(버튼/링크)
4) 데이터(리스트/카드 항목)

### ✅ 디자인 탭 그룹(고급 옵션 켰을 때 노출)
1) 레이아웃(폭/정렬/컬럼)
2) 타이포(크기 단계/굵기/자간)
3) 색/배경(스킴/표면톤/포인트)
4) 간격(패딩Y/갭)
5) 카드/컨테이너(라운드/보더/그림자)
6) 버튼(스타일/크기/라운드/아이콘)
7) 미디어(비율/object-fit/오버레이)
8) 효과(토글 + 강도)

---

## 5) 개발자 모드(Developer) — 무엇을 넣을까?

개발자 탭은 “운영자가 몰라도 되는 **확장 포인트 + 문제 해결 도구 + 안전 장치**”만 모은다.

### A) 확장 포인트 (가장 유용)
- `customClass` : 블록 루트에 추가 클래스 (캠페인/페이지별 스타일 분기)
- `customId` : 루트 ID/앵커 (스크롤 이동, 타겟팅)
- `dataAttr`(선택) : data-* 속성(추적/테스트)  
  - **주의:** 너무 자유롭게 열면 위험하므로 “키 제한” 또는 “JSON 한 줄”처럼 통제된 입력을 권장

### B) 호환성/성능 토글 (운영 사고 감소)
- `disableBackdropFilter` : 블러 비활성화
- `disableAnimation` : 모션 비활성화
- `reduceShadow` : 그림자 약화/비활성화
- `safeMode`(선택) : 파서/렌더링 안전 폴백 모드(내부적으로 안전한 기본값 강제)

### C) 충돌 해결(현장 최다 이슈)
- `zIndexOverride`
- `containerMaxWidthOverride`
- `breakpointOverride`(예: 768)  
  - **주의:** 전역과 충돌할 수 있으므로 “이 블록에만 적용”되게 설계

### D) 디버그/진단 (프리뷰/개발에서만)
- `debugOverlay` : 그리드/패딩/컨테이너 영역 표시
- `debugLog` : 콘솔 로그 출력
- `showPropertyPreview` : property 일부 화면 표시

### E) 기능 플래그 (점진 배포/실험)
- `featureFlags`(간단 토글 몇 개로 시작)  
  예) `enableNewLayout`, `enableNewTypography`

---

## 6) 개발자 모드에 “넣지 말아야 할 것”
- 자유 JS 코드 입력(보안/검수/운영 리스크)
- 정의되지 않은 외부 URL 호출(임의 API 호출)
- SEO/결제/회원 등 **전역 영향이 큰 기능**을 블록 개발자 탭에서 직접 제어

---

## 7) 크래시/저장 실패를 막는 “강제 안전 규칙” 2개

### ✅ (1) 값 포맷 통일 규칙
- `COLOR_PICKER`는 **HEX만** 사용: `#RRGGBB` 또는 `#RRGGBBAA`
- `LINK`는 빌더 표준 포맷 **1가지로만** 통일  
  - 문자열형(`/path`) 또는 객체형(빌더 스펙) 중 하나를 팀 표준으로 고정  
  - 템플릿도 그 표준에 맞춰서만 접근

### ✅ (2) 안전한 기본값 규칙
- `contents/design/developer` 기본값은 **false**
- 고급 옵션이 꺼져도 블록은 **완전 정상 동작**해야 한다  
  (필수 기능이 고급 옵션 뒤에 숨으면 안 됨)

---

## 8) 세팅 타입별 권장 사용표

| 목적 | 권장 타입 | 비고 |
|---|---|---|
| 섹션 구분 제목 | TITLE | `content` 필수(없으면 저장 에러) |
| 긴 안내 문구 | DESCRIPTION | 운영자 주의/가이드 |
| 단문 텍스트 | TEXT | 줄바꿈 필요하면 TEXTAREA |
| 여러 줄 텍스트 | TEXTAREA | `pre-line` 처리 |
| 온/오프 | CHECKBOX | 고급 옵션 토글에 사용 |
| 숫자(px 등) | NUMBER | min/max/step 가능하면 지정 |
| 색상 | COLOR_PICKER | HEX만(권장) |
| 이미지 | IMAGE_PICKER | 기본값 null 허용 |
| 반복 항목 | LIST | 하위 필드는 `settings`로 정의(키 통일) |
| 링크 | LINK | 포맷 팀 표준 고정(문자열/객체 중 택1) |

---

## 9) “복붙용” 표준 설정 템플릿 (3탭 + 고급 토글)

> 아래는 **구조 템플릿**이다.  
> `TAB`/`condition` 지원 여부는 빌더 구현에 따라 다를 수 있으니, 팀 표준에 맞게 조정한다.  
> (조건 기능이 없다면: 고급 옵션을 개발자 탭으로만 보내고, 디자인 탭은 최소 옵션만 노출)

```json
{
  "settings": [
    {
      "type": "TAB",
      "content": "콘텐츠",
      "settings": [
        { "type": "TITLE", "content": "콘텐츠" },
        {
          "id": "advancedMode",
          "type": "CHECKBOX",
          "label": "고급 옵션 보기",
          "description": "이 값은 고급 옵션 노출을 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": false
        },

        { "type": "TITLE", "content": "텍스트" },
        {
          "id": "title",
          "type": "TEXT",
          "label": "제목",
          "description": "이 값은 섹션 제목을 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "placeholder": "예: 이번 주 인기 상품"
        },
        {
          "id": "body",
          "type": "TEXTAREA",
          "label": "설명",
          "description": "이 값은 섹션 설명을 변경합니다.\n권장: 2~3줄로 간결하게 작성하세요.\n문제가 생기면 기본값으로 되돌리세요.\n엔터로 줄바꿈이 가능해요.",
          "placeholder": "설명 문구를 입력하세요."
        },

        { "type": "TITLE", "content": "미디어" },
        {
          "id": "image",
          "type": "IMAGE_PICKER",
          "label": "대표 이미지",
          "description": "이 값은 대표 이미지를 변경합니다.\n문제가 생기면 기본값으로 되돌리세요."
        },

        { "type": "TITLE", "content": "CTA" },
        {
          "id": "ctaText",
          "type": "TEXT",
          "label": "버튼 텍스트",
          "description": "이 값은 버튼 문구를 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "placeholder": "예: 자세히 보기"
        },
        {
          "id": "ctaLink",
          "type": "LINK",
          "label": "버튼 링크",
          "description": "이 값은 버튼 이동 링크를 변경합니다.\n문제가 생기면 기본값으로 되돌리세요."
        }
      ]
    },

    {
      "type": "TAB",
      "content": "디자인",
      "settings": [
        { "type": "TITLE", "content": "디자인" },
        {
          "id": "preset",
          "type": "SELECT",
          "label": "프리셋",
          "description": "이 값은 섹션 스타일 프리셋을 변경합니다.\n권장: 기본 프리셋을 사용하세요.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": "default",
          "options": [
            { "label": "기본", "value": "default" },
            { "label": "강조", "value": "emphasis" },
            { "label": "미니멀", "value": "minimal" }
          ]
        },

        { "type": "TITLE", "content": "색/배경 (고급)" },
        {
          "id": "bgColor",
          "type": "COLOR_PICKER",
          "label": "배경색",
          "description": "이 값은 배경색을 변경합니다.\n권장: 투명도가 필요하면 #RRGGBBAA를 사용하세요.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": "#FFFFFFF2"
          /* condition 사용 가능 시:
          ,"condition": { "variable": "advancedMode", "value": true }
          */
        },

        { "type": "TITLE", "content": "간격 (고급)" },
        {
          "id": "paddingY",
          "type": "NUMBER",
          "label": "상하 여백(px)",
          "description": "이 값은 섹션 상하 여백을 변경합니다.\n권장: 24~64 사이를 사용하세요.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": 48
          /* condition: advancedMode */
        }
      ]
    },

    {
      "type": "TAB",
      "content": "개발자",
      "settings": [
        { "type": "TITLE", "content": "개발자 설정" },
        {
          "id": "developerMode",
          "type": "CHECKBOX",
          "label": "개발자 모드",
          "description": "이 값은 개발자 옵션 노출을 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": false
        },

        { "type": "TITLE", "content": "확장 포인트" },
        {
          "id": "customClass",
          "type": "TEXT",
          "label": "커스텀 클래스",
          "description": "이 값은 루트 클래스 추가를 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "placeholder": "e.g. theme-dark compact"
          /* condition: developerMode */
        },
        {
          "id": "customId",
          "type": "TEXT",
          "label": "커스텀 ID",
          "description": "이 값은 루트 ID/앵커를 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "placeholder": "e.g. section-hero"
          /* condition: developerMode */
        },

        { "type": "TITLE", "content": "충돌 해결" },
        {
          "id": "zIndexOverride",
          "type": "NUMBER",
          "label": "z-index 오버라이드",
          "description": "이 값은 레이어 순서를 변경합니다.\n권장: 문제가 있을 때만 조정하세요.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": 1000
          /* condition: developerMode */
        },

        { "type": "TITLE", "content": "성능/호환" },
        {
          "id": "disableAnimation",
          "type": "CHECKBOX",
          "label": "애니메이션 비활성화",
          "description": "이 값은 모션 효과를 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": false
          /* condition: developerMode */
        },
        {
          "id": "disableBackdropFilter",
          "type": "CHECKBOX",
          "label": "블러 비활성화",
          "description": "이 값은 블러 효과를 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": false
          /* condition: developerMode */
        },

        { "type": "TITLE", "content": "디버그" },
        {
          "id": "debugOverlay",
          "type": "CHECKBOX",
          "label": "디버그 오버레이",
          "description": "이 값은 디버그 표시를 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": false
          /* condition: developerMode */
        },
        {
          "id": "debugLog",
          "type": "CHECKBOX",
          "label": "콘솔 로그",
          "description": "이 값은 로그 출력을 변경합니다.\n문제가 생기면 기본값으로 되돌리세요.",
          "default": false
          /* condition: developerMode */
        }
      ]
    }
  ],
  "property": {
    "advancedMode": false,

    "title": "",
    "body": "",
    "image": null,
    "ctaText": "",
    "ctaLink": null,

    "preset": "default",
    "bgColor": "#FFFFFFF2",
    "paddingY": 48,

    "developerMode": false,
    "customClass": "",
    "customId": "",
    "zIndexOverride": 1000,
    "disableAnimation": false,
    "disableBackdropFilter": false,
    "debugOverlay": false,
    "debugLog": false
  }
}
```
---

## 10) 컬러 규칙(필수)
- 컬러는 RADIO로 ‘단색/그라데이션’ 선택
- 단색 선택 시: color picker 1개
- 그라데이션 선택 시: 시작색/끝색 color picker + 각도 range
- 디자인 탭에 배치

---

# FILE: 05_LINT_RULES.md

# QA 자동검증(린트) 규칙

## 목적
- 생성된 설정 JSON과 코드가 “실무에서 바로 납품 가능한 수준”인지 자동 판정한다.
- FAIL이면 REPAIR 루프로 최소 수정하여 PASS를 만든다.

---

## A. 구조 규칙(치명)
L001: 코드에 style/template/script 태그가 각각 정확히 1개 존재해야 한다.
L002: style/template/script 태그에 속성(attribute)을 붙이지 않는다.
L003: data 태그는 필요할 때만 사용한다. (불필요하면 금지)

---

## B. 보안/네트워크 규칙(치명)
L101: 외부 네트워크 호출 API 사용 금지(fetch, XMLHttpRequest, sendBeacon 등)
L102: eval() 금지
L103: 신뢰되지 않은 HTML을 triple-stash로 그대로 삽입하는 패턴은 금지(필요 시 제한적으로만 허용)

---

## C. 렌더/라이프사이클 규칙(치명~중대)
L201: 템플릿이 리렌더링될 때 DOM이 통째로 교체될 수 있으므로,
      특정 DOM에 직접 이벤트 바인딩을 지양하고 컨테이너에서 위임한다.
L202: setInterval/MutationObserver/이벤트리스너 등 “해제 필요 리소스”는 onDestroy에서 정리한다.
L203: 스크립트에서 property 기반 UI를 갱신하면 onContextChange로 재동기화한다.

---

## D. 설정/프로퍼티 일치 규칙(치명)
L301: 코드에서 참조하는 property 경로는 settings에 존재해야 한다.
L302: settings에 선언한 id는 코드에서 반드시 사용되거나, “정보용(TITLE/DESCRIPTION/TAB)”이어야 한다.
L303: Desktop/Mobile 분리 강제 항목이 분리되어 있지 않으면 client_delivery에서 FAIL.

---

## E. 설정 UX 규칙(클라이언트 트랙에서 치명)
L401: 탭 3개(콘텐츠/디자인/개발자) 미구성 시 FAIL
L402: 그룹핑 IA(TITLE/DESCRIPTION) 미준수 시 FAIL
L403: 모든 setting에 description 템플릿 4문장 미포함 시 FAIL
L404: 디자인/개발자 탭 옵션이 developerMode 없이 노출되면 FAIL
L405: 줄바꿈 필요한 텍스트에 TEXT 사용 시 FAIL(권장: TEXTAREA)

---

## F. 스타일/반응형 규칙(중대)
L501: 텍스트/레이아웃/패딩은 Desktop/Mobile이 다르면 반드시 2트랙으로 분리한다.
L502: template에서 TEXTAREA 줄바꿈을 pre-line으로 처리한다.

---

## G. 트랙별 가중치
- client_delivery: L401~L405는 치명(FAIL)
- marketplace: L401~L405는 기본적으로 PASS 목표이나 일부는 경고로 완화 가능(단 developerMode 보호는 유지 권장)




# FILE: 06_RECIPES.md

# Recipes (레시피 문서 템플릿 10종)
각 레시피는 “설정 설계 + 코드 패턴 + 주의점”을 가진 재사용 단위입니다.

---

## R01. 탭/그룹핑 기본 골격
- 목적: 탭 3개 + 그룹 TITLE/DESCRIPTION 고정 + developerMode 보호
- 설정: developerMode (default false) + 디자인/개발자 옵션 isVisible 조건
- 주의: client_delivery는 콘텐츠 탭만 기본 노출

---

## R02. 텍스트 Desktop/Mobile 2트랙 (TEXTAREA + pre-line)
- 목적: 제목/본문/버튼 텍스트가 모바일에서 가독성이 달라지는 문제 해결
- 설정: content.*.desktop + content.*.mobile (TEXTAREA)
- 코드: white-space: pre-line;

---

## R03. 레이아웃(정렬/배치/최대폭) 2트랙
- 목적: 모바일 1열/중앙정렬 등 UX 변화 대응
- 설정: layout.align.desktop/mobile, layout.maxWidth, layout.stackOnMobile 등

---

## R04. 컬러(단색 vs 그라데이션)
- 설정:
  - design.bgType: RADIO(solid/gradient)
  - solid: design.bgSolid COLOR_PICKER
  - gradient: design.bgGradStart, design.bgGradEnd COLOR_PICKER + design.bgGradAngle RANGE
- 코드: linear-gradient(angle, start, end)

---

## R05. 카드 스타일(라운드/보더/그림자)
- 설정: design.radius, borderWidth, borderColor, shadowStrength
- 주의: 과한 그림자는 모바일에서 지저분해질 수 있음(권장값 명시)

---

## R06. 오버레이/블러
- 설정: effects.overlayOpacity, effects.blurPx
- 주의: blur는 성능 비용 큼 → 기본 0, 범위 제한

---

## R07. 애니메이션(transition + hoverScale)
- 설정: effects.transitionMs, effects.hoverScale
- 주의: hoverScale 과하면 레이아웃 튐 → 권장 1.02~1.05

---

## R08. Z-index 안전 설계
- 설정: advanced.zIndex
- 주의: 헤더/모달과 충돌 가능 → developerMode에서만

---

## R09. 버튼/링크(내부 링크 지원)
- 설정: LINK 타입 사용
- 코드: link.value 있을 때만 렌더

---

## R10. 스크립트 프리뷰 동기화 패턴
- 언제 필요한가: JS가 DOM 내용/상태를 직접 바꿀 때
- 규칙:
  - container에 이벤트 위임
  - property 사용 시 onContextChange에서 재적용
  - 해제 필요 리소스는 onDestroy에서 정리




# FILE: 07_EVAL_SCHEMA.md

# 평가(Eval) 세트 스키마 & 점수 기준

## 케이스 컬럼(노션/시트)
- caseId: TEXT
- track: SELECT(client_delivery / marketplace)
- requestText: TEXTAREA
- expectedVerdict: SELECT(possible / partial / impossible)
- expectedConstraints: TEXTAREA (1~3개)
- expectedRecipes: MULTI-SELECT (R01~R10)
- mustHaveSettings: TEXTAREA (developerMode, D/M 분리 항목 등)
- forbiddenPatterns: TEXTAREA (금지 패턴)
- goldOutput: TEXTAREA 또는 URL(정답 산출물)
- scoreVerdict: NUMBER(0/1)
- scoreSettingsUX: NUMBER(0~5)
- scoreQA: NUMBER(0~5)
- notes: TEXTAREA

## 점수 가이드
- Verdict(0/1): 판정이 맞으면 1, 틀리면 0
- SettingsUX(0~5):
  - 탭 3개 + 그룹핑 + description 4문장 + D/M 분리 + developerMode 보호
- QA(0~5):
  - 치명 위반 0이면 5, 있으면 0에 가까움

## KPI 추천
- client_delivery:
  - Verdict 0.85+
  - 치명 QA 위반 0%
  - SettingsUX 0.95+ 준수
- marketplace:
  - Verdict 0.80+ (애매하면 partial 허용)
  - 치명 QA 위반 0%
  - 범용성 경고를 잘 포함



# FILE: 08_OUTPUT_FORMAT.md

# 표준 출력 포맷(강제)
아래 순서를 절대 바꾸지 않는다.

1) 트랙 선언 + 가능/불가능 판정
- track:
- verdict: possible/partial/impossible
- reason:
- constraints:
- alternatives:

2) 설계(IA/반응형/운영자 UX)
- IA 그룹(콘텐츠/레이아웃/디자인/효과&특수)
- Desktop/Mobile 분리 목록
- developerMode 보호 정책
- 기본값(최적값) 요약

3) 설정 스키마 표
- type | id | label | description

4) settings + property JSON
- 복붙 가능한 완전한 JSON

5) 블록 코드
- <style>
- <template>
- <script>

6) QA 체크 결과
- PASS/FAIL
- failed rules (있으면)
- 수정 요약(있으면)



# FILE: 09_RAG_INGESTION.md

# RAG 지식베이스 구성/청킹 가이드

## 문서 레이어(3층)
1) policy: 변하지 않는 규칙(금지 패턴/구조/라이프사이클/설정 타입 규칙)
2) recipes: 패턴 문서(조립 가능한 단위)
3) cases: 실제 의뢰 사례(정답 포함)

## 청킹(Chunking) 권장
- policy: 섹션 단위로 300~800 토큰
- recipes: 레시피 1개 = 1문서 또는 1~2청크
- cases: 케이스 1개 = 1문서(요구/판정/근거/대안/설정/코드/QA)

## 메타데이터(검색 정확도 향상)
- layer: policy/recipe/case
- track: client_delivery/marketplace/both
- topic: settings/layout/design/effects/security/lifecycle
- recipeId: R01~R10 (recipe 문서에만)
- verdict: possible/partial/impossible (case 문서에만)

## 운영 팁
- 생성 모델이 틀리는 지점은 cases로 보강한다.
- “판정” 성능은 policy가 아니라 cases가 끌어올린다.
- “QA 합격률”은 lint rules + repair 루프가 끌어올린다.



# 10_BLOCK_ENGINE_CORE.md
# Sixshop Pro Block Engine Core Knowledge

이 문서는 식스샵 프로 블록메이커의 “엔진 구조”를 설명합니다.
AI가 정확히 이해해야 할 핵심 런타임 특성만 정리합니다.

---

# 1. 블록 기본 구조

블록은 다음 4가지 태그로 구성됩니다:

- <data> (선택)
- <template>
- <style>
- <script>

필수 규칙:
- <template> 1개
- <style> 1개
- <script> 1개
- 속성(attribute) 금지
- <data>는 필요할 때만 사용

---

# 2. 렌더링 구조 (중요)

## 2.1 CSR 기반

블록은 CSR(Client Side Rendering) 방식으로 동작합니다.

초기 렌더:
1) context 전달
2) template 컴파일
3) style prefix 처리
4) script 실행

---

## 2.2 리렌더링 동작

다음 상황에서 리렌더 발생:

- Block Settings 변경
- <data> 값 변경
- bm.apply() 호출

리렌더 시:

- <template> 전체 DOM 교체
- <style> 재적용
- <script>는 자동 재실행되지 않음

👉 핵심:
이전 DOM 참조는 모두 무효가 된다.

---

# 3. Block Inner Container

- 블록은 독립 컨테이너 안에서 렌더링됨
- style은 자동 prefix 되어 외부에 영향 없음
- 전역 CSS 오염 방지

---

# 4. Script 격리

- <script>는 IIFE로 감싸져 실행됨
- 전역 오염 방지
- Handlebars 글로벌 객체 접근 불가

---

# 5. Context 흐름

context 구조:

{
  id: "...",
  property: { ... },
  dataFromDataTags: { ... }
}

- template/style → Handlebars로 property 접근
- script → bm.context로 접근

---

# 6. Production Quality 핵심 이해

실무에서 가장 많이 터지는 문제:

1) 리렌더 후 이벤트 사라짐
2) DOM 직접 참조 유지 시 오류
3) property 변경 후 apply 누락
4) apply 내부에서 apply 호출 → 무한루프

이 엔진 구조를 이해하지 못하면
프로덕션 품질 개발은 불가능하다.




# 11_BM_OBJECT_COMPLETE.md
# bm 객체 완전 정리

bm은 블록 인스턴스 객체입니다.

---

# 1. bm.container

블록 내부 컨테이너 DOM

사용 목적:
- 이벤트 위임
- DOM 탐색 시작점

주의:
- 리렌더 시 내부 DOM은 교체됨
- container 자체는 유지됨

안전 패턴:

container.addEventListener('click', (e)=>{
  const btn = e.target.closest('.btn');
  if(btn){ ... }
});

---

# 2. bm.context

현재 블록 context

읽기:
bm.context.property.title

쓰기:
bm.context.customValue = 1;
bm.apply();

---

# 3. bm.apply()

- context 변경 사항 확정
- template/style 재렌더 트리거

절대 금지:
bm.onContextChange 내부에서 bm.apply()

---

# 4. bm.config()

블록 설정값 읽기/쓰기

bm.config('property:board.posts.page')
bm.config('property:board.posts.page', 2)
bm.apply()

---

# 5. bm.call()

HTTP 요청 전용 메서드

fetch 금지
XMLHttpRequest 금지

반드시 bm.call 사용

---

# 6. bm.localStorage / sessionStorage

- 블록 스코프 저장소
- 자동 JSON 직렬화

---

# 7. bm.onContextChange

context 확정 후 호출

사용 상황:
- script에서 property 기반 DOM 조작 필요할 때

---

# 8. bm.onDestroy

블록 제거 시 호출

필수 사용 상황:
- setInterval
- MutationObserver
- 외부 이벤트리스너




# 12_RUNTIME_BEHAVIOR.md
# 런타임 동작 이해

---

# 1. 리렌더 시 DOM 파괴

template 재컴파일 → 내부 HTML 전체 교체

결과:
- querySelector로 저장한 DOM 참조 무효
- 직접 바인딩 이벤트 사라짐

해결:
- container 위임 방식 사용

---

# 2. 상태 동기화 패턴

property 기반 UI 제어 시:

bm.onContextChange = ()=>{
  // property 기반 재적용 로직
}

---

# 3. 성능 경계

주의 요소:

- blur 과다 사용
- 과도한 hoverScale
- 무거운 JS 반복

모바일 성능을 항상 고려

---

# 4. Z-index 충돌

헤더/모달과 충돌 빈번

항상 developerMode에서 조정 가능하게 설계




# 13_SAFE_JS_ARCHITECTURE.md
# 안전한 JS 설계 패턴

---

# 1. 이벤트 위임 패턴 (필수)

❌ 잘못된 방식
const btn = container.querySelector('.btn');
btn.addEventListener(...);

✅ 올바른 방식
container.addEventListener('click', (e)=>{
  if(e.target.closest('.btn')){
    ...
  }
});

---

# 2. 타이머 정리

let timer;

timer = setInterval(...);

bm.onDestroy = ()=>{
  clearInterval(timer);
};

---

# 3. apply 안전 사용

bm.context.count++;
bm.apply();

주의:
onContextChange 안에서 apply 호출 금지




# 14_FAILURE_INTELLIGENCE.md
# 실전 실패 사례 분석

---

# 1. 리렌더 이벤트 소실

원인:
DOM 직접 바인딩

해결:
이벤트 위임

---

# 2. 무한 apply 루프

원인:
onContextChange 내부 apply

해결:
상태 변경 로직 분리

---

# 3. property 불일치

원인:
settings 정의 안 된 property 사용

해결:
Lint L301 규칙 적용

---

# 4. 모바일 깨짐

원인:
Desktop/Mobile 분리 미적용

해결:
2트랙 설계 강제




# 15_EXPERT_THINKING_FRAMEWORK.md
# Sixshop 전문가 사고 프레임워크

---

# 1. 항상 제약부터 본다

이 요구가 식스샵 구조에서 가능한가?

---

# 2. JS는 최후 수단

가능하면 template/style로 해결

---

# 3. 설정 패널은 UX 제품이다

운영자가 실수하지 않게 설계

---

# 4. 납품 트랙 구분

client_delivery:
- 안전 최우선

marketplace:
- 확장성 고려

---

# 5. QA는 생성보다 중요하다

코드가 아니라
PASS하는 코드가 목표

