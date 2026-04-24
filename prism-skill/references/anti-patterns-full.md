# Anti-patterns（Prism 完全リスト）

Prism Design System の禁止事項。SKILL.md の 17 項目の禁止リストを起点に、JS / CSS specificity / アクセシビリティ領域を加えた完全リファレンス（全 29 項目）。

## カテゴリ

1. 命名 / 構造（#1–#8）
2. Tokens / 変数参照（#9–#17、うち **#17 は raw rhythm token 禁止で最重要**）
3. JavaScript（#18–#24）
4. CSS specificity / inline（#25–#26）
5. アクセシビリティ（#27–#29）

---

## 1. 命名 / 構造

### #1. `_wrap` / `_layout` / `_content` などアンダースコア命名（旧 Lumos 流）

❌ `<section class="hero_wrap u-section">`
✅ `<section class="hero u-region"><div class="u-wrapper"><div class="l-stack">...`

### #2. Utility に `.u-*` 接頭辞を付けない

❌ `.visually-hidden`
✅ `.u-visually-hidden`

### #3. Composition に `.l-*` 接頭辞を付けない

❌ `.stack`, `.cluster`
✅ `.l-stack`, `.l-cluster`

### #4. Block に `.u-*` / `.l-*` / `.c-*` 接頭辞を付ける

❌ `.u-card`, `.c-hero`, `.l-nav`
✅ `.card`, `.hero`, `.nav`

### #5. Composition class を単独で意味付与に使う

Composition は「構造の形」だけ。意味を持たせるのは Block の仕事。

❌ `.l-stack { color: red; font-size: 2rem; }`
✅ Composition は layout 専用。色・タイポは Block で指定。

### #6. Composition 内で子孫セレクタ

❌ `.l-stack > h2 { ... }`
✅ `.hero h2 { ... }`（Block 側）

### #7. 3 段以上の子孫セレクタ

❌ `.card .wrapper .l-stack h2 { ... }`
✅ 2 段まで: `.card h2 { ... }` または `.card .card-action { ... }`

### #8. Block 内に別 Block を入れ子にして同名要素混在

❌ `.hero` の中に `.card` を入れて `.hero h2` と `.card h2` が競合
✅ セクション内の派生要素は `.hero-offset` のように派生名で独立

---

## 2. Tokens / 変数参照

### #9. Exception を `.is-*` クラスで表現

❌ `.is-reversed`, `.is-active`, `.is-primary`
✅ `[data-variant="reversed"]`, `[data-state="active"]`, `[data-variant="primary"]`

### #10. 動的状態を `.is-active` で表現

❌ `element.classList.toggle("is-active")`
✅ `element.dataset.state = "active"`

### #11. `classList.toggle("is-*")`

❌ `classList.add("is-open")` / `classList.remove("is-open")`
✅ `element.dataset.state = "open"` / `"closed"`

### #12. OP 変数をホワイトリスト外で Block 直接参照

❌ `.button { color: var(--gray-0); font-weight: var(--font-weight-7); }`
✅ `.button { color: var(--color-button-ink); font-weight: var(--font-weight-bold); }`

許可リスト（Block 直接 OK）: `--gray-*`（theme 再定義のみ）, `--space-*`, `--step-*`, `--radius-*`, `--shadow-*`, `--ease-*`, `--ratio-*`, `--layer-*`, `--border-size-*`。詳細は `open-props-integration.md`。

### #13. Utopia `--step-*` を幅（max-width / grid-template-columns）に使う

❌ `grid-template-columns: var(--step-4) 1fr;` / `max-width: var(--step-7);`
✅ `grid-template-columns: var(--space-2xl) 1fr;` / `max-width: var(--max-width-m);`

### #14. `.l-stack` と `.u-flow` を同じ要素に付ける

❌ `<div class="l-stack u-flow">`
✅ どちらか片方。`.l-stack` は flex gap、`.u-flow` は lobotomized owl（margin）で競合。

### #15. `.l-switcher --switcher-threshold` を `vw` で書く

❌ `--switcher-threshold: 50vw;`（viewport 依存でブレる）
✅ `--switcher-threshold: 50rem;`（コンテナ幅に応じて切替）

### #16. `--_theme---*` / `--_spacing---*` アンダースコア複合命名で新規変数作成

❌ `--_theme---button-primary--background`
✅ `--color-button`（Prism セマンティック命名）

---

## 🎯 #17. raw `--space-xs/l/2xl` をセクション rhythm に直接使う（**最重要**）

このパターンはセクション間で余白がバラバラになる最大の原因。

❌ **NG**:
```html
<div class="l-stack" style="--stack-space: var(--space-xs);">      <!-- tagline → h2 -->
<div class="l-stack" style="--stack-space: var(--space-l);">       <!-- description → cluster -->
<div class="l-stack" style="--stack-space: var(--space-2xl);">     <!-- heading → body -->
```

✅ **OK**:
```html
<div class="l-stack" style="--stack-space: var(--space-heading-group);">
<div class="l-stack" style="--stack-space: var(--space-body-group);">
<div class="l-stack" style="--stack-space: var(--space-heading-to-body);">
```

### Rhythm tokens の 5 バケット

| Token | 用途 |
|---|---|
| `--space-heading-group` | tagline → h1/h2/h3 |
| `--space-heading-to-body` | h2 → description（Pattern B の見出し ⇄ ボディ） |
| `--space-body-group` | description → cluster → accordion |
| `--space-body-to-images` | body group → 画像ブロック |
| `--space-images` | image ↔ image |

### 判断: rhythm か？非 rhythm か？

**section rhythm（セマンティック token 必須）**:
- `.u-region > .u-wrapper > .l-stack` に直接現れる見出し ⇄ 本文 ⇄ 画像のブロック間ギャップ
- `.l-stack --stack-space` / `.l-switcher --switcher-space` / `gallery-layout gap` がセクションレベルで機能する場合
- セクション間の余白 / セクション全体のリズム

**Block 内部 rhythm（raw token OK）**:
- card / modal / nav 内部の「独自レジスター」を持つ stack
  - 例: card の `[data-role="content"]` 内 tagline → h3 → details を `--space-s` / `--space-m` でタイトに刻む
- section rhythm と独立した Block 固有の縦並び

**非 rhythm 余白（raw token OK）**:
- button の padding / icon との gap
- card / box の内側 padding
- icon と text の間
- 例: `.button { padding-inline: var(--space-l); gap: var(--space-xs); }` は OK

判定法: 「この余白を調整した時、**他のセクション**の視覚リズムに影響するか？」
- YES → section rhythm → rhythm tokens 必須
- NO （Block 内部 or 装飾に閉じる） → raw OK

### なぜ重要か

rhythm token で書いておけば、値を調整したい時に `tokens.css` の 1 行を変えるだけで全セクションが同期する。raw token を使うと、各セクションを個別に調整する羽目になり、必ずどこかで漏れてバラバラになる。

---

## 3. JavaScript

### #18. `var` / `any` 型の使用

❌ `var x: any = ...`
✅ `const x: string = ...` / 正確な型指定

### #19. `id` による DOM 選択（ARIA 用途以外）

❌ `document.getElementById("my-accordion")`
✅ `component.querySelector('[data-role="trigger"]')`
（ARIA の `aria-controls` / `aria-labelledby` で id を参照する場合は OK）

### #20. `innerHTML +=` / template literal で視覚構造生成

❌ `container.innerHTML += '<div class="item">...</div>';`
✅ `_hidden` テンプレートから `const clone = template.cloneNode(true); container.append(clone);`

### #21. `window.innerWidth` で判定

❌ `if (window.innerWidth < 768) { ... }`
✅ CSS 側で `--responsive-m` フラグを参照、JS からは `getComputedStyle(document.documentElement).getPropertyValue('--responsive-m')` を読む

### #22. `console.log` / `.env` をコミット

❌ production に `console.log` 残る
✅ `import.meta.env.DEV && console.log(...)` ガード、`.env` は gitignore

### #23. init guard なしのコンポーネント初期化

複数インスタンスで二重初期化が走る。

❌:
```ts
document.querySelectorAll('.accordion').forEach((component) => {
  // init 毎回実行
});
```
✅:
```ts
document.querySelectorAll<HTMLElement>('.accordion').forEach((component) => {
  if (component.dataset.scriptInitialized) return;
  component.dataset.scriptInitialized = 'true';
  // init
});
```

### #24. `document.querySelector` で他コンポーネントに干渉

❌ `document.querySelector('[data-role="trigger"]')`（全ページで最初の 1 つを選ぶ）
✅ `component.querySelector('[data-role="trigger"]')`（component-scoped）

---

## 4. CSS specificity / inline

### #25. Block CSS で `!important` を使う

❌ `.hero h2 { font-size: 4rem !important; }`
✅ Cascade Layers の順序に任せる（Prism は `@layer` で優先度管理済み）

### #26. inline style にハードコード色・px

❌ `<div style="color: #333; padding: 16px;">`
✅ `<div style="color: var(--color-ink); padding: var(--space-s);">`

---

## 5. アクセシビリティ

### #27. `<button>` に `data-role="trigger"` 忘れて `<div>` でボタン化

❌ `<div data-role="trigger" onclick="...">Click</div>`
✅ `<button type="button" data-role="trigger">Click</button>`（キーボード操作 + focus 自動対応）

### #28. `aria-expanded` / `aria-selected` を `data-state` と同期しない

❌ `data-state="active"` のみ更新して `aria-expanded` を放置
✅ 両方を同時に更新:
```ts
item.dataset.state = isActive ? 'inactive' : 'active';
trigger.setAttribute('aria-expanded', String(!isActive));
```

### #29. 画像の alt を空にする（装飾以外で）

❌ `<img src="hero.webp">`
✅ 意味のある画像: `<img src="hero.webp" alt="デザイナー山村が作業中">`
装飾画像: `<img src="decorative.svg" alt="">`

---

## 統一チェックリスト

新規セクションを書く前/後で以下を確認:

- [ ] Block 名は接頭辞なし、単数形（`.hero`, `.card`）
- [ ] `.u-region` + `.u-wrapper` でセクションを包んでいる
- [ ] `.l-*` を構造に、`.u-*` を単一目的に使っている
- [ ] 見出しグループと本文グループを別の `.l-stack` に分けている
- [ ] **rhythm tokens** を使っている（raw `--space-*` を rhythm に使っていない）
- [ ] 動的状態は `[data-state]` + `dataset.state`
- [ ] 要素識別は `data-role`
- [ ] init guard がある
- [ ] `@media` は SP と Desktop で意味が変わる例外的ケースのみ
- [ ] button / card の padding に raw `--space-*` を使っているのは OK

## 参考

- `naming-conventions.md` — 命名ルール
- `vertical-rhythm.md` — rhythm tokens 詳細
- `interactive-components.md` — data-state / data-role 実装
