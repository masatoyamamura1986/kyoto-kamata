# Every Layout Primitives（`.l-*`）

## 出典

https://every-layout.dev/layouts/

Prism は Every Layout の 13 primitives を `.l-*` 接頭辞で実装。Composition レイヤーに所属し、各 primitive には **ローカル CSS 変数**で挙動を調整するパラメータがある。

## 13 Primitives 早見表

| Primitive | 主ローカル変数 | デフォルト | 用途 |
|---|---|---|---|
| `.l-stack` | `--stack-space` | `var(--space-l)` | ブロック集合の縦並び（flex gap） |
| `.l-cluster` | `--cluster-space`, `--cluster-justify`, `--cluster-align` | `space-s`, `flex-start`, `center` | インライン群（折り返し可）、タグ・ボタン群 |
| `.l-switcher` | `--switcher-threshold`, `--switcher-space`, `--switcher-limit` | `30rem`, `space-m`, — | 閾値で 1 行 ↔ 縦スタック切替 |
| `.l-sidebar` | `--sidebar-side`, `--sidebar-content-min`, `--sidebar-space` | `20rem`, `50%`, `space-m` | サイドバー + 本文 |
| `.l-grid` | `--grid-min`, `--grid-space` | `20rem`, `space-m` | auto-fit minmax |
| `.l-frame` | `--frame-ratio` | `16/9` | aspect-ratio + object-fit |
| `.l-cover` | `--cover-min-height`, `--cover-space` | `100svh`, `space-m` | 縦フル + 中央寄せ要素 |
| `.l-box` | `--box-padding`, `--box-border`, `--box-radius` | `space-m`, `border-main solid color-border`, `radius-m` | padded container |
| `.l-center` | `--center-max`, `--center-gutters` | `max-width-s`, `0` | 中央寄せ + max-width |
| `.l-container` | — | — | Container Query スコープ（`.u-wrapper` なしで CQ のみ欲しい時） |
| `.l-reel` | `--reel-height`, `--reel-space`, `--reel-item-width` | `auto`, `space-s`, `auto` | 横スクロール |
| `.l-icon` | `--icon-size`, `--icon-space` | `1em`, `0.5em` | インライン SVG + text との整列 |
| `.l-imposter` | `--imposter-margin`, `[data-fixed]`, `[data-contain]` | — | オーバーレイ |

## Stack 拡張バリアント

- **`.l-stack-recursive`** — stack を入れ子のすべての要素に再帰適用。全階層で同じ `--stack-space` を使える。
- **`.l-stack[data-split-after="N"]`** — N 番目の子のあとに `margin-block-end: auto` を付与。残りを下に押し出して「header 上、footer 下」のようなレイアウトを作れる。N は 1-8 が実装済み。

```html
<div class="l-stack" data-split-after="2" style="min-block-size: 100vh;">
  <header>...</header>       <!-- 1st -->
  <main>...</main>            <!-- 2nd（この後に auto margin） -->
  <footer>...</footer>        <!-- bottom pinned -->
</div>
```

## 使い分け

### `.l-stack` vs `.u-flow`

| ケース | 選ぶべき |
|---|---|
| ブロック集合（見出し / 段落 / 画像 / ボタン群が混ざる） | `.l-stack` |
| 純粋なテキスト流し（記事本文の `<p>` 連続） | `.u-flow` |

`.l-stack` は flex gap でスペースを均等化（margin collapse なし）。`.u-flow` は lobotomized owl（`> * + *`）で margin-block-start を付与。

**同じ要素に両方付けるのは NG**（flex だと margin が効かない）。

### `.l-switcher` の挙動

- `--switcher-threshold` 以上の幅 → 横 1 行に並ぶ
- それ未満 → 縦スタック
- `--switcher-limit` で「N 個超えたら強制縦スタック」を指定可

```html
<div class="l-switcher" style="--switcher-threshold: 50rem; --switcher-space: var(--space-heading-to-body);">
  <div>Left (heading group)</div>
  <div>Right (body group)</div>
</div>
```

### `.l-grid` vs `.l-switcher`

| ケース | 選ぶべき |
|---|---|
| カード一覧など「等幅の繰り返し」 | `.l-grid` |
| 2 カラム「閾値で切替」 | `.l-switcher` |
| サイドバー + 本文の「比率が違う 2 カラム」 | `.l-sidebar` |

## Block からの変数注入

Block CSS の `<style>` で Composition のローカル変数を上書きできる:

```html
<style>
.hero .l-cluster {
  --cluster-space: var(--space-s);
}
.hero .l-stack {
  --stack-space: var(--space-body-group);
}
</style>
```

または inline で:

```html
<div class="l-stack" style="--stack-space: var(--space-heading-group);">...</div>
```

## よくある間違い

### ❌ NG: Composition 内に子孫セレクタ

```css
.l-stack > h2 { font-size: ...; }  /* ❌ Composition に意味を付与しない */
```

→ Block 側で `.hero h2 { ... }` と書く。

### ❌ NG: `.l-switcher --switcher-threshold` を `vw` で指定

```css
--switcher-threshold: 50vw;  /* ❌ viewport 依存でブレる */
--switcher-threshold: 50rem; /* ✅ コンテナ幅に応じて切替 */
```

### ❌ NG: `.l-stack` を画像グリッドに使う

画像が縦に並ぶときは OK、横に並べたいなら `.l-grid` / `.l-switcher` を使う。

### ❌ NG: `.l-frame` に flex / gap を追加

`.l-frame` は aspect-ratio + object-fit に特化。子要素（img/video）が絶対位置的に張り付く設計なので、flex レイアウトを重ねない。

## Every Layout 原典との差分

| 項目 | 原典 | Prism |
|---|---|---|
| プリミティブ数 | 13（Container 含む） | 13（Container は `.l-container`、`.u-wrapper` は Container + max-width + padding-inline 統合版） |
| Stack の実装 | lobotomized owl | flex gap（`.l-stack`）+ lobotomized owl（`.u-flow`）両方提供 |
| CSS 変数名 | `--s1` 等の短名 | `--stack-space` 等の明瞭名 |
| Web Components（`<stack-l>` 等） | あり | 採用せず（CSS クラスのみ） |
| splitAfter | あり | `.l-stack[data-split-after="1..8"]` として実装済 |
| Recursive Stack | あり | `.l-stack-recursive` として実装済 |

## 参考

- 原典: https://every-layout.dev/layouts/
- 実装: `src/styles/prism/compositions.css`
