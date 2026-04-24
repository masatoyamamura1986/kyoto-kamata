# Prism 命名規則（判定フロー）

## 🎯 判定フロー

書こうとしている CSS を前にして、以下の順で自問する:

```
Step 1: このスタイル、複数箇所で使い回すか？
  - 使い回す & 1 つの CSS プロパティに寄与 → Utility (.u-*)
  - 使い回さない or 構造/コンポーネント固有 → Step 2

Step 2: Every Layout の 13 primitives のどれかか？
  （Stack / Cluster / Sidebar / Switcher / Cover / Grid / Frame / Box / Center / Reel / Icon / Imposter）
  - Yes → Composition (.l-*)
  - No → Step 3

Step 3: 特定のコンポーネント（hero, card, nav 等）の内部スタイルか？
  - Yes → Block（接頭辞なし、単純名）
  - No → Step 4

Step 4: バリアント・状態・アラインメント・サイズ・テーマの「例外ルール」か？
  - Yes → Exception（[data-*] 属性セレクタ）
```

## 接頭辞

| 層 | 接頭辞 | 命名ルール | 例 |
|---|---|---|---|
| Composition | **`.l-*`** (layout) | 単純名 / 単数形 / Every Layout 準拠 | `.l-stack`, `.l-cluster`, `.l-switcher` |
| Utility | **`.u-*`** (utility) | 意味ベース / ハイフン区切り | `.u-region`, `.u-wrapper`, `.u-text-h1` |
| Block | **なし** | 単純名 / 単数形 | `.hero`, `.card`, `.nav` |
| Exception | **`[data-*]`** 属性 | variant / state / align / size / theme 等 | `[data-variant="primary"]` |

## なぜ接頭辞（CUBE 原典と差分）

CUBE CSS 原典は `.u-*` / `.l-*` 接頭辞を明示推奨していない。Prism はこれを採用する理由:

1. **可読性**: 1 行見て `.u-region` がユーティリティと即判断できる
2. **SUIT CSS / inuitcss の業界慣習**との整合性
3. **Block 名との衝突を避ける**: `.stack` は Block とも誤読されるが `.l-stack` なら明白

CUBE 原典で Andy 自身は BEM の `__` を維持しているが、Prism は「子孫セレクタを深さ 2 まで許容」する派 — これも CUBE の "open season" 哲学の範疇。

## Block 命名の詳細

- **単純名 / 単数形**: `.card`（OK）, `.cards`（NG）, `.feature-card`（NG）
- **接頭辞なし**: `.u-card` / `.c-card` / `.b-card`（全て NG）
- **子孫セレクタは深さ 2 まで**:
  ```css
  .card h2 { ... }              /* ✅ OK */
  .card .card-action { ... }    /* ✅ OK（Block 内部の派生名） */
  .card .wrapper .list h2 { }   /* ❌ 深すぎ */
  ```
- **Block 内に別 Block を入れ子にして同名要素混在を避ける**:
  - NG: `.hero` の中に `.card` を入れて `.hero h2` と `.card h2` が競合
  - 対策: Block 内の派生要素は `.card-action` のような命名で独立

## Composition 命名の詳細

- **Every Layout 公式名に準拠**: `stack`, `cluster`, `switcher`, `sidebar`, `grid`, `frame`, `cover`, `box`, `center`, `reel`, `icon`, `imposter`
- **複数形にしない**: `.l-stacks`（NG）
- **Composition に子孫セレクタを書かない**: `.l-stack > h2 { ... }` は NG（Block 側で指定）

## Utility 命名の詳細

- **意味ベース**: `.u-region`（section の意味）, `.u-wrapper`（コンテナの意味）
- **CSS プロパティそのものの名前は避ける**: `.u-flex`（NG）, `.u-display-flex`（NG）
  - 例外: Prism では tailwind 的な utility は最小限に抑える（Composition で代用）
- **組み合わせ可**: `.u-heading` + `.u-text-h1` は両立 OK

## Exception 命名の詳細

| 属性 | 責務 | 例 |
|---|---|---|
| `data-variant` | 見た目の亜種 | `primary`, `ghost`, `danger`, `reversed` |
| `data-align` | アラインメント | `start`, `center`, `end`, `between` |
| `data-size` | サイズ | `sm`, `md`, `lg`, `xl` |
| `data-theme` | テーマ | `light`, `dark`, `brand`, `invert` |
| `data-density` | 密度 | `compact`, `comfortable`, `spacious` |
| `data-state` | **動的状態** | `active`, `open`, `expanded`, `checked`, `disabled` |
| `data-trigger` | インタラクション起点 | `hover`, `focus`, `group`, `hover-other` |
| `data-role` | JS 識別 | `trigger`, `panel`, `item`, `tagline` |

## 変数命名

| カテゴリ | プレフィックス | 例 |
|---|---|---|
| カラー（セマンティック） | `--color-*` | `--color-ink`, `--color-surface`, `--color-button` |
| スペース（raw） | `--space-*` (Utopia 直) | `--space-xs`..`--space-3xl` |
| スペース（rhythm セマンティック） | `--space-{role}` | `--space-heading-group` 等 |
| タイポサイズ（raw） | `--step-*` (Utopia 直) | `--step--2`..`--step-7` |
| フォント | `--font-*`, `--leading-*`, `--tracking-*`, `--font-weight-*` | `--font-sans`, `--leading-tight` |
| レイアウト | `--max-width-*`, `--radius-*`, `--border-*` | `--max-width-m` |
| モーション | `--ease-*`, `--duration-*` | `--ease-standard` |
| 状態 | `--state-*`, `--trigger-*`, `--responsive-*` | `--trigger-on` |
| Composition ローカル | `--{primitive}-{prop}` | `--stack-space`, `--switcher-threshold` |

**禁止**:
- `--_theme---*` / `--_spacing---*` のアンダースコア複合命名（旧 Lumos 流）
- `--foo--bar--baz` の 3 段以上の名前階層

## ファイル命名

### CSS

```
src/styles/prism/
├── main.css               # @layer 宣言 + @import
├── tokens.css
├── global.css             # Foundation
├── compositions.css
├── utilities.css
├── exceptions.css
└── utopia.css             # Utopia 生成 fluid scale
```

### Astro

- **PascalCase**: `HeroMain.astro`, `FeatureAccordion.astro`, `CardTestimonial.astro`
- **ページは kebab-case**: `pages/about.astro`

### JavaScript / TypeScript

- **kebab-case**: `component-accordion.ts`, `theme.ts`
- **named export 推奨**

### 画像

- **kebab-case** + `{context}-{subject}[-{variant}]`:
  - `hero-main.webp`, `team-yamamura-portrait.webp`, `icon-arrow-right.svg`
- サイズは命名に含めない（Astro `<Image>` srcset 自動生成）
