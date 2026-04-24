# Open Props Integration

## 出典

https://open-props.style/

## 原則: Prism セマンティック層を経由する

Open Props（OP）は `--gray-0..12`, `--shadow-1..6`, `--radius-1..5`, `--ease-1..5` 等 500+ の raw トークンを提供するが、**Block から直接参照してはいけない**。まず Prism セマンティック alias に変換する。

```css
/* src/styles/prism/tokens.css */
--color-ink: var(--gray-12);        /* OP gray-12 → Prism 意味層 */
--color-button: var(--gray-12);
--color-surface: var(--gray-1);
--color-shadow-1: var(--shadow-2);
--radius-s: var(--radius-2);
```

Block では Prism セマンティック alias だけを参照:

```css
.button { color: var(--color-button-ink); }    /* ✅ */
.button { color: var(--gray-0); }              /* ❌ OP raw 直参照 */
```

## ホワイトリスト（Block から直接参照可の raw）

以下の raw トークンは Block から直接参照しても良い:

| カテゴリ | 変数 | 理由 |
|---|---|---|
| カラー（theme 再定義のみ） | `--gray-0..12`, `--green-*` 等 | `[data-theme="dark"]` の `--color-*` 再定義で使う |
| **レイアウト（Utopia 直）** | `--space-3xs..3xl`, `--space-s-l` 等 | Prism の rhythm token が参照する基盤 |
| **タイポサイズ（Utopia 直）** | `--step--2..7` | Prism の `.u-text-*` が参照する基盤 |
| 形状 | `--radius-1..5`, `--border-size-1..5`, `--shadow-1..6`, `--inner-shadow-0..4` | シンプルな幾何値 |
| 動き | `--ease-in-1..5`, `--ease-out-1..5`, `--ease-in-out-1..5`, `--ease-spring-1..5`, `--ease-elastic-1..5` | easing 関数 |
| その他 | `--layer-1..5`, `--ratio-*` | z-index / aspect-ratio |

## ホワイトリスト外（Block 直参照禁止）

以下は Prism セマンティック層を必ず経由:

| OP raw | Prism 経由 |
|---|---|
| `--font-sans`, `--font-mono` | そのまま OK（`--font-display` は alias） |
| `--font-weight-1..9` | `--font-weight-regular\|medium\|bold` |
| `--font-size-0..8` | `--step-*`（Utopia 直）or `.u-text-*` クラス |
| `--font-lineheight-0..5` | `--leading-tight\|main\|loose` |
| `--font-letterspacing-0..7` | `--tracking-tight\|normal\|wide\|uppercase` |
| `--size-*`, `--size-fluid-*` | 使用しない（Utopia `--space-*` を代わりに） |

## 重要な勘違い: OP には `--space-*` はない

Open Props には `--size-*` と `--size-fluid-*` はあるが、`--space-*` は **Utopia 発**。Prism では Utopia の `--space-3xs..3xl` を基盤に使っているので、「`--space-l` は Open Props の変数ですか？」と聞かれたら **No**（Utopia 発）と答える。

## Cascade Layer 配置

```css
@layer openprops, utopia, prism.global, prism.tokens, ..., prism.blocks, prism.exceptions;

@import url("open-props/style.css") layer(openprops);
```

- `openprops` 層に OP 全体を流し込み、
- `utopia` 層に Utopia の `--step-*` / `--space-*` / pair を配置（OP より後勝ち）、
- `prism.tokens` 層で OP / Utopia の raw を **Prism セマンティック alias** に結び付ける。

この 3 段構造により、Block は `prism.tokens` 層が提供するセマンティック名だけで書ける。

## テーマ再定義での OP raw 参照

`[data-theme="dark"]` のような theme block で `--color-*` を OP raw に再結合する時は、OP raw 直参照が許容される:

```css
[data-theme="dark"] {
  --color-background: var(--gray-12);
  --color-ink: var(--gray-0);
  --color-button: var(--gray-0);
  --color-button-ink: var(--gray-12);
}
```

これは Block ではなく Tokens 層の再定義なので問題ない。

## JIT Props（推奨: 本番ビルドで採用検討）

OP は PostCSS プラグイン [JIT Props](https://github.com/argyleink/open-props#jit-props) を提供。本番ビルドで「実際に使われている変数だけ」を抽出してバンドルを削減できる。Prism は `package.json` にセットアップすると `main.css` の全 OP import をそのまま書いても、ビルド時にツリーシェイクされる。

## よく使う OP raw（チートシート）

```css
/* カラー（theme 再定義でのみ） */
var(--gray-0)   /* 白系 */
var(--gray-12)  /* 黒系 */

/* Shadow */
var(--shadow-2)  /* light: card 用 */
var(--shadow-4)  /* light: 強調用 */
var(--shadow-6)  /* light: hero overlay 用 */

/* Radius */
var(--radius-2)     /* s: button */
var(--radius-3)     /* m: card */
var(--radius-round) /* 円形 */

/* Border width */
var(--border-size-1)  /* main */
var(--border-size-2)  /* 強調 */

/* Ease */
var(--ease-in-out-3)  /* standard */
var(--ease-out-3)     /* decelerate */

/* Layer */
var(--layer-1)  /* base */
var(--layer-3)  /* nav */
var(--layer-5)  /* modal */
```

## 参考

- 原典: https://open-props.style/
- Prism tokens 層: `src/styles/prism/tokens.css`
