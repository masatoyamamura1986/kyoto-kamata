# Scalable Visual Compositions（Prism 版）

Relume の複雑ビジュアル構図を **Every Layout primitives + Block** で組む手法。

## 基本方針

- ビジュアル構造は **`.l-*` Composition** で構成
- セクション固有のスタイルは **Block**（接頭辞なし）
- 余白は **rhythm semantic tokens**（`--space-heading-group` 等）
- バリアントは **`[data-*]` Exception**

## 代表パターン

### 1. 3 カラムグリッド（image + text + image）

```html
<section class="gallery u-region">
  <div class="u-wrapper">
    <div class="gallery-layout">
      <!-- DOM 順: content → portrait → square（SP での縦スタック順） -->
      <div class="gallery-content">
        <div class="l-stack" style="--stack-space: var(--space-heading-group);">
          <p data-role="tagline">Tagline</p>
          <h2 class="u-text-h2">Heading</h2>
        </div>
        <div class="l-stack" style="--stack-space: var(--space-body-group);">
          <p>Description</p>
          <div class="l-cluster"><button class="button">CTA</button></div>
        </div>
      </div>
      <div class="gallery-portrait l-frame"><img ...></div>
      <div class="gallery-square l-frame"><img ...></div>
    </div>
  </div>
</section>
```

```css
.gallery .gallery-layout {
  display: var(--flex-m, grid);
  flex-direction: var(--column-m, row);
  grid-template-columns: repeat(12, minmax(0, 1fr));
  gap: var(--space-images);
  align-items: var(--start-m, stretch);
}

.gallery .gallery-portrait { grid-column: 1 / span 5; grid-row: 1; aspect-ratio: 2/3; }
.gallery .gallery-content  { grid-column: 6 / span 4; grid-row: 1;
                              display: flex; flex-direction: column;
                              justify-content: space-between;
                              gap: var(--space-heading-to-body); }
.gallery .gallery-square   { grid-column: 10 / span 3; grid-row: 1; aspect-ratio: 1/1; }

/* SP: content → images は大きめ、image ↔ image は通常 */
@media (max-width: 50em) {
  .gallery .gallery-content {
    margin-block-end: calc(var(--space-body-to-images) - var(--space-images));
  }
}
```

**ポイント**:
- Desktop: 12-col grid で visual 順（portrait | content | square）
- SP: DOM 順で縦スタック（content → portrait → square）
- `--flex-m` 等の discrete responsive フラグで display を切替
- `align-items: var(--start-m, stretch)` で Desktop stretch / SP start

### 2. Body + Accent Image + Main Image（feature）

```html
<section class="feature u-region">
  <div class="u-wrapper">
    <div class="l-switcher" style="--switcher-threshold: 50rem;">

      <!-- Left column: body group + accent image -->
      <div class="l-stack">
        <div class="l-stack" style="--stack-space: var(--space-heading-to-body);">
          <div class="l-stack" style="--stack-space: var(--space-heading-group);">
            <p data-role="tagline">Tagline</p>
            <h2 class="u-text-h2">Heading</h2>
          </div>
          <div class="l-stack" style="--stack-space: var(--space-body-group);">
            <p>Description</p>
            <div class="l-cluster"><button class="button">CTA</button></div>
            <div class="accordion">...</div>
          </div>
        </div>
        <div class="l-frame" data-role="accent-image"><img ...></div>
      </div>

      <!-- Right column: main image -->
      <div class="l-frame" data-role="main-image"><img ...></div>

    </div>
  </div>
</section>
```

```css
.feature {
  --stack-space: var(--space-body-to-images);  /* body group ↔ accent image */
}

/* Outer switcher: desktop body↔images / SP image↔image */
.feature > .u-wrapper > .l-switcher {
  --switcher-space: var(--space-body-to-images);
}
@media (max-width: 50em) {
  .feature > .u-wrapper > .l-switcher {
    --switcher-space: var(--space-images);
  }
}
```

### 3. Showcase Cards（header + card grid）

```html
<section class="showcase u-region">
  <div class="u-wrapper">
    <div class="l-stack">  <!-- inherits .showcase { --stack-space: --space-heading-to-body } -->

      <div class="showcase-header l-stack" style="--stack-space: var(--space-heading-to-body);" data-align="center">
        <div class="l-stack" style="--stack-space: var(--space-heading-group);">
          <p data-role="tagline">Tagline</p>
          <h2 class="u-text-h2">Heading</h2>
        </div>
        <p>Description</p>
      </div>

      <div class="l-switcher" style="--switcher-threshold: 40rem;">
        <a class="card">...</a>
        <a class="card">...</a>
      </div>

    </div>
  </div>
</section>
```

```css
.showcase {
  --stack-space: var(--space-heading-to-body);  /* header ↔ cards */
}

.showcase .showcase-header {
  max-inline-size: var(--max-width-s);
  margin-inline: auto;
}
```

### 4. Hero with Multi-Image Stack

```html
<section class="layout u-region">
  <div class="u-wrapper">
    <div class="l-stack">  <!-- inherits .layout { --stack-space: --space-body-to-images } -->

      <!-- Text row: 2-col switcher -->
      <div class="l-switcher" style="--switcher-threshold: 50rem; --switcher-space: var(--space-heading-to-body);">
        <div class="l-stack" style="--stack-space: var(--space-heading-group);">
          <p data-role="tagline">Tagline</p>
          <h1 class="u-text-h1">Heading</h1>
        </div>
        <div class="l-stack layout-offset" style="--stack-space: var(--space-body-group);">
          <p>Description</p>
          <div class="l-cluster">...</div>
        </div>
      </div>

      <!-- Image row: big square + stacked smaller -->
      <div class="l-switcher" style="--switcher-threshold: 50rem; --switcher-space: var(--space-images);">
        <div class="l-frame" style="--frame-ratio: 1/1;"><img ...></div>
        <div class="l-stack" style="--stack-space: var(--space-images);">
          <div class="l-frame"><img ...></div>
          <div class="l-frame"><img ...></div>
        </div>
      </div>

    </div>
  </div>
</section>
```

```css
.layout {
  --stack-space: var(--space-body-to-images);
}

.layout .layout-offset {
  /* Desktop: offset 下方向 */
  margin-block-start: calc(var(--space-3xl) * var(--responsive-l));
}
```

## Composition 選択フロー

| 表現したいもの | Primitive |
|---|---|
| ブロック集合の縦並び | `.l-stack` |
| ボタン / タグ群（折り返し可） | `.l-cluster` |
| 2 カラム（閾値で切替） | `.l-switcher` |
| サイドバー + 本文 | `.l-sidebar` |
| 等幅カード一覧（auto-fit） | `.l-grid` |
| aspect-ratio 固定メディア | `.l-frame` |
| Hero（縦フル + 中央寄せ） | `.l-cover` |
| 枠囲み（padding + border） | `.l-box` |
| 中央寄せ記事本文 | `.l-center` |
| 横スクロール | `.l-reel` |
| inline SVG icon | `.l-icon` |
| オーバーレイ / モーダル位置 | `.l-imposter` |

## Block の命名パターン

- セクション名: `.hero`, `.feature`, `.gallery`, `.showcase`, `.faq`, `.testimonial`, `.nav`, `.footer`
- セクション内派生要素: `.gallery-content`, `.gallery-portrait`, `.showcase-header`, `.card-action`
  - 接頭辞なし + ハイフン（セクション名と整合）
- 子孫セレクタ（深さ 2 まで）も使える: `.gallery h2`, `.hero .layout-offset`

## 参考

- `every-layout-primitives.md` — 各 Composition の詳細
- `vertical-rhythm.md` — rhythm tokens の使い方
- `naming-conventions.md` — Block 命名
