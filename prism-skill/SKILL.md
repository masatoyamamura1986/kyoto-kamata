---
name: prism-skill
description: "Build responsive Astro websites with the Prism Design System — a 5-lens hybrid of Lumos / Every Layout / Open Props / CUBE CSS / Utopia. Triggers on Prism CSS (.u-*, .l-*, data-*), semantic rhythm tokens (--space-heading-group, --space-body-group, --space-heading-to-body, --space-body-to-images, --space-images), Pattern B section layout, [data-state] dynamic toggling, or Relume-to-Prism conversion."
---

# Prism Skill

**Prism Design System** — *One light, five lenses.*

Pure CSS design system for Astro. No Tailwind, no shadcn/ui.
5 要素対等統合: **Lumos**（discrete responsive + `data-trigger`）/ **Every Layout**（13 `.l-*` primitives）/ **Open Props**（tokens raw）/ **CUBE CSS**（F+T+C-U-B-E 方法論）/ **Utopia**（fluid typography + space）。

## 環境前提

- **Astro** ベース、`<style>` scoped を活用
- 素の CSS（Tailwind / shadcn/ui 不使用）
- エントリ: `src/styles/prism/main.css`（Open Props + Utopia + Prism レイヤー群を `@layer` で統合）
- Block の `<style>` は `<section>` の**最初の子**、`<script>` は**最後の子**として配置

```astro
<section class="hero u-region">
  <style>/* Block CSS */</style>
  <div class="u-wrapper">
    <!-- markup -->
  </div>
  <script>/* Block JS */</script>
</section>
```

---

## 判断フロー（最初に読むフローチャート）

```
質問: この CSS は何？

①「複数箇所で使い回す単一目的のクラス」？
  YES → Utility `.u-*`
    例: .u-region, .u-wrapper, .u-flow, .u-text-h1, .u-visually-hidden, .u-skeleton

② Every Layout のレイアウトプリミティブ？
  YES → Composition `.l-*`
    例: .l-stack, .l-cluster, .l-switcher, .l-sidebar, .l-grid, .l-frame, .l-cover, .l-box

③ 固有のコンポーネント（hero / card / nav / faq など）？
  YES → Block（接頭辞なし、単純名・単数形）
    例: .hero, .card, .nav, .button, .accordion
    子孫セレクタは深さ 2 まで可: .card h2, .hero .hero-offset

④ バリアント・状態・アラインメント等の「例外」？
  YES → Exception `[data-*]`（クラスは使わない）
    例: [data-variant="primary"], [data-state="active"], [data-align="center"]
        [data-size="lg"], [data-theme="dark"], [data-trigger="hover"]
```

---

## 命名規則（CUBE 原典 + コミュニティ慣習）

| 層 | 接頭辞 | 例 |
|---|---|---|
| Composition | `.l-*` | `.l-stack`, `.l-cluster`, `.l-switcher`, `.l-sidebar`, `.l-grid`, `.l-frame`, `.l-cover`, `.l-box`, `.l-center`, `.l-reel`, `.l-icon`, `.l-imposter` |
| Utility | `.u-*` | `.u-region`, `.u-wrapper`, `.u-flow`, `.u-text-display`..`.u-text-small`, `.u-heading`, `.u-text`, `.u-visually-hidden`, `.u-skeleton`, `.u-image-contain` |
| Block | 接頭辞なし | `.hero`, `.card`, `.nav`, `.button`, `.accordion`, `.gallery`, `.feature` |
| Exception | `[data-*]` | `[data-variant]`, `[data-state]`, `[data-align]`, `[data-size]`, `[data-theme]`, `[data-density]`, `[data-trigger]`, `[data-role]` |

CSS 変数の命名:
- **Prism セマンティック**: `--color-ink`, `--color-surface`, `--color-button`, `--font-weight-bold`, `--tracking-tight`, `--leading-main`, `--max-width-m`, `--radius-s`, `--border-main`, `--ease-standard`, `--duration-main`, `--nav-height`
- **Rhythm**（セクション Pattern B 専用）: `--space-heading-group`, `--space-heading-to-body`, `--space-body-group`, `--space-body-to-images`, `--space-images`
- **raw（Utopia / OP）**: `--step--2`..`--step-7`, `--space-3xs`..`--space-3xl`, `--space-s-l` 等, `--gray-0`..`--gray-12`, `--radius-2`, `--shadow-*`, `--ease-*`
- **Composition ローカル**: `--stack-space`, `--cluster-space`, `--switcher-space`, `--switcher-threshold`, `--sidebar-side`, `--frame-ratio`, `--grid-min`, `--cover-min-height`, `--box-padding`

禁止:
- `--_theme---*` / `--_spacing---*` など旧 Lumos のアンダースコア複合命名
- Block 単独名に `.c-*` / `.b-*` 等の接頭辞を付ける

---

## 🎯 セクション rhythm の唯一のルール（最重要）

セクションの縦リズムには **rhythm tokens** を使う。`--space-xs` / `--space-l` / `--space-2xl` を直接使ってはいけない（Anti-pattern #17）。

| Rhythm Token | 値 | 使い所 |
|---|---|---|
| `--space-heading-group` | `var(--space-xs)` | tagline → h1/h2/h3 |
| `--space-heading-to-body` | `var(--space-2xl)` | h2 → description（Pattern B の見出し⇄ボディ） |
| `--space-body-group` | `var(--space-l)` | description → cluster → accordion |
| `--space-body-to-images` | `var(--space-2xl)` | body group → 画像ブロック |
| `--space-images` | `var(--space-l)` | image ↔ image |

**Raw token の使い所**: button padding / icon gap / component 内部 padding のような**非リズム余白**。例: `.button { padding-inline: var(--space-l); }` は OK（rhythm ではない）。

**値を一斉調整したい時**: `src/styles/prism/tokens.css` の rhythm token 定義 1 行を変更するだけで全セクション同期。

---

## Pattern B の標準構造（Prism の基本セクション）

```html
<section class="hero u-region">
  <div class="u-wrapper">
    <div class="l-switcher" style="--switcher-threshold: 50rem; --switcher-space: var(--space-heading-to-body);">

      <!-- Heading group -->
      <div class="l-stack" style="--stack-space: var(--space-heading-group);">
        <p class="u-text-main" data-role="tagline" style="font-weight: var(--font-weight-bold);">Tagline</p>
        <h2 class="u-text-h2">Medium length section heading goes here</h2>
      </div>

      <!-- Body group -->
      <div class="l-stack" style="--stack-space: var(--space-body-group);">
        <p class="u-text-main">Lorem ipsum dolor sit amet...</p>
        <div class="l-cluster">
          <button class="button" data-variant="primary" data-size="lg" data-trigger="hover focus">CTA</button>
          <button class="link" data-trigger="hover focus">Secondary</button>
        </div>
      </div>

    </div>
  </div>
</section>
```

**構造のポイント**:
- `<section>` は `.{block} .u-region`（`.u-region` が `padding-block` と theme 色を提供）
- `<div class="u-wrapper">` が max-width + margin-inline + container-type
- 中に `.l-switcher` / `.l-stack` / `.l-grid` 等の Composition を配置
- 見出しグループとボディグループを別の `.l-stack` に分離 → rhythm tokens で余白を明示
- Desktop では switcher が 2 カラム、SP では縦スタック

---

## SP 対応の 3 層メンタルモデル

> **99% のケースで `@media` は書かない**。Fluid scaling + rhythm tokens で自動対応できる。

### 層 1: Fluid auto-scaling（Utopia clamp）

Utopia の `clamp()` トークンが viewport に応じて自動縮小:
```
--space-2xl: clamp(3rem, 2rem + 5vw, 5rem)
  → 320px viewport: 48px
  → 1240px viewport: 80px
```
Desktop と SP で「2xl」の概念は同じ、物理値だけ違う。

### 層 2: Semantic rhythm tokens

`--space-heading-group` 等を参照すれば、Desktop も SP も同じコードで意味を記述できる。`@media` 不要。

### 層 3: `@media (max-width: 50em)` の 2 例外

**SP と Desktop で "意味そのもの" が変わる場合のみ** `@media` を使う:

**例 1: feature 外側 switcher**
```css
/* Desktop: body-col ↔ image-col */
.feature > .u-wrapper > .l-switcher {
  --switcher-space: var(--space-body-to-images);
}
/* SP: accent image ↔ main image */
@media (max-width: 50em) {
  .feature > .u-wrapper > .l-switcher {
    --switcher-space: var(--space-images);
  }
}
```

**例 2: gallery content → images margin**
単一 `gap` では「content→portrait = 2xl / portrait→square = l」を両立不可なので、gallery-content に SP のみ margin-block-end で補正:
```css
.gallery .gallery-layout { gap: var(--space-images); }
@media (max-width: 50em) {
  .gallery .gallery-content {
    margin-block-end: calc(var(--space-body-to-images) - var(--space-images));
  }
}
```

---

## 動的状態の唯一の表現: `[data-state]`

**`.is-active` / `.is-open` / `classList.toggle("is-*")` は全廃**。

```html
<div data-role="item" data-state="active">
  <button data-role="trigger" aria-expanded="true" data-trigger="hover focus">...</button>
  <div data-role="panel" role="region">...</div>
</div>
```

```ts
trigger.addEventListener('click', () => {
  const isActive = item.dataset.state === 'active';
  item.dataset.state = isActive ? 'inactive' : 'active';
  trigger.setAttribute('aria-expanded', String(!isActive));
});
```

CSS 側は tokens.css で自動変換:
```css
[data-state="active"], [aria-expanded="true"] {
  --state-true: 1;
  --state-false: 0;
}
```

---

## 要素識別: `data-role`

JS でターゲットする要素はクラスで identify せず `data-role` を使う:

```html
<button data-role="trigger">...</button>
<div data-role="panel">...</div>
<p data-role="tagline">...</p>
```

```ts
component.querySelector('[data-role="trigger"]')
```

---

## Composition 早見表（13 primitives）

| Primitive | ローカル変数 | 用途 |
|---|---|---|
| `.l-stack` | `--stack-space` | 縦並び（flex gap） |
| `.l-cluster` | `--cluster-space`, `--cluster-justify`, `--cluster-align` | インライン群（折返し可） |
| `.l-switcher` | `--switcher-threshold`, `--switcher-space`, `--switcher-limit` | 閾値で 1 行 ↔ 縦切替 |
| `.l-sidebar` | `--sidebar-side`, `--sidebar-content-min`, `--sidebar-space` | サイドバー + 本文 |
| `.l-grid` | `--grid-min`, `--grid-space` | auto-fit minmax |
| `.l-frame` | `--frame-ratio` | aspect-ratio + object-fit |
| `.l-cover` | `--cover-min-height`, `--cover-space` | 縦フル + 中央寄せ |
| `.l-box` | `--box-padding`, `--box-border` | padded container |
| `.l-center` | `--center-max`, `--center-gutters` | 中央寄せ + max-width |
| `.l-container` | — | Container Query scope のみ（max-width 不要なら `.u-wrapper` より軽量） |
| `.l-reel` | `--reel-height`, `--reel-space`, `--reel-item-width` | 横スクロール |
| `.l-icon` | `--icon-size`, `--icon-space` | インライン SVG |
| `.l-imposter` | `--imposter-margin`, `[data-fixed]` | オーバーレイ |

補足:
- `.u-flow > * + *` は lobotomized owl（記事本文の縦リズム）。`.l-stack` と同じ要素に付けない。
- `.l-stack-recursive` — 入れ子すべてに stack を適用（Every Layout 原典機能）。
- `.l-stack[data-split-after="N"]` — N 番目の子のあとに `margin-block-end: auto` を付けて以降を下に押し出し（Every Layout splitAfter）。

---

## Exception data-* 早見表

| 属性 | 値 | 対象 |
|---|---|---|
| `data-variant` | `primary`, `ghost`, `danger`, `reversed` | button, card, badge |
| `data-align` | `start`, `center`, `end`, `between`, `around` | Composition |
| `data-size` | `sm`, `md`, `lg`, `xl` | button, icon, container |
| `data-theme` | `light`, `dark`, `brand`, `invert` | section, block |
| `data-density` | `compact`, `comfortable`, `spacious` | list, card |
| `data-state` | `active`, `inactive`, `open`, `closed`, `expanded`, `collapsed`, `checked`, `disabled`, `loading` | **動的状態** |
| `data-trigger` | `hover`, `focus`, `group`, `hover-other`, `focus-other` | behavioral（`--trigger-on/off` を 0/1 切替） |
| `data-role` | `trigger`, `panel`, `item`, `tagline`, `close`, `overlay` | JS 識別 |

---

## Core Utilities (`.u-*`)

| クラス | 用途 |
|---|---|
| `.u-region` | section 枠（padding-block + 背景色 + 文字色） |
| `.u-wrapper` | max-width + margin-inline + padding-inline + container-type |
| `.u-flow > * + *` | テキスト縦リズム（lobotomized owl） |
| `.u-heading` / `.u-text` | max-width 継承用ラッパー |
| `.u-text-display` / `.u-text-h1`..`.u-text-h6` / `.u-text-large` / `.u-text-main` / `.u-text-small` / `.u-text-xsmall` | タイポスタイル（`--step-*` 使用） |
| `.u-visually-hidden` | スクリーンリーダー専用 |
| `.u-skeleton` | ロード前の背景色 |
| `.u-image-contain` | `object-fit: contain`（`.l-frame` 内のロゴ等） |

---

## References

- `references/cube-methodology.md` — CUBE の F+T+C-U-B-E レイヤー思想 + Cascade Layers
- `references/naming-conventions.md` — `.u-*` / `.l-*` / Block / `data-*` 判定フロー
- `references/every-layout-primitives.md` — 13 primitives の詳細と使い分け
- `references/utopia-fluid-scale.md` — `--step-*` / `--space-*` / pair 計算式 + 設定値
- `references/open-props-integration.md` — raw OP 変数のホワイトリスト
- `references/vertical-rhythm.md` — Pattern B rhythm tokens 詳細 + @media 例外
- `references/interactive-components.md` — accordion / tabs / modal（`dataset.state` ベース）
- `references/visual-compositions.md` — Scalable Visual Composition
- `references/relume-conversion.md` — Relume → Prism 変換
- `references/anti-patterns-full.md` — Anti-pattern 完全リスト（29 項目、命名/Tokens/JS/CSS/a11y）

値の具体値（ブレークポイント / max-width / space 数値）は `src/styles/prism/tokens.css` + `src/styles/prism/utopia.css` を参照。

---

## 旧 Lumos → Prism 対応表

### クラス

| Lumos | Prism |
|---|---|
| `u-section` | `.u-region` |
| `u-container` | `.u-wrapper` |
| `u-margin-trim` | 不要（`.l-stack` or `.u-flow`） |
| `u-button-wrapper` | `.l-cluster` |
| `u-alignment-center` | `[data-align="center"]` |
| `u-text-style-display\|h1..h6\|large\|main\|small` | `.u-text-display` / `.u-text-h1` / ... / `.u-text-small` |
| `u-theme-light\|dark\|brand` | `[data-theme="light\|dark\|brand"]` |
| `u-sr-only` | `.u-visually-hidden` |
| `u-background-skeleton` | `.u-skeleton` |
| `u-img-contain` | `.u-image-contain` |
| `u-display-none` | `[hidden]` |
| `.is-active` | `[data-state="active"]` |
| `.is-reversed` | `[data-variant="reversed"]` |
| `hero_wrap hero_contain hero_layout` | `<section class="hero u-region">` / `<div class="u-wrapper">` / `<div class="l-stack">` |
| `hero_tagline` | `<p data-role="tagline">` |
| `hero_title u-text-style-h1` | `<h1 class="u-text-h1">` |
| `hero_button` | `<button class="button" data-variant="primary">` |

### CSS 変数

| Lumos | Prism |
|---|---|
| `--_theme---background` | `--color-background` |
| `--_theme---background-2` | `--color-surface` |
| `--_theme---text` | `--color-ink` |
| `--_theme---border` | `--color-border` |
| `--_theme---button-primary--background` | `--color-button` |
| `--_theme---button-primary--text` | `--color-button-ink` |
| `--_theme---text-link--text` / `-hover` | `--color-link` / `--color-link-hover` |
| `--_theme---shadow--1\|2` | `--color-shadow-1\|2` |
| `--_spacing---space--1..8` | `--space-3xs..3xl`（Utopia 直） |
| `--_spacing---section-space--small\|main\|large` | `--space-section-s\|m\|l` |
| `--_spacing---gutter` | `--gutter` |
| `--_text-style---margin-bottom` | `--flow-space` |
| `--_typography---family--primary` | `--font-sans` |
| `--_typography---font--primary-bold` | `--font-weight-bold` |
| `--_typography---letter-spacing--*` | `--tracking-*` |
| `--_typography---line-height--*` | `--leading-*` |
| `--_typography---size--h1..display` | `.u-text-h1` クラス or `--step-*` |
| `--max-width--small\|main\|full` | `--max-width-s\|m\|full` |
| `--site--margin\|gutter` | `--site-margin\|gutter` |
| `--radius--small\|main\|round` | `--radius-s\|m\|round` |
| `--border-width--main` | `--border-main` |
| `--nav--height-main\|total` | `--nav-height\|nav-offset` |
| `--_motion---ease-*` | `--ease-*` |
| `--_motion---duration-*` | `--duration-*` |
| `--_responsive---large\|medium\|small\|xsmall` | `--responsive-l\|m\|s\|xs` |
| `--_trigger---on\|off` | `--trigger-on\|off` |
| `--_state---true\|false` | `--state-true\|false` |

---

## 禁止リスト（Anti-patterns まとめ）

1. **raw `--space-*` をセクション rhythm に使う** → rhythm tokens 使用（詳細 `anti-patterns-full.md` #17）
2. **`.is-active` トグル / `classList.toggle("is-*")`** → `[data-state]` + `element.dataset.state`
3. **`--_theme---*` / `--_spacing---*`** アンダースコア複合命名 → `--color-*` / `--space-*`
4. **Block に `.c-*` / `.b-*` / `.u-*` / `.l-*` 接頭辞を付ける** → Block は接頭辞なし単純名
5. **Utility / Composition に接頭辞を付けない** → `.u-*` / `.l-*` 必須
6. **Composition 内で子孫セレクタ**（`.l-stack > h2`）→ Block 側で指定
7. **3 段以上の子孫セレクタ** → 2 段まで
8. **OP 変数のホワイトリスト外直接参照**（`--gray-5`, `--size-*`, `--font-weight-5` 等）→ Prism セマンティック層経由
9. **Utopia `--step-*` を幅（max-width, grid-template-columns）に使う** → `--max-width-*` / `--space-*`
10. **`.l-stack` と `.u-flow` を同じ要素に付ける** → どちらか片方
11. **`.l-switcher --switcher-threshold` を `vw` で書く** → `rem` / `em`
12. **`_wrap` / `_layout` / `_content`** アンダースコア命名 → 接頭辞なし単純名 + Composition
13. **Exception を `.is-reversed` クラスで表現** → `[data-variant="reversed"]`
14. **`id` で DOM 選択**（ARIA 用途以外）→ `data-role`
15. **`window.innerWidth` で判定** → `--responsive-l\|m\|s\|xs` + `getComputedStyle`
16. **`innerHTML +=` で視覚構造生成** → `_hidden` テンプレートから `cloneNode(true)`
17. **TS で `any`** → 正確な型指定
