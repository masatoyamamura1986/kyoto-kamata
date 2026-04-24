# CUBE Methodology — Prism 拡張版（F+T+C-U-B-E）

## 出典

- Andy Bell: https://cube.fyi/
- 原典記事: https://piccalil.li/blog/cube-css/

## CUBE オリジナル 4 レイヤー

| レイヤー | 役割 |
|---|---|
| **Composition** | レイアウトの「形」。内部にコンテンツが流し込まれる器 |
| **Utility** | 単一関心事のクラス。CSS プロパティのエイリアス的役割 |
| **Block** | 固有コンポーネント（hero, card, nav 等）。子孫セレクタ許容 |
| **Exception** | Block のバリアント / 状態。`data-*` 属性で表現 |

## Prism 拡張（F+T を追加）

Prism は CUBE 原典の 4 レイヤーに **Foundation** と **Tokens** を加えた **5+1 レイヤー**構成:

| レイヤー | 役割 | Prism ファイル |
|---|---|---|
| **0. Foundation** | リセット、モーション基礎 | `src/styles/prism/global.css` |
| **1. Tokens** | 値のソース・オブ・トゥルース | `src/styles/prism/tokens.css` |
| **2. Composition** | `.l-*` レイアウト primitives | `src/styles/prism/compositions.css` |
| **3. Utility** | `.u-*` 単一目的クラス | `src/styles/prism/utilities.css` |
| **4. Block** | 各コンポーネントの `<style>` 内 | `sections/{category}/*.html` 等 |
| **5. Exception** | `data-*` attribute セレクタ | `src/styles/prism/exceptions.css` |

## Cascade Layers

CSS `@layer` で優先度を厳密に管理:

```css
/* src/styles/prism/main.css */
@layer
  openprops,
  utopia,
  prism.global,
  prism.tokens,
  prism.compositions,
  prism.utilities,
  prism.blocks,
  prism.exceptions;

@import url("open-props/style.css") layer(openprops);
@import url("./utopia.css") layer(utopia);
@import url("./global.css") layer(prism.global);
@import url("./tokens.css") layer(prism.tokens);
@import url("./compositions.css") layer(prism.compositions);
@import url("./utilities.css") layer(prism.utilities);
@import url("./exceptions.css") layer(prism.exceptions);
```

Block は `<style>` 内で `@layer prism.blocks { ... }` に所属させる慣習（Astro の scoped style では自動的に Block 相当の優先度になる）。

## レイヤー別の許容ルール

| 層 | 他層参照 | 子孫セレクタ | 備考 |
|---|---|---|---|
| Foundation | — | 要素セレクタのみ | normalize / reset |
| Tokens | OP raw → Prism 意味づけ | NG | 変数定義のみ |
| Composition | Tokens のみ | NG（単一ブロック内のみ） | 構造のみ、意味を持たない |
| Utility | Tokens のみ | NG | 1 クラス 1 責務 |
| Block | Tokens + Composition + Utility | 深さ 2 まで | 単純名、単数形 |
| Exception | 横断参照 OK | Block / Utility 上に重ねる | `data-*` 属性セレクタ |

## F+T を独立させた理由（Prism 独自判断）

- CUBE 原典は Tokens を「design tokens」として言及するが独立レイヤーは設けない
- Prism は Open Props + Utopia + セマンティック alias の 3 段を明示管理したいため、Tokens を独立レイヤー化
- Foundation も `global.css` として独立させ、`body` / `img` / `:focus-visible` 等のリセットを明示

## CUBE 原典との差分（プラン独自拡張の明示）

| 項目 | 原典 | Prism | 判定 |
|---|---|---|---|
| 4 レイヤー（C-U-B-E） | ✅ | ✅ | 準拠 |
| Foundation / Tokens 独立 | ❌ | ✅ | **拡張** |
| `.u-*` / `.l-*` 接頭辞 | ❌ 明示推奨なし（コミュニティ慣習） | ✅ | **慣習採用** |
| Block で BEM `__` | ✅ Andy 個人は継続 | ❌ 子孫セレクタ派 | **乖離**（CUBE「open season」範囲内） |
| `[data-state]` Exception | ✅ 明示例あり | ✅ | 準拠 |
| Cascade Layers | ❌（2020 年記事で未普及） | ✅ | **現代化拡張** |

## 参考

- `naming-conventions.md` — 命名規則の判断フロー
- `anti-patterns-full.md` — 各レイヤーの禁止事項
