# Vertical Rhythm — Prism Pattern B + Rhythm Tokens

セクション内の縦方向の余白設計ルール。**rhythm semantic tokens** で記述するのが唯一の方法。

## 🎯 Pattern B を唯一の標準にする（Prism の方針）

Lumos 時代は Pattern A（連続）/ B（分割）/ C（中央短見出し）の 3 パターンを使い分けていた。Prism ではそれらを統合し、**Pattern B（heading group / body group を分離する構造）を標準**にして rhythm semantic tokens で記述する。中央寄せヘッダーのような「tight header」も Pattern B を入れ子にして実装する。

## Rhythm Semantic Tokens（5 個で全セクション統制）

| Token | 値 | 使い所 |
|---|---|---|
| `--space-heading-group` | `var(--space-xs)` | tagline → h1/h2/h3 |
| `--space-heading-to-body` | `var(--space-2xl)` | h2 → description（heading group ⇄ body group） |
| `--space-body-group` | `var(--space-l)` | description → cluster → accordion 等 |
| `--space-body-to-images` | `var(--space-2xl)` | body group → 画像ブロック |
| `--space-images` | `var(--space-l)` | image ↔ image |

**raw `--space-*` を rhythm に使うのは禁止**（Anti-pattern #17）。button padding / icon gap 等の非リズム余白のみ raw 使用 OK。

### 「section rhythm」と「Block 内部 rhythm」の線引き

rhythm tokens は **section 全体の縦リズム** に使う。Block 内部の「独自レジスター」を持つ compact な stack — 例えば card の `tagline → h3 → details` の内側リズム — は **raw `--space-s` / `--space-m` 使用 OK**。

**判断基準**:
- **section rhythm**: `.u-region > .u-wrapper > .l-stack` に直接現れる heading/body グループ、セクション間の余白 → **rhythm tokens 必須**
- **Block 内部 rhythm**: `.card [data-role="content"]`、`.nav-menu` 内部、`.modal [data-role="content"]` 等、section の rhythm と独立した Block 固有の縦並び → **raw tokens OK**

例（OK）:
```html
<!-- card の内側は独自レジスター -->
<a class="card">
  <div class="l-stack" data-role="content" style="--stack-space: var(--space-s);">
    <p data-role="tagline">...</p>
    <h3>...</h3>
    <div class="l-stack" data-role="details" style="--stack-space: var(--space-m);">...</div>
  </div>
</a>
```

例（NG — section rhythm に raw）:
```html
<!-- section 直下の .l-stack に raw は ❌ -->
<section class="hero u-region">
  <div class="u-wrapper">
    <div class="l-stack" style="--stack-space: var(--space-2xl);">  <!-- ❌ --space-heading-to-body を使う -->
      ...
    </div>
  </div>
</section>
```

## Canonical Pattern B 構造

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
        <p class="u-text-main">Lorem ipsum...</p>
        <div class="l-cluster">
          <button class="button" data-variant="primary" data-size="lg">CTA</button>
          <button class="link">Secondary</button>
        </div>
      </div>

    </div>
  </div>
</section>
```

**SP で switcher が縦スタック時**:
- tagline
- ↕ `--space-heading-group` (xs)
- h2
- ↕ `--space-heading-to-body` (2xl)  ← switcher-space
- description
- ↕ `--space-body-group` (l)
- buttons

## Tight Header（中央寄せの 3 要素）

tagline + h2 + subcopy を視覚的に 1 ユニットとして見せたい場合、Pattern B を入れ子にする:

```html
<div class="showcase-header l-stack" style="--stack-space: var(--space-heading-to-body);" data-align="center">
  <!-- Heading group（内側） -->
  <div class="l-stack" style="--stack-space: var(--space-heading-group);">
    <p data-role="tagline">Tagline</p>
    <h2 class="u-text-h2">Heading</h2>
  </div>
  <!-- Subcopy -->
  <p class="u-text-main">Lorem ipsum...</p>
</div>
```

これで:
- tagline → h2 = `--space-heading-group` (xs)
- h2 → subcopy = `--space-heading-to-body` (2xl)

## 画像ブロックを含むセクション（Image-rich Section）

### Example: Hero with image row

```html
<section class="layout u-region">
  <div class="u-wrapper">
    <div class="l-stack">  <!-- inherits .layout { --stack-space: --space-body-to-images } -->

      <!-- Text row: heading switcher -->
      <div class="l-switcher" style="--switcher-space: var(--space-heading-to-body);">
        <div class="l-stack" style="--stack-space: var(--space-heading-group);">...</div>
        <div class="l-stack" style="--stack-space: var(--space-body-group);">...</div>
      </div>

      <!-- Image row -->
      <div class="l-switcher" style="--switcher-space: var(--space-images);">
        <div class="l-frame" style="--frame-ratio: 1/1;"><img ...></div>
        <div class="l-stack" style="--stack-space: var(--space-images);">
          <div class="l-frame" style="--frame-ratio: 1/1;"><img ...></div>
          <div class="l-frame" style="--frame-ratio: 3/2;"><img ...></div>
        </div>
      </div>

    </div>
  </div>
</section>
```

```css
.layout {
  --stack-space: var(--space-body-to-images);  /* text-row ↔ image-row */
}
```

これで:
- text-row → image-row = `--space-body-to-images` (2xl)
- image-row 内の画像 ↔ 画像 = `--space-images` (l)

## @media 例外（SP と Desktop で意味が変わる場合）

### 例 1: feature 外側 switcher

Desktop では「body column ↔ image column」の区切り（body-to-images = 2xl）、SP では縦スタック時に「accent image ↔ main image」（images = l）。

```css
.feature > .u-wrapper > .l-switcher {
  --switcher-space: var(--space-body-to-images);
}

@media (max-width: 50em) {
  .feature > .u-wrapper > .l-switcher {
    --switcher-space: var(--space-images);
  }
}
```

HTML:
```html
<section class="feature u-region">
  <div class="u-wrapper">
    <div class="l-switcher" style="--switcher-threshold: 50rem;">
      <div class="l-stack">
        <!-- body group + accent image -->
      </div>
      <div class="l-frame"><img ...></div>  <!-- main image -->
    </div>
  </div>
</section>
```

### 例 2: gallery content → images margin

Gallery は 3 カラムグリッドで content / portrait / square を並べる。Desktop は 3 カラム gap = images (l)。SP は content → portrait = body-to-images (2xl) にしたいが、grid の単一 `gap` では portrait → square = l と両立不可。→ SP のみ `gallery-content` に extra `margin-block-end` で補正:

```css
.gallery .gallery-layout {
  gap: var(--space-images);
}

@media (max-width: 50em) {
  .gallery .gallery-content {
    margin-block-end: calc(var(--space-body-to-images) - var(--space-images));
  }
}
```

## 「同じ要素に `.l-stack` と `.u-flow` を付けない」

- `.l-stack` は flex gap — margin ベースの `.u-flow` と競合する
- 記事本文の `<p>` 連続には `.u-flow`、ブロック混在（h2 + p + button + img）には `.l-stack`

## 参考

- `utopia-fluid-scale.md` — 各 rhythm token の実値
- `every-layout-primitives.md` — `.l-stack` / `.l-switcher` の詳細
- `anti-patterns-full.md` — raw `--space-*` 直参照禁止（#17）
