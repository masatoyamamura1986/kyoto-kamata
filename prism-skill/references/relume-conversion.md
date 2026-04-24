# Relume → Prism Conversion

Relume の Tailwind ベースのコンポーネントを Prism Design System に変換するルール。

## バッチ変換ワークフロー

ユーザーが **カテゴリ名 + Relume コード** を提示したら、以下の手順で自動処理:

1. カテゴリを特定（例: `hero`, `feature`, `cta`, `navbar`, `footer` 等 15 カテゴリ、詳細下記）
2. `ls sections/{category}/{category}-*.html 2>/dev/null` で最大連番を確認し **+1** した番号を採番（該当フォルダが無ければ連番 1 から、同時に `mkdir -p sections/{category}`）
3. Prism の規則で変換し、`sections/{category}/{category}-{N}.html` として保存
4. **1 ファイル 1 セクション**（Light 前提）。`data-theme="dark"` は「画像 + オーバーレイ」等、デザイン自体が暗い時だけ付与。両テーマ確認は DevTools で `data-theme` を付け外しすれば可能なので、マークアップを二重に書かない
5. 変換の要点をユーザーに要約返答

**ディレクトリ構造**:
```
sections/
├── hero/
│   ├── hero-1.html
│   ├── hero-2.html
├── feature/
│   ├── feature-1.html
├── cta/
│   └── cta-1.html
... （15 カテゴリ分、使われる時だけ作成）
```

**カテゴリ 15 種**: `navbar`, `hero`, `layout`, `feature`, `gallery`, `cta`, `testimonial`, `pricing`, `faq`, `team`, `stats`, `logo`, `contact`, `blog`, `footer`

**例**: ユーザー「`feature`」+ Relume code → 既存 `sections/feature/feature-3.html` まであれば → `sections/feature/feature-4.html` として保存。

**CSS パス**: 各セクションは `<link rel="stylesheet" href="../../src/styles/prism/main.css">` でルートの Prism CSS を参照（サブフォルダなので `../../` で 2 階層上がる）。

**セクション冒頭コメント規約**: 各 section 直前に 1 行コメントで概要を付ける。

形式:
```html
<!-- {filename} | {pattern description} -->
```

- `filename`: 番号付き（`hero-3` など）
- `pattern description`: レイアウト / 画像比 / バリアントを 1 行で凝縮

例:
- `<!-- hero-1 | 2-col (text LEFT, image RIGHT) + image 1/1 + center-aligned -->`
- `<!-- hero-2 | full-bleed bg image + dark overlay + min-h 100svh + data-theme="dark" -->`
- `<!-- hero-3 | image-top (SP 1/1 → PC 16/9) + 2-col text below -->`
- `<!-- hero-5 | 2-col reversed (image LEFT, text RIGHT) + data-variant="reversed" + image 1/1 -->`

grep で構造パターンを横断検索しやすいメリットあり（例: `grep -r "2-col reversed" sections/`）。

**命名規則（重要）**: ファイル固有の `<style>` 内で class を定義する場合も、class 名に**セクション番号 (hero-3) を入れない**。Prism の Block 派生名は**カテゴリ単位の汎用名**で書く:
- ✅ `.hero-media`, `.hero-cta`, `.feature-grid`, `.card-header`
- ❌ `.hero-3__media`, `.feature-4-grid`（番号を含む / BEM `__` は使わない）

番号は**ファイル名**に留める（`hero-3.html`）。class は別セクションで再利用できる汎用名にして、必要なら `data-variant="reversed"` 等の exception attribute で差別化する。

## 基本方針

1. `<section>` に Block 名 + `.u-region` を付ける（旧 `_wrap` / `.section_header-*` 廃止）
2. 内側を `.u-wrapper` でラップ
3. Composition（`.l-stack` / `.l-switcher` / `.l-grid` 等）で構造を組む
4. 余白は **rhythm tokens**（`--space-heading-group` 等）
5. 状態は `[data-state]`、バリアントは `[data-variant]`、テーマは `[data-theme]`

## Domain Blocks

Prism の `src/styles/prism/blocks.css` に定義された **domain-specific Block** を優先的に使う。`.l-*` primitives を生のまま使うより意図が明快で、余計なネストやインライン `--stack-space` 切替が減る。

### .heading-group

`tagline + h1/h2/h3 + description + cluster` のテキストブロック。**隣接セレクタで rhythm token を自動適用**するので、`.l-stack` を 2 段ネストして `--stack-space` を切り替える書き方は**禁止**。

```html
<div class="heading-group">
  <span class="tagline">Tagline (optional)</span>
  <h1 class="u-text-h1">Heading</h1>
  <p class="u-text-main">Description</p>
  <div class="l-cluster">
    <button class="button" data-variant="primary">Button</button>
  </div>
</div>
```

自動適用される rhythm:
| 隣接関係 | rhythm token |
|---|---|
| `.tagline → h1/h2/h3` | `--space-heading-group` |
| `h1/h2/h3 → p` | `--space-heading-to-body` |
| `h1/h2/h3 → .l-cluster` (description なし) | `--space-body-group` |
| `p → .l-cluster` | `--space-body-group` |

**Variants:**
| attribute | effect |
|---|---|
| `data-width="narrow"` | `max-inline-size: var(--max-content)` (36rem) |
| `data-align="center"` | 中央揃え + 36rem + cluster 中央配置（`--cluster-justify: center` 継承）※ global `[data-align="center"]` exception と合わせて text-align / align-items が同時適用 |

**`data-width="narrow"` を付けるか付けないか（判断基準）:**
- **h1 が heading-group の中**（1-col 構造 / h1+cluster 構造など）→ **付ける**。36rem キャップ
- **h1 が heading-group の外**（2-col grid/switcher で h1 が別 cell）→ **付けない**。heading-group は grid cell 幅に追従

### .hero-split

2-col hero レイアウト。SP stacked → md+ (≥48em) 2-col `minmax(0, 1fr) minmax(0, 1fr)`。

```html
<div class="hero-split">
  <div class="heading-group" data-width="narrow">...</div>
  <div class="l-frame" style="--frame-ratio: 1/1; min-block-size: 0;">
    <img src="..." alt="...">
  </div>
</div>
```

**Variants:**
| attribute | effect |
|---|---|
| `data-variant="reversed"` | md+ で列を swap（DOM 順序は SP 読み順を維持） |

**Override 変数 (inline style 上書き可能):**
| 変数 | default | 用途 |
|---|---|---|
| `--hero-split-row-gap` | `var(--space-body-to-images)` | SP stacked 時の縦 gap |
| `--hero-split-col-gap` | `var(--space-xl)` | md+ 2-col 時の横 gap |

**用途:**
- text + image split (hero-1, hero-5, hero-28, hero-29)
- ローカルで `hero-text-grid` / `hero-image-grid` 等を自作する前にまず `.hero-split` で済むか検討する

## 画像 overlay パターン

Relume で頻出する「base 画像 + 絶対配置 overlay（dim バージョン等）」の変換ルール。

### DOM 順で stacking が決まる（.l-frame 固有の罠）

`.l-frame` は `position: relative` をデフォルトで持つため、同じ親内の絶対配置 `.l-frame` overlay と stacking context 的に**同列**になる。すると DOM tree 順で前後決定され、Relume の static 前提の stacking が崩れる。

**パターン A: base → overlay (DOM 順)**
```html
<div class="hero-image-overlay">
  <div class="l-frame" style="--frame-ratio: 3/2;">  <!-- base (先) -->
    <img src="..."><
  </div>
  <div class="l-frame" style="--frame-ratio: 1/1;">  <!-- overlay (後 = 前面になる) -->
    <img src="...">
  </div>
</div>
```
→ tree order 後の overlay が自然に前面。**z-index 不要**。

**パターン B: overlay → base (DOM 順)**
```html
<div class="hero-image-overlay">
  <div class="l-frame">...</div>  <!-- overlay (先 = 背面になってしまう) -->
  <div class="l-frame">...</div>  <!-- base (後) -->
</div>
```
→ 先にある overlay が背面になってしまう。overlay に **`z-index: 1` を明示**:

```css
.hero-image-overlay > :first-child {
  position: absolute;
  z-index: 1;  /* base（.l-frame relative + z-auto）より前面に出す */
}
```

### 張り出し % の補正

Relume は `bottom-[-15%]` のようにパーセント値で overlay を画像下にはみ出させる。この値は **Relume の画像サイズ（flex-1 残スペース = 通常 150-400px）前提**。Prism で aspect-ratio を使って画像を大きくする（例: 16/9 で 800px 超）と、-15% が過剰に大きくなり text 領域にかぶる。

**ルール**: aspect-ratio で画像を full-bleed サイズにする場合、overlay の張り出しを **-3〜-5% 程度に圧縮**。Relume のピクセル感（25-60px）を保つ。

```
Relume:  画像 ~200px × -15% = 30px 張り出し
Prism aspect 16/9 at 1440:  画像 810px × -5% = 40px 張り出し (近似)
```

## full-bleed 画像 + テキストパターン

### パターン 1: 100svh + 画像 flex-leftover（Relume 標準）

Relume の hero で頻出する「`h-svh flex-col` + `flex-1` 画像 + テキスト下」構造は、**そのまま Prism に移植する**のがデフォルト:

```html
<section class="hero" style="min-block-size: 100svh; display: flex; flex-direction: column;">
  <div style="flex: 1; min-block-size: 0; overflow: hidden;">
    <img src="..." alt="..." style="block-size: 100%; inline-size: 100%; object-fit: cover;">
  </div>
  <div class="u-wrapper" style="padding-block: var(--space-xl);">
    <!-- text ... -->
  </div>
</section>
```

**注意点**:
- 画像高さ = 100svh − テキスト高さ。Prism の `.u-text-h1`（fluid max 7rem = 112px）と組み合わせると、2-col grid cell で h1 が 500-600px 占有し、画像が極端に薄くなる or 潰れることがある
- **初回は Relume 通りに移植**。気になる点があっても独断で変えず、ユーザーに「Relume 通り / カスタマイズ」を確認してから判断
- Relume の設計が壊れているケース（hero-3, hero-33 はこれだった）もある。その場合は**代替案（パターン 2）を提示 → ユーザー承認 → 実装**の順

### パターン 2: aspect-ratio で full-bleed（推奨代替）

パターン 1 で画像が潰れる場合、**aspect-ratio ベース**に切り替える。hero-3, hero-33, hero-34, hero-35 で採用済み。

```css
.hero-fullbleed-image {
  position: relative;
  min-block-size: 0;
  aspect-ratio: 1 / 1;   /* SP: portrait-ish full-bleed */
}
@media (min-width: 48em) {
  .hero-fullbleed-image {
    aspect-ratio: 16 / 9;   /* md+: landscape */
  }
}
.hero-fullbleed-image > img {
  position: absolute;
  inset: 0;
  inline-size: 100%;
  block-size: 100%;
  object-fit: cover;
}
```

**セクション構造（hero-3 パターン）:**
```html
<section class="hero u-region" style="padding-block-start: 0;">
  <div class="l-stack" style="--stack-space: var(--space-body-to-images);">
    <div class="hero-fullbleed-image">
      <img src="..." alt="...">
    </div>
    <div class="u-wrapper">
      <!-- text -->
    </div>
  </div>
</section>
```

- 画像は u-wrapper の**外**に置いて viewport 全幅（Relume の `w-screen` 相当）
- section の `padding-block-start: 0` で画像が上端まで張り付く（bottom 配置なら `padding-block-end: 0`）
- 画像 → テキスト間は `var(--space-body-to-images)` rhythm で統一

## Tailwind → Prism 変換

### レイアウト

| Tailwind (Relume) | Prism |
|---|---|
| `container mx-auto px-4` | `<div class="u-wrapper">` |
| `py-16 md:py-24 lg:py-32` | `<section class="{block} u-region">`（`.u-region` で fluid padding） |
| `flex flex-col gap-8` | `<div class="l-stack" style="--stack-space: var(--space-body-group);">` |
| `flex flex-wrap gap-4` | `<div class="l-cluster">` |
| `grid md:grid-cols-2 gap-12` | `<div class="l-switcher" style="--switcher-threshold: 50rem; --switcher-space: var(--space-heading-to-body);">` ※ Relume に `items-center` / `items-start` 等がある場合は `align-items: center;` 等を**必ず移植**（既定 stretch のままだと `.l-frame` のアスペクト比が潰れて画像変形） |
| `grid-cols-2`（生の Grid で 2-col）| `grid-template-columns: minmax(0, 1fr) minmax(0, 1fr)`（**`1fr 1fr` は NG**）。Tailwind の `grid-cols-2` は内部実装で `minmax(0, 1fr)` を使っており、min-content floor を回避している。素の `1fr` は `minmax(auto, 1fr)` 扱いで、子の intrinsic 幅でセルが膨らむ |
| `grid md:grid-cols-3 gap-8` | `<div class="l-grid" style="--grid-min: 18rem;">` |
| `aspect-square overflow-hidden` | `<div class="l-frame" style="--frame-ratio: 1/1;">` |
| `aspect-video` | `<div class="l-frame" style="--frame-ratio: 16/9;">` |
| `w-full object-cover` **(height 指定なし)** | **画像サイズは基本 Relume 準拠**。画像の実サイズから比率を読んで `.l-frame` の `--frame-ratio` を合わせる（Relume placeholder-image.svg=1/1, placeholder-image-landscape.svg=16/9）。独自の 4/3 / 21/9 等を勝手に当てない。意図的にクロップしたい場合はユーザー確認 |

**⚠️ 落とし穴**: `.l-stack`（flex column）の子として `.l-frame` を置き、aspect-ratio を media query で切り替える場合、**必ず `min-block-size: 0`** を付ける。付け忘れると flex item のデフォルト `min-block-size: auto` (= min-content) が子 img の natural size に引きずられて aspect-ratio の計算結果を上書きし、比率が効かない。`.l-switcher` / `.l-cluster`（flex row）の子では発生しない。

```css
.hero-media {
  min-block-size: 0;           /* ← flex child の min-content floor を解放 */
  aspect-ratio: 1 / 1;
}
@media (min-width: 768px) {
  .hero-media { aspect-ratio: 16 / 9; }
}
```
| `max-w-md mx-auto` | `<div class="l-center" style="--center-max: var(--max-content);">` ※ hero text の max-width は **`var(--max-content)` で統一**（tokens.css で 36rem 定義）。個別の rem 値を書かない |

**⚠️ 落とし穴**: `max-inline-size` と `padding-inline` を同一要素に付けると、Prism 既定 `box-sizing: border-box` によってコンテンツ幅 = `max-inline-size − padding` になる（例: 36rem + padding 4rem → 実コンテンツ 32rem）。grid cell 配下に置く text block は **padding-inline を付けない**（cell が余白を担う）。mobile で viewport edge から離したい場合は mobile 限定 padding、lg で `padding-inline: 0` に上書きする。
| `sr-only` | `<span class="u-visually-hidden">` |
| `hidden md:block` | `<div data-role="..."> + [data-role="..."] の CSS で discrete responsive` |

### タイポ

| Tailwind (Relume) | Prism |
|---|---|
| `text-6xl font-bold leading-tight` | `<h1 class="u-text-h1">` |
| `text-5xl font-bold` | `<h2 class="u-text-h2">` |
| `text-xl font-medium` | `<p class="u-text-large">` |
| `text-base` | `<p class="u-text-main">` |
| `text-sm` | `<p class="u-text-small">` |
| `text-xs uppercase tracking-wider` | `<span class="u-text-xsmall" style="text-transform: uppercase; letter-spacing: var(--tracking-uppercase);">` |
| `font-semibold` | `style="font-weight: var(--font-weight-medium);"`（500）or `--font-weight-bold`（700） |

### カラー

| Tailwind (Relume) | Prism |
|---|---|
| `bg-white text-black` | `<section class="{block} u-region">`（`.u-region` で `--color-background` + `--color-ink`） |
| `bg-gray-900 text-white` | `<section class="{block} u-region" data-theme="dark">` |
| `bg-primary` | `data-theme="brand"` + `--color-background: var(--green-10)` |
| `text-gray-600` | `color: color-mix(in hsl, currentColor 70%, transparent)` |

### ボタン

Relume:
```jsx
<button className="bg-black text-white px-6 py-3 rounded-lg hover:bg-gray-800">
  Click me
</button>
```

Prism:
```html
<button class="button" type="button" data-variant="primary" data-size="lg" data-trigger="hover focus">
  Click me
</button>
```

### スペーシング

**絶対に raw `--space-*` を直接使わない（section rhythm の場合）**。

| Tailwind | Prism |
|---|---|
| `gap-2` (0.5rem) | 概念として heading group 内 → `var(--space-heading-group)` |
| `gap-4` (1rem) | button icon gap 等 → raw `var(--space-xs)` OK |
| `gap-6` (1.5rem) | body group 内 → `var(--space-body-group)` |
| `gap-12` (3rem) | heading → body / body → images → `var(--space-heading-to-body)` / `var(--space-body-to-images)` |

判断基準: **section rhythm（見出し ⇄ 本文 ⇄ 画像）** → rhythm tokens。それ以外（button padding, icon gap）→ raw tokens OK。

### 状態

Relume:
```jsx
<div className={isActive ? "opacity-100" : "opacity-0"}>
```

Prism:
```html
<div data-state="active">
```
```css
[data-role="panel"] {
  opacity: calc(1 * var(--state-true));
  transition: opacity var(--duration-main) var(--ease-standard);
}
```

## フルセクション変換例

### Before（Relume / Tailwind）

```jsx
<section className="py-24 bg-white">
  <div className="container mx-auto px-4">
    <div className="grid md:grid-cols-2 gap-12 items-center">
      <div>
        <p className="text-sm font-bold mb-4">Tagline</p>
        <h2 className="text-5xl font-bold mb-6">Heading</h2>
      </div>
      <div>
        <p className="text-base mb-8">Description...</p>
        <button className="bg-black text-white px-6 py-3 rounded-lg">
          CTA
        </button>
      </div>
    </div>
  </div>
</section>
```

### After（Prism）

```html
<section class="hero u-region">
  <div class="u-wrapper">
    <div class="l-switcher" style="--switcher-threshold: 50rem; --switcher-space: var(--space-heading-to-body);">

      <div class="l-stack" style="--stack-space: var(--space-heading-group);">
        <p class="u-text-main" data-role="tagline" style="font-weight: var(--font-weight-bold);">Tagline</p>
        <h2 class="u-text-h2">Heading</h2>
      </div>

      <div class="l-stack" style="--stack-space: var(--space-body-group);">
        <p class="u-text-main">Description...</p>
        <div class="l-cluster">
          <button class="button" data-variant="primary" data-size="lg" data-trigger="hover focus">CTA</button>
        </div>
      </div>

    </div>
  </div>
</section>
```

## 旧 Lumos → Prism の追加変換

Masato Lumos 資産を持ち込む場合:

| Lumos | Prism |
|---|---|
| `<section class="hero_wrap u-section">` | `<section class="hero u-region">` |
| `<div class="hero_contain u-container">` | `<div class="u-wrapper">` |
| `<div class="hero_layout">` | `<div class="l-stack">` or `.l-switcher` / `.l-grid` |
| `<div class="hero_content u-margin-trim">` | `<div class="l-stack">` |
| `<p class="hero_tagline u-text-style-main">` | `<p class="u-text-main" data-role="tagline">` |
| `<h1 class="hero_title u-text-style-h1">` | `<h1 class="u-text-h1">` |
| `<button class="hero_button">` | `<button class="button" data-variant="primary">` |
| `class="is-active"` | `data-state="active"` |
| `class="is-reversed"` | `data-variant="reversed"` |
| `class="u-theme-dark"` | `data-theme="dark"` |
| `class="u-sr-only"` | `class="u-visually-hidden"` |
| `var(--_theme---text)` | `var(--color-ink)` |
| `var(--_spacing---space--5)` | **rhythm 用途なら** `var(--space-heading-to-body)` など / **非リズムなら** `var(--space-m)` |

## Lint ルール（自動検出）

`.claude/hooks/prism-lint.sh` が `sections/*.html` の Write/Edit 時に以下を自動検出:

| # | Rule | 検出内容 |
|---|---|---|
| 1 | `grid-template-columns` は `minmax(0, 1fr)` | raw の `1fr 1fr` / `repeat(N, 1fr)` |
| 2 | class 名にセクション番号を入れない | `.hero-3__media` のような番号入り名 |
| 3 | section 冒頭コメント必須 | `<!-- {filename} \| {pattern} -->` 形式 |
| 4 | text block 幅は `var(--max-content)` | `max-inline-size: 32rem` 等の raw 値 |
| 5 | vertical rhythm は rhythm token | `--stack-space` / `row-gap` / 小 `gap` に raw `--space-*` |
| 6 | `max-inline-size` + `padding-inline` 衝突 | 同一 inline style に両方 |

**lint で検出できない（変換時に意識するルール）:**
- `.l-stack`（flex column）直下の `.l-frame` には `min-block-size: 0` が必須（aspect-ratio potato 化対策）→ [feedback_flex_aspect_ratio_gotcha.md](~/.claude/projects/-Users-yamamuramasato-Documents-Lumos-System/memory/feedback_flex_aspect_ratio_gotcha.md)
- `.heading-group` の `data-width="narrow"` は h1 が内部の時のみ → [feedback_heading_group_block.md](~/.claude/projects/-Users-yamamuramasato-Documents-Lumos-System/memory/feedback_heading_group_block.md)
- overlay が base より DOM 先にある時のみ `z-index: 1` → [feedback_lframe_absolute_overlay_zindex.md](~/.claude/projects/-Users-yamamuramasato-Documents-Lumos-System/memory/feedback_lframe_absolute_overlay_zindex.md)

## 参考

- `vertical-rhythm.md` — rhythm tokens 使用ルール
- `naming-conventions.md` — Block 命名
- `visual-compositions.md` — Relume 的構図の組み方
