# GOLPRO Design Tokens v1

전 사이트에 공통 디자인 토큰(CSS custom properties)을 주입하는 글로벌 블록입니다.

## 배치 위치
- 푸터 영역에 **한 번만** 배치하세요. (중복 배치 시 자동 감지되어 1개만 활성화됩니다.)

## 동작 방식
- 블록 로드 시 JS가 `document.head`에 `<style id="gp-design-tokens-v1">:root { ... }</style>`를 삽입합니다.
- 토큰은 `:root`에 선언되므로 페이지 내 **모든 블록**에서 `var(--gp-...)` 형태로 참조 가능합니다.
- 설정 패널에서 색상·폰트 등을 변경하면 `bm.onContextChange`가 실행되어 즉시 반영됩니다.
- 블록메이커 컴파일러가 일반 `<style>` 블록을 자동 스코핑하는 문제를 피하기 위해 JS-injection 방식을 채택했습니다.

## 제공 토큰 카테고리

### Space scale
`--gp-space-1` ~ `--gp-space-20` (4/8/12/16/20/24/32/40/48/64/80px)

### Brand colors (커스터마이즈 가능)
- `--gp-color-accent` (기본 #1A4731)
- `--gp-color-accent-hover` (기본 #133526)
- `--gp-color-accent-soft` (기본 rgba(26,71,49,0.08))
- `--gp-color-gold` (기본 #C89B3C)
- `--gp-color-gold-soft` (기본 rgba(200,155,60,0.12))

### Neutral palette (고정)
`--gp-color-neutral-50` ~ `--gp-color-neutral-900` (9단계)

### Semantic colors (일부 커스터마이즈)
- `--gp-color-bg`, `--gp-color-surface`, `--gp-color-border` (커스터마이즈 가능)
- `--gp-color-divider`, `--gp-color-text-primary/secondary/muted` (고정)
- `--gp-color-success/warning/danger` (고정)

### Typography (폰트 패밀리만 커스터마이즈)
- `--gp-font-family-heading`, `--gp-font-family-body`
- Font sizes: `--gp-font-size-xs` (11px) ~ `--gp-font-size-display` (48px) — 10단계
- Weights: regular(400), medium(500), semibold(600), bold(700), black(900)
- Line heights: tight(1.2), base(1.5), relaxed(1.75)
- Letter spacings: tight(-0.02em), normal, wide(0.05em), wider(0.1em)

### Radius
sm(4) / md(8) / lg(12) / xl(16) / 2xl(24) / full(9999)

### Shadow
xs / sm / md / lg / xl + card-hover

### Transition
Durations: fast(180ms) / base(260ms) / slow(420ms)
Easings: standard / emphasized / bounce

### Z-index
dropdown(100) / sticky(200) / fixed(300) / modal-backdrop(400) / modal(500) / popover(600) / tooltip(700)

## 사용 예시 (다른 블록 CSS에서)

```css
.myCard {
  background: var(--gp-color-surface);
  border: 1px solid var(--gp-color-border);
  border-radius: var(--gp-radius-lg);
  padding: var(--gp-space-4);
  box-shadow: var(--gp-shadow-sm);
  transition: box-shadow var(--gp-duration-base) var(--gp-ease-standard);
}
.myCard:hover {
  box-shadow: var(--gp-shadow-card-hover);
}
.myButton {
  background: var(--gp-color-accent);
  color: #FFFFFF;
  font-family: var(--gp-font-family-body);
  font-weight: var(--gp-font-weight-semibold);
  font-size: var(--gp-font-size-base);
  padding: var(--gp-space-3) var(--gp-space-5);
  border-radius: var(--gp-radius-md);
}
.myButton:hover {
  background: var(--gp-color-accent-hover);
}
```

## 블록 ID
`69df5934481eed3a2b68cdda`
