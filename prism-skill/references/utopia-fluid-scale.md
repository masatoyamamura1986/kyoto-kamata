# Utopia Fluid Scale

## 出典

- https://utopia.fyi/type/calculator
- https://utopia.fyi/space/calculator

## Prism の Utopia 設定

| パラメータ | 値 | 理由 |
|---|---|---|
| Min viewport | `320px` | iPhone SE 対応（原典 default 360px より小さく） |
| Max viewport | `1240px` | 原典 default |
| Min font size | `16px` | iOS zoom 防止 + body 標準 |
| Max font size | `20px` | 原典 default |
| Min type scale | `1.2` (Minor Third) | 原典 default |
| Max type scale | `1.333` (Perfect Fourth) | SaaS / ブランドサイトのドラマ性（原典 default 1.25 より大きく） |

## Type Scale（`--step-*`）

| トークン | 用途 | 近似 px（320 → 1240px） |
|---|---|---|
| `--step--2` | 補助（キャプション等） | 11.1 → 11.2 |
| `--step--1` | 小文字 | 13.3 → 15.0 |
| `--step-0` | 本文 | 16.0 → 20.0 |
| `--step-1` | large text | 19.2 → 26.7 |
| `--step-2` | h5 | 23.0 → 35.6 |
| `--step-3` | h4 | 27.6 → 47.4 |
| `--step-4` | h3 | 33.2 → 63.2 |
| `--step-5` | h2 | 39.8 → 84.3 |
| `--step-6` | h1 | 47.8 → 112.3 |
| `--step-7` | display | 57.3 → 149.8 |

**原典との差**: 原典は `--step--2 ... --step-5` が標準。Prism は hero 用に `--step-6, 7` を追加（上方拡張）。

## Space Scale（`--space-*`）

| トークン | 近似 px（320 → 1240px） | 使い所 |
|---|---|---|
| `--space-3xs` | 4 → 5 | 極小ギャップ（tight 内側） |
| `--space-2xs` | 8 → 10 | icon ↔ text |
| `--space-xs` | 12 → 15 | rhythm: tagline ↔ h2 |
| `--space-s` | 16 → 20 | button padding |
| `--space-m` | 24 → 30 | component 内 padding |
| `--space-l` | 32 → 40 | rhythm: body group |
| `--space-xl` | 48 → 60 | section 内の大ブロック分割 |
| `--space-2xl` | 64 → 80 | rhythm: heading → body, body → images |
| `--space-3xl` | 96 → 120 | section 上下 padding |

## Space Pair（`--space-{from}-{to}`）

隣接段階のペアが拡大するにつれて変化する clamp。section の padding や marquee の間隔に便利。

- `--space-s-l`
- `--space-m-xl`
- `--space-l-2xl`
- `--space-2xl-3xl`

## Prism の Space Section Aliases

```css
--space-section-s: var(--space-xl);
--space-section-m: var(--space-2xl-3xl);   /* ← pair 使用で section 上下 padding を fluid 化 */
--space-section-l: var(--space-3xl);
--space-section-top: var(--space-3xl);
```

`.u-region` の `padding-block` はこれらを参照:
- `.u-region`: `padding-block: var(--space-section-m);`
- `.u-region[data-region="s"]`: `padding-block: var(--space-section-s);`
- `.u-region[data-region="l"]`: `padding-block: var(--space-section-l);`
- `.u-region[data-region="top"]`: `padding-block-start: var(--space-section-top);`

## 🎯 Rhythm Semantic Tokens（Pattern B セクションの心臓）

```css
--space-heading-group: var(--space-xs);      /* tagline → h1/h2/h3 */
--space-heading-to-body: var(--space-2xl);   /* h2 → description（Pattern B 見出し⇄ボディ） */
--space-body-group: var(--space-l);          /* description → cluster → accordion */
--space-body-to-images: var(--space-2xl);    /* body group → 画像ブロック */
--space-images: var(--space-l);              /* image ↔ image */
```

**Block から参照すべきはこれらの rhythm token**。raw `--space-*` は button padding / icon gap 等の非リズム用途のみ。

## どの変数を使うか

| 用途 | 使うもの |
|---|---|
| Section の内側リズム | **rhythm tokens** |
| フォントサイズ | `--step-*`（または `.u-text-*` クラス） |
| コンポーネント内 padding（button / card / box） | raw `--space-*` |
| Section の上下 padding | `--space-section-*`（alias） |
| max-width（コンテナ幅） | `--max-width-*`（Utopia `step` を幅に使わない） |
| grid-template-columns | raw `--space-*` or rem |

## ❌ NG: Utopia step を幅指定に使う

```css
grid-template-columns: var(--step-4) 1fr;  /* ❌ type scale は幅ではない */
max-width: var(--step-7);                  /* ❌ */
```

→ `--space-*` / `--max-width-*` / 直接 rem を使う。

## clamp() 式の仕組み

```
--space-l: clamp(2rem, 1rem + 3.125vw, 3rem);
           ┬────  ┬─────────────────  ┬─────
           min    preferred (slope)   max
```

- viewport が狭い → `min` 値に漸近
- viewport が広い → `max` 値に漸近
- 中間は `preferred` の線形補間

Prism では `src/styles/prism/utopia.css` に具体 clamp() 値が定義されている。

## 参考

- 原典: https://utopia.fyi/
- 実装: `src/styles/prism/utopia.css` + `src/styles/prism/tokens.css`
