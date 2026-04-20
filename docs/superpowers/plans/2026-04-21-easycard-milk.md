# 悠遊卡換鮮乳條碼產生器 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 建立單一 `index.html` 靜態網頁，讓家長輸入悠遊卡識別碼與16碼卡號後生成一維條碼，支援多筆幼兒資料儲存於 localStorage，部署至 GitHub Pages。

**Architecture:** 單一 `index.html`，純 Vanilla JS + 內嵌 CSS。三個 view（列表、表單、條碼）以 `display:none/block` 切換，無頁面跳轉。JS 分為四個邏輯段：Storage（localStorage CRUD）、Router（view 切換）、ProfileList（列表渲染）、BarcodeView（條碼生成）。

**Tech Stack:** HTML5, CSS3, Vanilla JS (ES6+), JsBarcode 3.x (CDN), crypto.randomUUID()

---

## File Structure

```
index.html          # 全部：HTML + 內嵌 <style> + 內嵌 <script>
.gitignore          # 排除 .superpowers/
```

`index.html` 內部結構：
- `<style>` — 全站 CSS（mobile-first，藍白主題）
- `<body>` — 3 個 `<div id="view-*">` 容器
  - `#view-list` — 首頁卡片列表
  - `#view-form` — 新增/編輯表單
  - `#view-barcode` — 條碼顯示
- `<script>` — 4 個邏輯段（Storage / Router / ProfileList / BarcodeView）+ 初始化

---

## Task 1: 專案基礎設定

**Files:**
- Create: `index.html`
- Create: `.gitignore`

- [ ] **Step 1: 建立 `.gitignore`**

```
.superpowers/
```

- [ ] **Step 2: 建立 `index.html` 基本骨架（含 CDN 與三個 view 容器）**

```html
<!DOCTYPE html>
<html lang="zh-Hant-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>悠遊卡換鮮乳條碼產生器</title>
  <script src="https://cdn.jsdelivr.net/npm/jsbarcode@3.11.6/dist/barcodes/JsBarcode.code128.min.js"></script>
  <style>
    /* placeholder — 下一個 task 填入 */
    body { margin: 0; font-family: sans-serif; }
  </style>
</head>
<body>
  <div id="view-list"></div>
  <div id="view-form" style="display:none"></div>
  <div id="view-barcode" style="display:none"></div>

  <script>
    // placeholder — 後續 task 填入
  </script>
</body>
</html>
```

- [ ] **Step 3: 在瀏覽器開啟 `index.html`，確認無 console error，頁面空白但正常載入**

- [ ] **Step 4: Commit**

```bash
git add index.html .gitignore
git commit -m "feat: project scaffold with three view containers"
```

---

## Task 2: CSS 樣式（藍白主題，手機優先）

**Files:**
- Modify: `index.html` — `<style>` 區塊

- [ ] **Step 1: 將 `<style>` 替換為完整 CSS**

```css
/* === Reset & Base === */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background: #f0f4ff;
  min-height: 100vh;
  color: #1a1a2e;
}

/* === Layout === */
.app-container {
  max-width: 480px;
  margin: 0 auto;
  padding: 16px;
  min-height: 100vh;
}

/* === Header === */
.app-header {
  background: #1a56db;
  color: white;
  border-radius: 12px;
  padding: 14px 16px;
  margin-bottom: 20px;
  font-size: 16px;
  font-weight: bold;
  text-align: center;
}

/* === Card List === */
.profile-card {
  background: white;
  border-radius: 10px;
  padding: 12px 14px;
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 1px 6px rgba(0,0,0,.08);
  user-select: none;
}
.profile-card .profile-info .profile-name {
  font-size: 15px;
  font-weight: 600;
}
.profile-card .profile-info .profile-type {
  font-size: 11px;
  color: #888;
  margin-top: 2px;
}
.btn-barcode {
  background: #1a56db;
  color: white;
  border: none;
  border-radius: 7px;
  padding: 7px 12px;
  font-size: 12px;
  cursor: pointer;
  white-space: nowrap;
}
.btn-barcode:active { background: #1344b0; }

/* === Add Button === */
.btn-add {
  width: 100%;
  border: 2px dashed #aac0f0;
  border-radius: 10px;
  padding: 12px;
  text-align: center;
  color: #1a56db;
  font-size: 13px;
  cursor: pointer;
  background: transparent;
  margin-top: 4px;
}
.btn-add:active { background: #e8eeff; }

/* === Empty State === */
.empty-state {
  text-align: center;
  padding: 40px 16px;
  color: #888;
}
.empty-state p { margin-bottom: 8px; font-size: 14px; }

/* === Form === */
.form-back {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #1a56db;
  font-size: 13px;
  cursor: pointer;
  margin-bottom: 16px;
}
.form-title { font-size: 15px; font-weight: bold; margin-bottom: 16px; }
.field-group { margin-bottom: 14px; }
.field-label {
  font-size: 11px;
  color: #555;
  margin-bottom: 4px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: .4px;
}
.field-input {
  width: 100%;
  border: 1.5px solid #d0d9f0;
  border-radius: 8px;
  padding: 9px 12px;
  font-size: 14px;
  outline: none;
  transition: border-color .2s;
}
.field-input:focus { border-color: #1a56db; }
.field-input.error { border-color: #e53e3e; }
.field-error { font-size: 11px; color: #e53e3e; margin-top: 3px; display: none; }
.field-error.visible { display: block; }
.field-hint { font-size: 10px; color: #888; margin-top: 3px; }

/* === Card Type Toggle === */
.type-toggle { display: flex; gap: 8px; }
.type-btn {
  flex: 1;
  border: 1.5px solid #d0d9f0;
  border-radius: 8px;
  padding: 9px;
  text-align: center;
  font-size: 12px;
  cursor: pointer;
  background: white;
  color: #555;
  transition: all .2s;
}
.type-btn.active {
  background: #1a56db;
  color: white;
  border-color: #1a56db;
  font-weight: 600;
}

/* === Love Card Hint === */
.love-card-hint {
  background: #fffbeb;
  border: 1px solid #f59e0b;
  border-radius: 8px;
  padding: 10px 12px;
  font-size: 11px;
  color: #92400e;
  margin-bottom: 14px;
  display: none;
}
.love-card-hint.visible { display: block; }
.love-card-hint a { color: #b45309; font-weight: 600; }

/* === How-To Expand === */
.howto-toggle {
  font-size: 11px;
  color: #1a56db;
  cursor: pointer;
  margin-bottom: 14px;
  text-decoration: underline;
}
.howto-content {
  background: #eef2ff;
  border-radius: 8px;
  padding: 10px 12px;
  font-size: 11px;
  color: #374151;
  margin-bottom: 14px;
  display: none;
  line-height: 1.6;
}
.howto-content.visible { display: block; }

/* === Submit Button === */
.btn-submit {
  width: 100%;
  background: #1a56db;
  color: white;
  border: none;
  border-radius: 10px;
  padding: 13px;
  font-size: 15px;
  font-weight: bold;
  cursor: pointer;
  margin-top: 4px;
}
.btn-submit:active { background: #1344b0; }

/* === Barcode View === */
.barcode-back {
  font-size: 13px;
  color: #1a56db;
  cursor: pointer;
  margin-bottom: 16px;
}
.barcode-name {
  font-size: 22px;
  font-weight: bold;
  color: #1a56db;
  text-align: center;
  margin-bottom: 4px;
}
.barcode-type-label {
  text-align: center;
  font-size: 11px;
  color: #888;
  margin-bottom: 20px;
}
.barcode-wrap {
  background: white;
  border-radius: 12px;
  padding: 20px 12px;
  margin-bottom: 12px;
  box-shadow: 0 1px 6px rgba(0,0,0,.08);
  text-align: center;
}
.barcode-wrap svg { width: 100%; height: auto; }
.barcode-number {
  font-size: 13px;
  letter-spacing: 2px;
  color: #444;
  margin-top: 8px;
}
.barcode-fallback {
  background: #fffbeb;
  border: 1px solid #f59e0b;
  border-radius: 8px;
  padding: 12px 14px;
  font-size: 12px;
  color: #92400e;
  margin-bottom: 12px;
}
.barcode-fallback .fallback-title { font-weight: bold; margin-bottom: 6px; }
.barcode-fallback .fallback-row { margin-bottom: 3px; }
.barcode-screenshot-hint {
  text-align: center;
  font-size: 10px;
  color: #aaa;
  margin-top: 8px;
}

/* === Context Menu (long press) === */
.context-menu {
  position: fixed;
  background: white;
  border-radius: 10px;
  box-shadow: 0 4px 20px rgba(0,0,0,.15);
  overflow: hidden;
  z-index: 100;
  min-width: 140px;
  display: none;
}
.context-menu.visible { display: block; }
.context-menu-item {
  padding: 12px 16px;
  font-size: 14px;
  cursor: pointer;
}
.context-menu-item:hover { background: #f5f5f5; }
.context-menu-item.danger { color: #e53e3e; }
.context-overlay {
  position: fixed;
  inset: 0;
  z-index: 99;
  display: none;
}
.context-overlay.visible { display: block; }

/* === LocalStorage Warning === */
.storage-warning {
  background: #fef2f2;
  border: 1px solid #fca5a5;
  border-radius: 8px;
  padding: 10px 12px;
  font-size: 11px;
  color: #991b1b;
  margin-bottom: 14px;
  display: none;
}
.storage-warning.visible { display: block; }
```

- [ ] **Step 2: 在瀏覽器重新整理，確認頁面無 CSS 錯誤，背景為 `#f0f4ff`**

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add complete CSS theme (blue/white, mobile-first)"
```

---

## Task 3: Storage 層（localStorage CRUD）

**Files:**
- Modify: `index.html` — `<script>` 區塊，新增 Storage 段

- [ ] **Step 1: 在 `<script>` 內加入 Storage 物件**

```js
// ===== STORAGE =====
const Storage = (() => {
  const KEY = 'easycard_profiles';

  function isAvailable() {
    try {
      localStorage.setItem('__test__', '1');
      localStorage.removeItem('__test__');
      return true;
    } catch { return false; }
  }

  function getAll() {
    if (!isAvailable()) return [];
    try {
      return JSON.parse(localStorage.getItem(KEY) || '[]');
    } catch { return []; }
  }

  function save(profiles) {
    if (!isAvailable()) return false;
    localStorage.setItem(KEY, JSON.stringify(profiles));
    return true;
  }

  function add(profile) {
    const profiles = getAll();
    profiles.push(profile);
    return save(profiles);
  }

  function update(id, patch) {
    const profiles = getAll().map(p => p.id === id ? { ...p, ...patch } : p);
    return save(profiles);
  }

  function remove(id) {
    return save(getAll().filter(p => p.id !== id));
  }

  function createProfile(name, type, identifier, cardNumber) {
    return {
      id: crypto.randomUUID(),
      name: name.trim(),
      type,             // 'general' | 'love'
      identifier: identifier.trim(),
      cardNumber: cardNumber.replace(/\s/g, ''),
      createdAt: new Date().toISOString()
    };
  }

  return { isAvailable, getAll, add, update, remove, createProfile };
})();
```

- [ ] **Step 2: 在瀏覽器 console 手動測試 Storage**

```js
// 在 DevTools console 執行：
const p = Storage.createProfile('小明', 'general', 'A123', '1234567890123456');
Storage.add(p);
console.log(Storage.getAll()); // 應看到一筆資料
Storage.remove(p.id);
console.log(Storage.getAll()); // 應為空陣列
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add localStorage storage layer with CRUD"
```

---

## Task 4: Router（View 切換）

**Files:**
- Modify: `index.html` — `<script>` 區塊，新增 Router 段

- [ ] **Step 1: 在 Storage 段之後加入 Router**

```js
// ===== ROUTER =====
const Router = (() => {
  const views = {
    list: document.getElementById('view-list'),
    form: document.getElementById('view-form'),
    barcode: document.getElementById('view-barcode')
  };

  function show(name) {
    Object.entries(views).forEach(([key, el]) => {
      el.style.display = key === name ? 'block' : 'none';
    });
  }

  return { show };
})();
```

- [ ] **Step 2: 在 console 測試切換**

```js
Router.show('form');   // view-form 顯示，其他隱藏
Router.show('list');   // view-list 顯示
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add view router"
```

---

## Task 5: ProfileList（首頁卡片列表）

**Files:**
- Modify: `index.html` — HTML `#view-list` 容器、`<script>` 新增 ProfileList 段

- [ ] **Step 1: 設定 `#view-list` HTML 骨架**

```html
<div id="view-list">
  <div class="app-container">
    <div class="app-header">🥛 悠遊卡換鮮乳條碼產生器</div>
    <div id="storage-warning" class="storage-warning">
      ⚠️ 此瀏覽器不支援本機儲存，資料將在關閉頁面後消失。
    </div>
    <div id="profiles-container"></div>
    <button class="btn-add" id="btn-add-profile">＋ 新增孩子</button>
  </div>
</div>

<!-- Context Menu -->
<div class="context-overlay" id="context-overlay"></div>
<div class="context-menu" id="context-menu">
  <div class="context-menu-item" id="ctx-edit">✏️ 編輯</div>
  <div class="context-menu-item danger" id="ctx-delete">🗑 刪除</div>
</div>
```

- [ ] **Step 2: 在 Router 段之後加入 ProfileList**

```js
// ===== PROFILE LIST =====
const ProfileList = (() => {
  let contextTargetId = null;
  let longPressTimer = null;

  function render() {
    const profiles = Storage.getAll();
    const container = document.getElementById('profiles-container');

    if (!Storage.isAvailable()) {
      document.getElementById('storage-warning').classList.add('visible');
    }

    if (profiles.length === 0) {
      container.innerHTML = `
        <div class="empty-state">
          <p>還沒有儲存任何孩子的資料</p>
          <p>點下方按鈕新增第一筆</p>
        </div>`;
      return;
    }

    container.innerHTML = profiles.map(p => `
      <div class="profile-card" data-id="${p.id}">
        <div class="profile-info">
          <div class="profile-name">${escapeHtml(p.name)}</div>
          <div class="profile-type">${p.type === 'general' ? '一般悠遊卡' : '兒童優惠卡（愛心卡）'}</div>
        </div>
        <button class="btn-barcode" data-id="${p.id}">顯示條碼</button>
      </div>`).join('');

    // Barcode buttons
    container.querySelectorAll('.btn-barcode').forEach(btn => {
      btn.addEventListener('click', e => {
        e.stopPropagation();
        const profile = Storage.getAll().find(p => p.id === btn.dataset.id);
        if (profile) BarcodeView.show(profile);
      });
    });

    // Long press for edit/delete
    container.querySelectorAll('.profile-card').forEach(card => {
      card.addEventListener('pointerdown', () => {
        longPressTimer = setTimeout(() => showContextMenu(card.dataset.id, card), 600);
      });
      card.addEventListener('pointerup', () => clearTimeout(longPressTimer));
      card.addEventListener('pointercancel', () => clearTimeout(longPressTimer));
      card.addEventListener('pointermove', () => clearTimeout(longPressTimer));
    });
  }

  function showContextMenu(id, card) {
    contextTargetId = id;
    const menu = document.getElementById('context-menu');
    const overlay = document.getElementById('context-overlay');
    const rect = card.getBoundingClientRect();
    menu.style.top = `${rect.bottom + 4}px`;
    menu.style.left = `${rect.left}px`;
    menu.classList.add('visible');
    overlay.classList.add('visible');
  }

  function hideContextMenu() {
    document.getElementById('context-menu').classList.remove('visible');
    document.getElementById('context-overlay').classList.remove('visible');
    contextTargetId = null;
  }

  function init() {
    document.getElementById('btn-add-profile').addEventListener('click', () => {
      ProfileForm.open(null);
    });

    document.getElementById('context-overlay').addEventListener('click', hideContextMenu);

    document.getElementById('ctx-edit').addEventListener('click', () => {
      const profile = Storage.getAll().find(p => p.id === contextTargetId);
      hideContextMenu();
      if (profile) ProfileForm.open(profile);
    });

    document.getElementById('ctx-delete').addEventListener('click', () => {
      if (contextTargetId && confirm('確定要刪除這筆資料？')) {
        Storage.remove(contextTargetId);
        hideContextMenu();
        render();
      }
    });
  }

  return { render, init };
})();

// Helper
function escapeHtml(str) {
  return str.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}
```

- [ ] **Step 3: 加入初始化呼叫（在 `</script>` 前）**

```js
// ===== INIT =====
ProfileList.init();
Router.show('list');
ProfileList.render();
```

- [ ] **Step 4: 在瀏覽器驗證**
  - 首頁顯示空狀態提示
  - DevTools console 中手動 `Storage.add(Storage.createProfile('測試', 'general', 'A1', '1234567890123456'))`，然後重新整理，確認卡片出現
  - 長按卡片 600ms 後出現右鍵選單
  - 點「刪除」→ 確認 → 卡片消失

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add profile list view with long-press context menu"
```

---

## Task 6: ProfileForm（新增 / 編輯表單）

**Files:**
- Modify: `index.html` — HTML `#view-form` 容器、`<script>` 新增 ProfileForm 段

- [ ] **Step 1: 設定 `#view-form` HTML 骨架**

```html
<div id="view-form" style="display:none">
  <div class="app-container">
    <div class="form-back" id="form-back">← 返回</div>
    <div class="form-title" id="form-title">新增孩子資料</div>

    <div class="field-group">
      <div class="field-label">孩子姓名</div>
      <input class="field-input" id="f-name" type="text" placeholder="例：小明" autocomplete="off">
      <div class="field-error" id="err-name">請填入孩子姓名</div>
    </div>

    <div class="field-group">
      <div class="field-label">卡片類型</div>
      <div class="type-toggle">
        <div class="type-btn active" data-type="general" id="type-general">一般悠遊卡</div>
        <div class="type-btn" data-type="love" id="type-love">兒童優惠卡（愛心卡）</div>
      </div>
    </div>

    <div class="love-card-hint" id="love-card-hint">
      ⚠️ 兒童優惠卡正面無識別碼，請先至官方平台查詢：<br>
      <a href="https://mepweb.easycard.com.tw/portal/about" target="_blank" rel="noopener">
        前往悠遊卡多元兌換平台查詢識別碼 →
      </a><br>
      （選縣市＋行政區，輸入16碼卡號，即可找到識別碼）
    </div>

    <div class="field-group">
      <div class="field-label">識別碼（卡片正面）</div>
      <input class="field-input" id="f-identifier" type="text" placeholder="例：A12345678" autocomplete="off">
      <div class="field-error" id="err-identifier">請填入識別碼（愛心卡請至官網查詢）</div>
    </div>

    <div class="field-group">
      <div class="field-label">16碼卡號（卡片背面）</div>
      <input class="field-input" id="f-card-number" type="text" inputmode="numeric" placeholder="1234 5678 9012 3456" autocomplete="off">
      <div class="field-error" id="err-card-number">卡號必須為16位數字</div>
    </div>

    <div class="howto-toggle" id="howto-toggle">📖 如何找到這兩個號碼？</div>
    <div class="howto-content" id="howto-content">
      <strong>識別碼</strong>：卡片正面右上角或左下角（幼兒數位卡在左下方）<br>
      <strong>16碼卡號</strong>：卡片背面中央印刷的數字
    </div>

    <button class="btn-submit" id="btn-submit-form">儲存並生成條碼</button>
  </div>
</div>
```

- [ ] **Step 2: 在 ProfileList 段之後加入 ProfileForm**

```js
// ===== PROFILE FORM =====
const ProfileForm = (() => {
  let editingId = null;
  let currentType = 'general';

  function open(profile) {
    editingId = profile ? profile.id : null;
    currentType = profile ? profile.type : 'general';

    document.getElementById('form-title').textContent = profile ? '編輯孩子資料' : '新增孩子資料';
    document.getElementById('f-name').value = profile ? profile.name : '';
    document.getElementById('f-identifier').value = profile ? profile.identifier : '';
    document.getElementById('f-card-number').value = profile ? profile.cardNumber : '';

    setType(currentType);
    clearErrors();
    Router.show('form');
  }

  function setType(type) {
    currentType = type;
    document.getElementById('type-general').classList.toggle('active', type === 'general');
    document.getElementById('type-love').classList.toggle('active', type === 'love');
    document.getElementById('love-card-hint').classList.toggle('visible', type === 'love');
  }

  function clearErrors() {
    ['name', 'identifier', 'card-number'].forEach(field => {
      document.getElementById(`f-${field}`).classList.remove('error');
      document.getElementById(`err-${field}`).classList.remove('visible');
    });
  }

  function showError(field, msg) {
    const input = document.getElementById(`f-${field}`);
    const err = document.getElementById(`err-${field}`);
    input.classList.add('error');
    if (msg) err.textContent = msg;
    err.classList.add('visible');
  }

  function validate() {
    let valid = true;
    const name = document.getElementById('f-name').value.trim();
    const identifier = document.getElementById('f-identifier').value.trim();
    const cardRaw = document.getElementById('f-card-number').value.replace(/\s/g, '');

    if (!name) { showError('name'); valid = false; }
    if (!identifier) { showError('identifier'); valid = false; }
    if (!/^\d{16}$/.test(cardRaw)) { showError('card-number'); valid = false; }
    return valid;
  }

  function submit() {
    if (!validate()) return;

    const name = document.getElementById('f-name').value.trim();
    const identifier = document.getElementById('f-identifier').value.trim();
    const cardNumber = document.getElementById('f-card-number').value.replace(/\s/g, '');

    if (editingId) {
      Storage.update(editingId, { name, type: currentType, identifier, cardNumber });
      const profile = Storage.getAll().find(p => p.id === editingId);
      ProfileList.render();
      BarcodeView.show(profile);
    } else {
      const profile = Storage.createProfile(name, currentType, identifier, cardNumber);
      Storage.add(profile);
      ProfileList.render();
      BarcodeView.show(profile);
    }
  }

  function init() {
    document.getElementById('form-back').addEventListener('click', () => {
      Router.show('list');
    });

    document.getElementById('type-general').addEventListener('click', () => setType('general'));
    document.getElementById('type-love').addEventListener('click', () => setType('love'));

    document.getElementById('howto-toggle').addEventListener('click', () => {
      document.getElementById('howto-content').classList.toggle('visible');
    });

    document.getElementById('btn-submit-form').addEventListener('click', submit);
  }

  return { open, init };
})();
```

- [ ] **Step 3: 在 INIT 段加入 `ProfileForm.init()`（在 `ProfileList.init()` 之後）**

```js
ProfileList.init();
ProfileForm.init();
Router.show('list');
ProfileList.render();
```

- [ ] **Step 4: 在瀏覽器驗證**
  - 點「＋ 新增孩子」→ 表單出現
  - 選「兒童優惠卡」→ 黃色提示框出現，官網連結可點
  - 點「如何找到這兩個號碼？」→ 展開說明
  - 空白送出 → 三個欄位均顯示紅色錯誤提示
  - 卡號填 15 碼 → 顯示「卡號必須為16位數字」
  - 正確填寫後點「儲存並生成條碼」→ 不出錯（BarcodeView 尚未完成，會 console error，正常）

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add profile add/edit form with validation"
```

---

## Task 7: BarcodeView（條碼顯示）

**Files:**
- Modify: `index.html` — HTML `#view-barcode` 容器、`<script>` 新增 BarcodeView 段

- [ ] **Step 1: 設定 `#view-barcode` HTML 骨架**

```html
<div id="view-barcode" style="display:none">
  <div class="app-container">
    <div class="barcode-back" id="barcode-back">← 返回列表</div>
    <div class="barcode-name" id="barcode-name"></div>
    <div class="barcode-type-label" id="barcode-type-label"></div>
    <div class="barcode-wrap">
      <svg id="barcode-svg"></svg>
      <div class="barcode-number" id="barcode-number"></div>
    </div>
    <div class="barcode-fallback" id="barcode-fallback">
      <div class="fallback-title">⚠️ 如掃碼失敗，請店員手動輸入：</div>
      <div class="fallback-row">識別碼：<strong id="fallback-identifier"></strong></div>
      <div class="fallback-row">卡號：<strong id="fallback-card-number"></strong></div>
    </div>
    <div id="barcode-error" style="display:none;color:#e53e3e;text-align:center;font-size:13px;padding:16px">
      條碼無法生成，請確認網路連線後重新整理。
    </div>
    <div class="barcode-screenshot-hint">請截圖保存此頁面</div>
  </div>
</div>
```

- [ ] **Step 2: 在 ProfileForm 段之後加入 BarcodeView**

```js
// ===== BARCODE VIEW =====
const BarcodeView = (() => {
  function show(profile) {
    document.getElementById('barcode-name').textContent = profile.name;
    document.getElementById('barcode-type-label').textContent =
      profile.type === 'general' ? '一般悠遊卡' : '兒童優惠卡（愛心卡）';
    document.getElementById('barcode-number').textContent = profile.cardNumber;
    document.getElementById('fallback-identifier').textContent = profile.identifier;
    document.getElementById('fallback-card-number').textContent = profile.cardNumber;

    const svg = document.getElementById('barcode-svg');
    const errEl = document.getElementById('barcode-error');

    try {
      JsBarcode(svg, profile.cardNumber, {
        format: 'CODE128',
        displayValue: false,
        width: 2,
        height: 100,
        margin: 10
      });
      svg.style.display = 'block';
      errEl.style.display = 'none';
    } catch (e) {
      svg.style.display = 'none';
      errEl.style.display = 'block';
    }

    Router.show('barcode');
  }

  function init() {
    document.getElementById('barcode-back').addEventListener('click', () => {
      Router.show('list');
    });
  }

  return { show, init };
})();
```

- [ ] **Step 3: 在 INIT 段加入 `BarcodeView.init()`**

```js
ProfileList.init();
ProfileForm.init();
BarcodeView.init();
Router.show('list');
ProfileList.render();
```

- [ ] **Step 4: 在瀏覽器完整流程測試**
  - 新增一筆一般悠遊卡（姓名：測試、識別碼：A123、卡號：1234567890123456）→ 應跳到條碼頁
  - 確認條碼 SVG 正確顯示（黑白條紋）
  - 確認下方數字、手動輸入備用區塊均顯示正確資料
  - 點「← 返回列表」→ 首頁出現剛才新增的卡片
  - 點卡片上「顯示條碼」→ 再次進入條碼頁
  - 長按卡片 → 選「刪除」→ 確認 → 卡片消失
  - 長按卡片 → 選「編輯」→ 表單帶入既有資料 → 修改姓名 → 儲存 → 條碼頁顯示新姓名

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add barcode display view with JsBarcode Code128"
```

---

## Task 8: 最終收尾與 GitHub Pages 設定

**Files:**
- Modify: `index.html` — meta tags、title
- Create: `README.md` (minimal)

- [ ] **Step 1: 補強 `<head>` meta tags（加在 viewport 之後）**

```html
<meta name="description" content="悠遊卡換鮮乳條碼產生器 — 忘記帶卡時，用識別碼+卡號生成超商掃碼條碼">
<meta name="theme-color" content="#1a56db">
<meta property="og:title" content="悠遊卡換鮮乳條碼產生器">
<meta property="og:description" content="忘記帶悠遊卡？輸入識別碼與卡號，立即生成超商可掃描的條碼">
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🥛</text></svg>">
```

- [ ] **Step 2: 在瀏覽器驗證 meta（DevTools → Elements 查看 head）**

- [ ] **Step 3: 建立 README.md**

```markdown
# 悠遊卡換鮮乳條碼產生器

台北市/新北市「生生喝鮮乳」政策輔助工具。忘記帶悠遊卡時，輸入識別碼與16碼卡號，生成超商可掃描的一維條碼。

## 功能
- 輸入識別碼 + 16碼卡號 → 生成 Code 128 條碼
- 支援多筆幼兒資料儲存（localStorage）
- 兒童優惠卡（愛心卡）引導至官方查詢平台
- 純靜態，無後端，可離線使用

## 使用方式
直接開啟 GitHub Pages 網址即可。

## 注意事項
- 資料儲存於瀏覽器 localStorage，換裝置需重新輸入
- 條碼格式為 Code 128，編碼內容為16碼卡號
- 如超商掃碼失敗，畫面會顯示識別碼與卡號供手動輸入
```

- [ ] **Step 4: 手機瀏覽器測試（用本機 IP 或 ngrok）**
  - 確認在手機上點擊流暢
  - 確認條碼在手機螢幕上清晰可見
  - 確認截圖後條碼仍可辨識

- [ ] **Step 5: Commit**

```bash
git add index.html README.md
git commit -m "feat: add meta tags and README, ready for GitHub Pages"
```

- [ ] **Step 6: 建立 GitHub repo 並 push**

```bash
# 在 GitHub 建立新 repo（名稱建議：easycard-milk）後：
git remote add origin https://github.com/<your-username>/easycard-milk.git
git push -u origin main
```

- [ ] **Step 7: 啟用 GitHub Pages**

GitHub repo → Settings → Pages → Source: Deploy from branch → Branch: `main` / `/ (root)` → Save

等待約 1 分鐘後，開啟 `https://<your-username>.github.io/easycard-milk/` 驗證正常顯示。
