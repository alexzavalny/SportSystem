# Effort Tag Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Show a live-updating funny sassy tag in the active workout day's header based on how many sets are checked.

**Architecture:** Single `index.html` modification — add CSS pill styles, add `EFFORT_TIERS` data + two JS functions (`getEffortTag`, `updateEffortTag`), wire into existing DOMContentLoaded and checkbox change handlers. No tag on rest days (Day 0).

**Tech Stack:** Vanilla JS, inline CSS, no build step, `localStorage` already used for state.

---

### Task 1: Add CSS for effort tag pill

**Files:**
- Modify: `index.html` — `<style>` block, after the `.bonus-set` rule (around line 515)

- [ ] **Step 1: Add CSS**

In `index.html`, locate the `.bonus-set` rule:
```css
.bonus-set {
    opacity: 0.38;
}
```

Insert immediately after it:
```css
/* ── Effort tag ── */
.effort-tag {
    font-size: 0.65rem;
    font-weight: 700;
    letter-spacing: 0.06em;
    background: rgba(139, 148, 158, 0.1);
    color: var(--text-muted);
    border-radius: 4px;
    padding: 2px 7px;
    margin-left: auto;
    white-space: nowrap;
}

.effort-tag--done {
    background: var(--can-bg);
    color: var(--can);
    border: 1px solid var(--can-border);
    box-shadow: 0 0 8px var(--checked-glow);
}
```

- [ ] **Step 2: Verify CSS loads**

Serve: `python3 -m http.server 8080`
Open: `http://localhost:8080`
Expected: no visual change yet (class not applied), no console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style: add effort-tag pill CSS"
```

---

### Task 2: Add effort tier data and `getEffortTag` function

**Files:**
- Modify: `index.html` — `<script>` block, before the `dayOfYear` function (around line 850)

- [ ] **Step 1: Add tier table and function**

In `index.html`, locate `function dayOfYear(date) {` and insert immediately before it:

```javascript
const EFFORT_TIERS = [
    { min: 0,   max: 0,   en: "🦥 Lazy Sloth",          ru: "🦥 Диванный Боец" },
    { min: 1,   max: 20,  en: "🥱 Just Showed Up",       ru: "🥱 Зашёл Поглазеть" },
    { min: 21,  max: 50,  en: "🤷 Half-Hearted Hero",    ru: "🤷 Ну Типа Старался" },
    { min: 51,  max: 79,  en: "💦 Sweaty Mess",          ru: "💦 Уже Не Позор" },
    { min: 80,  max: 99,  en: "😤 One More Set, Coward", ru: "😤 Ещё Подход, Слабак" },
    { min: 100, max: 100, en: "🔥 Absolute Unit",        ru: "🔥 Красавчик" },
];

function getEffortTag(pct) {
    return EFFORT_TIERS.find(t => pct >= t.min && pct <= t.max);
}
```

- [ ] **Step 2: Verify in console**

Open browser console at `http://localhost:8080`.
Run: `getEffortTag(0)` → expect `{min:0, max:0, en:"🦥 Lazy Sloth", ru:"🦥 Диванный Боец"}`
Run: `getEffortTag(100)` → expect `{min:100, max:100, en:"🔥 Absolute Unit", ru:"🔥 Красавчик"}`
Run: `getEffortTag(50)` → expect `{min:21, max:50, en:"🤷 Half-Hearted Hero", ...}`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add effort tier data and getEffortTag function"
```

---

### Task 3: Add `updateEffortTag` function

**Files:**
- Modify: `index.html` — `<script>` block, after `getEffortTag` function

- [ ] **Step 1: Add function**

Immediately after the `getEffortTag` function, insert:

```javascript
function updateEffortTag() {
    const activeSection = document.querySelector('.day-section.active');
    if (!activeSection || activeSection.id === 'section-day0') return;

    const checkboxes = activeSection.querySelectorAll('input[type="checkbox"]');
    const total = checkboxes.length;
    if (total === 0) return;

    const checked = Array.from(checkboxes).filter(cb => cb.checked).length;
    const pct = Math.round((checked / total) * 100);
    const tier = getEffortTag(pct);

    const header = activeSection.querySelector('.day-header');
    let tag = header.querySelector('#effort-tag');
    if (!tag) {
        tag = document.createElement('span');
        tag.id = 'effort-tag';
        header.appendChild(tag);
    }

    tag.className = 'effort-tag' + (pct === 100 ? ' effort-tag--done' : '');
    tag.innerHTML = `<span class="language-en">${tier.en}</span><span class="language-ru">${tier.ru}</span>`;

    const lang = localStorage.getItem('selectedLanguage') || 'en';
    tag.querySelector('.language-en').style.display = lang === 'en' ? 'inline' : 'none';
    tag.querySelector('.language-ru').style.display = lang === 'ru' ? 'inline' : 'none';
}
```

- [ ] **Step 2: Verify in console**

In browser console: `updateEffortTag()`
Expected: tag appears in active day's header showing correct tier for current checked state.
Check DOM: `document.querySelector('#effort-tag').textContent` should show one of the tier strings.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add updateEffortTag function"
```

---

### Task 4: Wire `updateEffortTag` into page init and checkbox events

**Files:**
- Modify: `index.html` — `DOMContentLoaded` handler (around line 937)

- [ ] **Step 1: Call on page load**

Locate the `DOMContentLoaded` handler. Find this line:
```javascript
restoreCheckboxState();
```

Add `updateEffortTag();` immediately after it:
```javascript
restoreCheckboxState();
updateEffortTag();
```

- [ ] **Step 2: Call on checkbox change**

In the same `DOMContentLoaded` handler, find the checkbox event listener:
```javascript
checkbox.addEventListener("change", (e) => saveCheckboxState(e.target));
```

Replace with:
```javascript
checkbox.addEventListener("change", (e) => {
    saveCheckboxState(e.target);
    updateEffortTag();
});
```

- [ ] **Step 3: Verify live update**

In browser at `http://localhost:8080`:
- Active day section shows tag in header (e.g. "🦥 Lazy Sloth" if nothing checked)
- Check one set → tag updates immediately
- Check all sets → tag shows "🔥 Absolute Unit" with green glow
- Uncheck a set → tag reverts to previous tier
- Switch to RU language → tag shows Russian text
- Switch back to EN → tag shows English text
- Navigate to rest day (Day 0): no tag in header

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: wire effort tag into page init and checkbox events"
```
