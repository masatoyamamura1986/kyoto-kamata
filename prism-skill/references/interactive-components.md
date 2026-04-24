# Interactive Components

JS 駆動のインタラクティブコンポーネントの実装パターン。

## 基本原則

- **`[data-state]` ベース**で制御（`.is-active` は全廃）
- JS は `element.dataset.state = "active"` でトグル（`classList.toggle("is-*")` は全廃）
- **init guard** を必ず設置（同じコンポーネントが複数ある場合の二重初期化防止）
- **component-scoped**: `document.querySelector` ではなく `component.querySelector`
- DOM 生成は **clone パターン** — `innerHTML` / `createElement` / template literal HTML は禁止
- **要素識別は `data-role`** でクラス名と分離

## 共通テンプレート

```ts
document.addEventListener("DOMContentLoaded", () => {
  document.querySelectorAll<HTMLElement>(".accordion").forEach((component) => {
    if (component.dataset.scriptInitialized) return;
    component.dataset.scriptInitialized = "true";

    const items = component.querySelectorAll<HTMLElement>("[data-role='item']");
    items.forEach((item) => {
      const trigger = item.querySelector<HTMLButtonElement>("[data-role='trigger']");
      trigger?.addEventListener("click", () => {
        const isActive = item.dataset.state === "active";
        item.dataset.state = isActive ? "inactive" : "active";
        trigger.setAttribute("aria-expanded", String(!isActive));
      });
    });
  });
});
```

---

## Accordion

### HTML

```html
<div class="accordion" role="region" aria-label="FAQ">

  <div data-role="item" data-state="active">
    <button type="button" data-role="trigger"
      aria-expanded="true"
      aria-controls="faq-panel-1"
      data-trigger="hover focus">
      <span data-role="trigger-text">質問タイトル</span>
      <span data-role="icon" aria-hidden="true">
        <span data-role="icon-horizontal"></span>
        <span data-role="icon-vertical"></span>
      </span>
    </button>
    <div id="faq-panel-1" data-role="panel" role="region">
      <div>
        <div class="u-text-main" data-role="panel-content">
          回答本文...
        </div>
      </div>
    </div>
  </div>

  <!-- 他の item -->

</div>
```

### CSS

```css
.accordion [data-role="item"] {
  /* item 自体に境界線等 */
  border-block-end: var(--border-main) solid var(--color-border);
}

.accordion [data-role="trigger"] {
  display: flex;
  inline-size: 100%;
  justify-content: space-between;
  gap: var(--space-s);
  padding-block: var(--space-s);
  background: transparent;
  border: 0;
  cursor: pointer;
}

.accordion [data-role="icon"] {
  position: relative;
  inline-size: 1em;
  aspect-ratio: 1 / 1;
  flex-shrink: 0;
}

.accordion [data-role="icon-horizontal"],
.accordion [data-role="icon-vertical"] {
  position: absolute;
  inset: 50% 0 auto 0;
  block-size: 2px;
  background: currentColor;
  transform-origin: center;
}

.accordion [data-role="icon-vertical"] {
  transform: rotate(calc(90deg * var(--state-false)));
  transition: transform var(--duration-main) var(--ease-standard);
}

/* Panel 開閉アニメーション（grid-template-rows 0fr → 1fr） */
.accordion [data-role="panel"] {
  display: grid;
  grid-template-rows: calc(var(--state-true) * 1fr);
  transition: grid-template-rows var(--duration-main) var(--ease-standard);
}

.accordion [data-role="panel"] > div {
  overflow: hidden;
}

.accordion [data-role="panel-content"] {
  padding-block-end: var(--space-s);
}
```

**ポイント**:
- `[data-role="panel"]` を `display: grid` にして `grid-template-rows: 0fr` (close) → `1fr` (open) でアニメーション
- `var(--state-true)` は `[data-state="active"]` で 1、それ以外で 0 に自動切替（tokens.css で定義済み）
- 値の変換に `.is-active` は不要

### JS（キーボード対応込み）

```ts
document.addEventListener("DOMContentLoaded", () => {
  document.querySelectorAll<HTMLElement>(".accordion").forEach((component) => {
    if (component.dataset.scriptInitialized) return;
    component.dataset.scriptInitialized = "true";

    const items = Array.from(
      component.querySelectorAll<HTMLElement>("[data-role='item']")
    );

    items.forEach((item) => {
      const trigger = item.querySelector<HTMLButtonElement>("[data-role='trigger']");
      if (!trigger) return;

      trigger.addEventListener("click", () => {
        const isActive = item.dataset.state === "active";
        item.dataset.state = isActive ? "inactive" : "active";
        trigger.setAttribute("aria-expanded", String(!isActive));
      });

      trigger.addEventListener("keydown", (event) => {
        const i = items.indexOf(item);
        let target = -1;
        if (event.key === "ArrowDown") { event.preventDefault(); target = (i + 1) % items.length; }
        else if (event.key === "ArrowUp") { event.preventDefault(); target = (i - 1 + items.length) % items.length; }
        else if (event.key === "Home") { event.preventDefault(); target = 0; }
        else if (event.key === "End") { event.preventDefault(); target = items.length - 1; }
        if (target >= 0) items[target].querySelector<HTMLButtonElement>("[data-role='trigger']")?.focus();
      });
    });
  });
});
```

---

## Tabs

```html
<div class="tabs" role="tablist">
  <div data-role="tab-list">
    <button data-role="tab" data-state="active" aria-selected="true" aria-controls="tab-panel-1" id="tab-1">Tab 1</button>
    <button data-role="tab" aria-selected="false" aria-controls="tab-panel-2" id="tab-2">Tab 2</button>
  </div>
  <div data-role="panel" data-state="active" id="tab-panel-1" role="tabpanel" aria-labelledby="tab-1">Panel 1</div>
  <div data-role="panel" id="tab-panel-2" role="tabpanel" aria-labelledby="tab-2" hidden>Panel 2</div>
</div>
```

JS:
```ts
component.querySelectorAll<HTMLButtonElement>("[data-role='tab']").forEach((tab) => {
  tab.addEventListener("click", () => {
    // deactivate all
    component.querySelectorAll<HTMLElement>("[data-role='tab']").forEach(t => {
      t.dataset.state = "inactive";
      t.setAttribute("aria-selected", "false");
    });
    component.querySelectorAll<HTMLElement>("[data-role='panel']").forEach(p => {
      p.dataset.state = "inactive";
      p.hidden = true;
    });
    // activate selected
    tab.dataset.state = "active";
    tab.setAttribute("aria-selected", "true");
    const panelId = tab.getAttribute("aria-controls");
    const panel = component.querySelector<HTMLElement>(`#${panelId}`);
    if (panel) {
      panel.dataset.state = "active";
      panel.hidden = false;
    }
  });
});
```

---

## Modal (`<dialog>` ベース)

```html
<button data-role="trigger" aria-controls="my-modal">Open</button>
<dialog id="my-modal" class="modal" data-role="panel">
  <div data-role="content">
    ...
    <button data-role="close" aria-label="Close">✕</button>
  </div>
</dialog>
```

JS:
```ts
component.querySelectorAll<HTMLElement>("[data-role='trigger']").forEach((trigger) => {
  const target = trigger.getAttribute("aria-controls");
  const dialog = document.getElementById(target!) as HTMLDialogElement | null;
  if (!dialog) return;
  trigger.addEventListener("click", () => dialog.showModal());
  dialog.querySelector<HTMLButtonElement>("[data-role='close']")
    ?.addEventListener("click", () => dialog.close());
});
```

- `<dialog>` は open/close で自動的に `open` 属性が切り替わる。CSS 側は `dialog[open]` を利用
- ESC キー / バックドロップクリックも自動対応

---

## 共通ベストプラクティス

### ✅ DO

- `element.dataset.state = "..."` で状態遷移
- `component.querySelector` でスコープを限定
- `aria-expanded` / `aria-selected` を `data-state` と同期
- `_hidden` テンプレートの `cloneNode(true)` で要素複製

### ❌ DON'T

- `classList.toggle("is-active")`
- `document.querySelector` で他コンポーネントに干渉
- `innerHTML +=` / `createElement` で構造組立
- `window.innerWidth` で分岐（`getComputedStyle(document.documentElement).getPropertyValue('--responsive-l')` で CSS 変数を読む）

## 参考

- `anti-patterns-full.md` — 禁止パターン全リスト
- `naming-conventions.md` — `data-role` / `data-state` 命名ルール
