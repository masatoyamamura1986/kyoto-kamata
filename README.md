# Prism Design System

*One light, five lenses.*

**Lumos** / **Every Layout** / **Open Props** / **CUBE CSS** / **Utopia** — 5 つの実績あるデザインシステムを対等統合した、Astro 用のピュア CSS デザインシステム。

- **Pure CSS**（Tailwind / shadcn-ui 不使用）
- **Pattern B セクション構造** を標準化、**rhythm semantic tokens** でセクション間の余白を統一
- **CUBE CSS 方法論**（`.u-*` / `.l-*` / Block / `[data-*]`）
- **Utopia fluid scale**（320-1240px, 1.2-1.333）で viewport auto-scaling
- **動的状態は `[data-state]` に一元化**（`.is-active` 廃止）

---

## 5 レンズの責務分担

| 領域 | 担当 |
|---|---|
| カラー原色 / Shadow / Easing / Radius | **Open Props** |
| Fluid typography / space / pair | **Utopia** |
| Layout primitives（`.l-stack` 等） | **Every Layout** |
| 命名方法論（F+T+C-U-B-E） / `[data-*]` Exception | **CUBE CSS** |
| Discrete responsive / `data-trigger` / セマンティックカラー | **Lumos**（Masato 拡張） |

---

## ディレクトリ構造

```
Lumos_System/
├─ src/styles/prism/            # Prism 本体 CSS（7 ファイル）
│   ├─ main.css                 #   エントリ（@layer 宣言 + 全 @import）
│   ├─ tokens.css               #   セマンティック alias + rhythm tokens
│   ├─ utopia.css               #   Utopia 生成 fluid scale（--step-*, --space-*）
│   ├─ global.css               #   Foundation（reset, motion）
│   ├─ compositions.css         #   Every Layout 13 primitives（.l-*）
│   ├─ utilities.css            #   .u-region / .u-wrapper / .u-text-* 等
│   └─ exceptions.css           #   [data-variant] / [data-align] / [data-size] 等
│
├─ prism-skill/                 # Claude Code 用スキル
│   ├─ SKILL.md                 #   メイン指示書（372 行）
│   └─ references/              #   詳細ドキュメント 10 本
│       ├─ cube-methodology.md          # F+T+C-U-B-E レイヤー思想
│       ├─ naming-conventions.md        # .u-* / .l-* / Block / data-* 判定フロー
│       ├─ every-layout-primitives.md   # 13 primitives の詳細
│       ├─ utopia-fluid-scale.md        # --step-* / --space-* 計算式
│       ├─ open-props-integration.md    # OP raw ホワイトリスト
│       ├─ vertical-rhythm.md           # Pattern B + rhythm tokens
│       ├─ interactive-components.md    # accordion / tabs / modal（dataset.state）
│       ├─ visual-compositions.md       # Relume 的構図の組み方
│       ├─ relume-conversion.md         # Relume → Prism 変換表
│       └─ anti-patterns-full.md        # 禁止事項 29 項目（JS/CSS/a11y 含む）
│
├─ sections/                    # 検証済みセクションパターン集（カテゴリ別サブフォルダ）
│   ├─ hero/
│   │   ├─ hero-1.html
│   │   ├─ hero-2.html
│   │   └─ ...
│   ├─ features/
│   │   ├─ features-1.html
│   │   └─ ...
│   ├─ navbar/
│   ├─ layout/
│   ├─ gallery/
│   ├─ cta/
│   ├─ testimonial/
│   ├─ pricing/
│   ├─ faq/
│   ├─ team/
│   ├─ stats/
│   ├─ logo/
│   ├─ contact/
│   ├─ blog/
│   └─ footer/
│
└─ README.md                    # このファイル
```

---

## セットアップ

### 1. Astro プロジェクト作成 + Open Props インストール

```bash
npm create astro@latest my-site
cd my-site
npm install open-props
```

### 2. Prism CSS の配置

```bash
mkdir -p src/styles
cp -r Lumos_System/src/styles/prism src/styles/
```

### 3. main.css の Open Props インポートを npm path に変更

デフォルトの `src/styles/prism/main.css` は、このリポジトリの sections/ をブラウザで直接プレビューできるよう **unpkg CDN** から OP を読み込む設定になっています。Astro プロジェクトでは npm path に切り替え:

```css
/* src/styles/prism/main.css L38 */
@import "open-props/style.css" layer(openprops);
```

### 4. Layout で読み込み

```astro
---
// src/layouts/BaseLayout.astro
import '../styles/prism/main.css';
---

<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Site Title</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

### 5. Claude Code にスキル配置

```bash
mkdir -p .claude/skills
cp -r Lumos_System/prism-skill .claude/skills/
```

これで Claude Code が Prism ルール（`.u-*` / `.l-*` / Block / `[data-*]` + rhythm tokens）でコード生成する。

### 6. ブランドカスタマイズ

`src/styles/prism/tokens.css` を編集:

```css
:root {
  /* セマンティックカラー */
  --color-background: var(--gray-0);
  --color-ink: var(--gray-12);
  --color-button: var(--gray-12);
}

[data-theme="dark"] {
  --color-background: var(--gray-12);
  --color-ink: var(--gray-0);
  /* ... */
}

[data-theme="brand"] {
  --color-background: var(--green-10);
  --color-ink: var(--gray-0);
  /* ... */
}
```

---

## セクションの基本構造（Pattern B）

```html
<section class="hero u-region">
  <div class="u-wrapper">
    <div class="l-switcher" style="--switcher-threshold: 50rem; --switcher-space: var(--space-heading-to-body);">

      <!-- Heading group -->
      <div class="l-stack" style="--stack-space: var(--space-heading-group);">
        <p class="u-text-main" data-role="tagline">Tagline</p>
        <h2 class="u-text-h2">Section heading</h2>
      </div>

      <!-- Body group -->
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

3 レイヤー:
- `{block}` + `.u-region`: section 枠、theme + padding
- `.u-wrapper`: max-width + 左右余白
- `.l-switcher` / `.l-stack` / `.l-grid` 等: Every Layout primitives で内部構造

---

## 🎯 Rhythm Semantic Tokens

セクションの縦リズムは **必ず** これら 5 つのトークンで書く（raw `--space-*` は禁止）:

| Token | 値 | 使い所 |
|---|---|---|
| `--space-heading-group` | `var(--space-xs)` | tagline → h1/h2/h3 |
| `--space-heading-to-body` | `var(--space-2xl)` | h2 → description |
| `--space-body-group` | `var(--space-l)` | description → cluster → accordion |
| `--space-body-to-images` | `var(--space-2xl)` | body group → 画像ブロック |
| `--space-images` | `var(--space-l)` | image ↔ image |

値を一斉変更したい時は `tokens.css` の定義 1 行を書き換えるだけで全セクション同期。

---

## テーマ切替

```html
<section class="hero u-region" data-theme="dark">...</section>
<section class="hero u-region" data-theme="brand">...</section>
```

---

## レスポンシブ戦略（3 層）

1. **Fluid**（99% のケース）: Utopia `clamp()` で viewport auto-scaling。`@media` 不要
2. **Intrinsic**: `.l-switcher` / `.l-grid` / `.l-sidebar` がコンテナ幅で折返し
3. **Discrete**: `--responsive-l\|m\|s\|xs` + `--flex-m` / `--column-m` キーワード変数

`@media (max-width: 50em)` は **SP と Desktop で意味そのものが変わる** 例外的ケースのみ（feature 外側 switcher、gallery content margin 等）。

---

## 動的状態: `[data-state]`

```html
<div data-role="item" data-state="active">
  <button data-role="trigger" aria-expanded="true">...</button>
  <div data-role="panel">...</div>
</div>
```

```ts
trigger.addEventListener('click', () => {
  const isActive = item.dataset.state === 'active';
  item.dataset.state = isActive ? 'inactive' : 'active';
  trigger.setAttribute('aria-expanded', String(!isActive));
});
```

`.is-active` / `classList.toggle("is-*")` は **全廃**。

---

## Sections（検証済みパターン集）

`sections/{category}/` 配下に、単体で動作する完全 HTML サンプルを蓄積。各ファイルは `{category}-{N}.html` の連番命名。

**カテゴリ 15 種**:
`navbar` / `hero` / `layout` / `features` / `gallery` / `cta` / `testimonial` / `pricing` / `faq` / `team` / `stats` / `logo` / `contact` / `blog` / `footer`

新規 Relume コードを Prism に変換したら、該当カテゴリの次の連番に保存する運用。Relume → Prism の変換ルールは `prism-skill/references/relume-conversion.md` を参照。

---

## Claude Code での開発フロー

```
You: "Prism 流でプライシングセクションを作って。3 プラン横並び、中央がおすすめ"

Claude:
1. prism-skill/SKILL.md を読む（常時ロード）
2. 必要に応じて references/ を参照
3. Pattern B + rhythm tokens + [data-variant] で生成
```

Claude がチェックすべき主要禁止事項:
- raw `--space-xs/l/2xl` をセクション rhythm に直接使う（Anti-pattern #17）
- `.is-active` トグル（`[data-state]` + `dataset.state` に置換）
- `_wrap` / `_layout` / `_content` アンダースコア命名
- OP 変数をホワイトリスト外で Block 直接参照
- `@media` 多用（Fluid + Intrinsic で済む場面）

詳細は `prism-skill/references/anti-patterns-full.md`（29 項目）。

---

## CSS ファイル構造

| ファイル | 役割 |
|---|---|
| `main.css` | `@layer` 宣言 + 全 `@import`（エントリ） |
| `tokens.css` | Prism セマンティック alias（`--color-*` / rhythm tokens / `--max-width-*`） |
| `utopia.css` | Utopia fluid scale（`--step-*` / `--space-*` / `--space-pair`） |
| `global.css` | Foundation（reset, `:focus-visible`, `prefers-reduced-motion`） |
| `compositions.css` | Every Layout 13 primitives（`.l-stack` / `.l-cluster` / `.l-switcher` / `.l-container` 等） |
| `utilities.css` | `.u-region` / `.u-wrapper` / `.u-text-*` / `.u-flow` 等 |
| `exceptions.css` | `[data-variant]` / `[data-align]` / `[data-size]` / `[data-density]` |

Cascade Layers 順序:
```
openprops → utopia → prism.global → prism.tokens
  → prism.compositions → prism.utilities → prism.blocks → prism.exceptions
```

---

## 対応 / 非対応

**対応**:
- Astro + 素の CSS + Vanilla JS（TypeScript 推奨）
- Open Props via `open-props` npm package
- Cascade Layers（モダンブラウザ 2022+）

**非対応（意図的）**:
- Tailwind / shadcn-ui
- React / Vue SPA（Astro islands は OK）

---

## 参考リンク

- **CUBE CSS**: https://cube.fyi/
- **Every Layout**: https://every-layout.dev/
- **Open Props**: https://open-props.style/
- **Utopia**: https://utopia.fyi/
- **Lumos Framework**: https://www.timothyricks.com/

---

## バージョン情報

- 作成: 2026-04（Prism Design System としての正式リリース）
- ベース: Masato Lumos を CUBE 5+1 レイヤーで再編成、Every Layout / Open Props / Utopia を統合
- 環境: Astro + 素 CSS + Vanilla JS / TypeScript
