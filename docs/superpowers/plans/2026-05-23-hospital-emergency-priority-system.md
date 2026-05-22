# Hospital Emergency Priority System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` file implementing a hospital emergency triage queue with a MaxHeap priority queue, patient form, live queue table, and treatment history log.

**Architecture:** Three JavaScript classes (`Patient`, `MaxHeap`, `HospitalQueue`) handle all data logic. A `UI` object owns all DOM operations and wires events. Everything lives in one `index.html` — styles in `<style>`, logic in `<script>`.

**Tech Stack:** Vanilla HTML5, CSS3, ES6+ JavaScript. No frameworks, no libraries, no build tools.

---

## File Structure

| File | Responsibility |
|------|---------------|
| `index.html` | Single deliverable — markup, styles, and all JS classes |

All tasks modify only `index.html`.

---

### Task 1: HTML Skeleton + CSS Reset

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create `index.html` with full document structure**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Hospital Emergency Priority System</title>
  <style>
    /* styles go here in Task 2 */
  </style>
</head>
<body>

  <!-- LEFT PANEL -->
  <aside class="panel panel--left">
    <h2 class="panel__title">Add Patient</h2>
    <form id="patientForm" novalidate>
      <div class="field">
        <label for="name">Patient Name</label>
        <input id="name" type="text" placeholder="Full name" autocomplete="off" />
        <span class="field__error" id="nameError"></span>
      </div>
      <div class="field">
        <label for="age">Age</label>
        <input id="age" type="number" min="1" max="120" placeholder="1 – 120" />
        <span class="field__error" id="ageError"></span>
      </div>
      <div class="field">
        <label for="severity">Severity: <strong id="severityLabel">5</strong> / 10</label>
        <input id="severity" type="range" min="1" max="10" value="5" />
      </div>
      <button type="submit" class="btn btn--add">Add Patient</button>
    </form>
  </aside>

  <!-- RIGHT PANEL -->
  <main class="panel panel--right">
    <div class="right-top">
      <h1 class="app-title">Emergency Priority Queue</h1>
      <button id="treatBtn" class="btn btn--treat">Treat Next Patient</button>
    </div>

    <section class="queue-section">
      <h2 class="section-title">Current Queue</h2>
      <div class="table-wrap">
        <table id="queueTable">
          <thead>
            <tr>
              <th>#</th>
              <th>Name</th>
              <th>Age</th>
              <th>Severity</th>
              <th>Arrival</th>
            </tr>
          </thead>
          <tbody id="queueBody"></tbody>
        </table>
        <p id="emptyMsg" class="empty-msg">No patients in queue.</p>
      </div>
    </section>

    <section class="history-section">
      <h2 class="section-title">Treatment History</h2>
      <ul id="historyList" class="history-list">
        <li class="history-empty">No patients treated yet.</li>
      </ul>
    </section>
  </main>

  <!-- TOAST -->
  <div id="toast" class="toast" aria-live="polite"></div>

  <script>
    /* JS goes here in Tasks 2–5 */
  </script>
</body>
</html>
```

- [ ] **Step 2: Open `index.html` in a browser and confirm the raw structure renders — two panels visible, form fields present, table present.**

---

### Task 2: CSS — Layout and Design System

**Files:**
- Modify: `index.html` — replace `/* styles go here in Task 2 */` with the full stylesheet below

- [ ] **Step 1: Replace the empty style comment with this complete CSS**

```css
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --navy: #0f172a;
      --navy-mid: #1e293b;
      --navy-light: #334155;
      --accent: #38bdf8;
      --accent-dark: #0284c7;
      --red: #ef4444;
      --red-dark: #b91c1c;
      --text: #f1f5f9;
      --text-muted: #94a3b8;
      --border: #334155;
      --green: #22c55e;
      --yellow: #eab308;
      --orange: #f97316;
      --radius: 10px;
    }

    body {
      display: flex;
      min-height: 100vh;
      font-family: system-ui, -apple-system, sans-serif;
      background: var(--navy);
      color: var(--text);
    }

    /* PANELS */
    .panel { padding: 28px 24px; }
    .panel--left {
      width: 320px;
      min-width: 280px;
      background: var(--navy-mid);
      border-right: 1px solid var(--border);
      display: flex;
      flex-direction: column;
      gap: 20px;
    }
    .panel--right {
      flex: 1;
      display: flex;
      flex-direction: column;
      gap: 24px;
      overflow-y: auto;
    }
    .panel__title {
      font-size: 1.1rem;
      font-weight: 700;
      color: var(--accent);
      letter-spacing: .03em;
    }

    /* FORM */
    form { display: flex; flex-direction: column; gap: 16px; }
    .field { display: flex; flex-direction: column; gap: 6px; }
    label { font-size: .85rem; color: var(--text-muted); font-weight: 500; }
    label strong { color: var(--accent); font-size: 1rem; }
    input[type="text"],
    input[type="number"] {
      background: var(--navy-light);
      border: 1px solid var(--border);
      border-radius: 6px;
      padding: 9px 12px;
      color: var(--text);
      font-size: .95rem;
      outline: none;
      transition: border-color .15s;
    }
    input[type="text"]:focus,
    input[type="number"]:focus { border-color: var(--accent); }
    input[type="range"] {
      width: 100%;
      accent-color: var(--accent);
      cursor: pointer;
    }
    .field__error { font-size: .78rem; color: var(--red); min-height: 16px; }

    /* BUTTONS */
    .btn {
      border: none;
      border-radius: var(--radius);
      padding: 11px 20px;
      font-size: .95rem;
      font-weight: 600;
      cursor: pointer;
      transition: background .15s, transform .08s, opacity .15s;
    }
    .btn:active { transform: scale(.97); }
    .btn:disabled { opacity: .45; cursor: not-allowed; transform: none; }
    .btn--add { background: var(--accent); color: var(--navy); }
    .btn--add:hover:not(:disabled) { background: var(--accent-dark); color: #fff; }
    .btn--treat { background: var(--red); color: #fff; }
    .btn--treat:hover:not(:disabled) { background: var(--red-dark); }

    /* RIGHT TOP */
    .right-top {
      display: flex;
      align-items: center;
      justify-content: space-between;
      flex-wrap: wrap;
      gap: 12px;
      padding-bottom: 16px;
      border-bottom: 1px solid var(--border);
    }
    .app-title { font-size: 1.4rem; font-weight: 800; }

    /* SECTION TITLES */
    .section-title {
      font-size: .95rem;
      font-weight: 700;
      color: var(--text-muted);
      text-transform: uppercase;
      letter-spacing: .08em;
      margin-bottom: 12px;
    }

    /* TABLE */
    .table-wrap { overflow-x: auto; }
    table { width: 100%; border-collapse: collapse; font-size: .9rem; }
    th {
      text-align: left;
      padding: 8px 12px;
      background: var(--navy-mid);
      color: var(--text-muted);
      font-size: .78rem;
      text-transform: uppercase;
      letter-spacing: .06em;
      border-bottom: 1px solid var(--border);
    }
    td { padding: 10px 12px; border-bottom: 1px solid var(--border); }
    tbody tr:hover { background: var(--navy-mid); }
    tbody tr:first-child td { background: rgba(239,68,68,.08); }

    .badge {
      display: inline-block;
      padding: 2px 10px;
      border-radius: 20px;
      font-size: .8rem;
      font-weight: 700;
      color: #fff;
    }
    .badge--low    { background: #16a34a; }
    .badge--mid    { background: #ca8a04; }
    .badge--high   { background: #ea580c; }
    .badge--crit   { background: #dc2626; }

    .rank-1 { font-weight: 800; color: var(--red); }
    .empty-msg { color: var(--text-muted); font-size: .9rem; padding: 16px 0; }

    /* HISTORY */
    .history-list { list-style: none; display: flex; flex-direction: column; gap: 8px; }
    .history-item {
      background: var(--navy-mid);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      padding: 10px 14px;
      font-size: .88rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 12px;
      animation: slideIn .25s ease;
    }
    .history-item__name { font-weight: 700; }
    .history-item__meta { color: var(--text-muted); font-size: .8rem; }
    .history-item__time { color: var(--text-muted); font-size: .78rem; white-space: nowrap; }
    .history-empty { color: var(--text-muted); font-size: .88rem; }

    /* TOAST */
    .toast {
      position: fixed;
      bottom: 28px;
      right: 28px;
      background: var(--navy-light);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      padding: 12px 20px;
      font-size: .9rem;
      font-weight: 500;
      color: var(--text);
      opacity: 0;
      transform: translateY(10px);
      transition: opacity .25s, transform .25s;
      pointer-events: none;
      max-width: 340px;
      z-index: 100;
    }
    .toast.show { opacity: 1; transform: translateY(0); }

    @keyframes slideIn {
      from { opacity: 0; transform: translateY(-6px); }
      to   { opacity: 1; transform: translateY(0); }
    }

    @media (max-width: 700px) {
      body { flex-direction: column; }
      .panel--left { width: 100%; border-right: none; border-bottom: 1px solid var(--border); }
    }
```

- [ ] **Step 2: Reload browser — confirm two-panel layout, dark navy theme, styled form, styled buttons, styled table headers.**

---

### Task 3: `Patient` and `MaxHeap` Classes

**Files:**
- Modify: `index.html` — replace `/* JS goes here in Tasks 2–5 */` with the classes below

- [ ] **Step 1: Add the `Patient` class**

```javascript
    class Patient {
      constructor(name, age, severity, arrivalIndex) {
        this.name = name;
        this.age = age;
        this.severity = severity;           // 1–10, higher = more critical
        this.arrivalIndex = arrivalIndex;   // lower = arrived earlier
      }
    }
```

- [ ] **Step 2: Add the `MaxHeap` class directly after `Patient`**

```javascript
    class MaxHeap {
      constructor() {
        this._heap = [];
      }

      size() { return this._heap.length; }
      peek() { return this._heap[0] ?? null; }

      // Higher severity wins. Equal severity: lower arrivalIndex wins.
      _compare(a, b) {
        if (b.severity !== a.severity) return b.severity - a.severity; // want max severity at root
        return a.arrivalIndex - b.arrivalIndex;                         // want min arrivalIndex at root
      }

      // Returns true if heap[i] should be above heap[j]
      _higher(i, j) {
        return this._compare(this._heap[i], this._heap[j]) < 0;
      }

      _swap(i, j) {
        [this._heap[i], this._heap[j]] = [this._heap[j], this._heap[i]];
      }

      _siftUp(i) {
        while (i > 0) {
          const parent = Math.floor((i - 1) / 2);
          if (this._higher(i, parent)) {
            this._swap(i, parent);
            i = parent;
          } else break;
        }
      }

      _siftDown(i) {
        const n = this._heap.length;
        while (true) {
          let top = i;
          const l = 2 * i + 1, r = 2 * i + 2;
          if (l < n && this._higher(l, top)) top = l;
          if (r < n && this._higher(r, top)) top = r;
          if (top === i) break;
          this._swap(i, top);
          i = top;
        }
      }

      insert(patient) {
        this._heap.push(patient);
        this._siftUp(this._heap.length - 1);
      }

      extractMax() {
        if (this._heap.length === 0) return null;
        this._swap(0, this._heap.length - 1);
        const max = this._heap.pop();
        this._siftDown(0);
        return max;
      }

      // Returns sorted snapshot without mutating the live heap.
      toSortedArray() {
        const copy = new MaxHeap();
        copy._heap = [...this._heap];
        const result = [];
        while (copy.size() > 0) result.push(copy.extractMax());
        return result;
      }
    }
```

- [ ] **Step 3: Open browser console and run a quick smoke test to verify the heap**

```javascript
// Paste into browser console:
const h = new MaxHeap();
h.insert(new Patient('A', 30, 5, 1));
h.insert(new Patient('B', 40, 9, 2));
h.insert(new Patient('C', 50, 9, 3));
console.log(h.peek().name);        // → "B"  (severity 9, earlier arrival)
console.log(h.extractMax().name);  // → "B"
console.log(h.extractMax().name);  // → "C"
console.log(h.extractMax().name);  // → "A"
console.log(h.extractMax());       // → null
```

Expected: `B`, `B`, `C`, `A`, `null` — in that order.

---

### Task 4: `HospitalQueue` Class + 5 Sample Patients

**Files:**
- Modify: `index.html` — add `HospitalQueue` after `MaxHeap`, then instantiate with sample data

- [ ] **Step 1: Add the `HospitalQueue` class**

```javascript
    class HospitalQueue {
      constructor() {
        this._heap = new MaxHeap();
        this._arrivalCounter = 0;
        this.treatmentHistory = [];   // [{patient, treatedAt}]
      }

      addPatient(name, age, severity) {
        const patient = new Patient(name, age, severity, this._arrivalCounter++);
        this._heap.insert(patient);
      }

      treatNext() {
        const patient = this._heap.extractMax();
        if (!patient) return null;
        this.treatmentHistory.unshift({ patient, treatedAt: new Date() });
        return patient;
      }

      showQueue() {
        return this._heap.toSortedArray();
      }

      size() {
        return this._heap.size();
      }
    }
```

- [ ] **Step 2: Instantiate the queue and insert the 5 sample patients**

```javascript
    const queue = new HospitalQueue();

    const samplePatients = [
      { name: 'Maria Santos',  age: 67, severity: 9  },
      { name: 'James Lee',     age: 34, severity: 6  },
      { name: 'Aisha Patel',   age: 52, severity: 10 },
      { name: 'Carlos Rivera', age: 78, severity: 4  },
      { name: 'Emma Chen',     age: 45, severity: 7  },
    ];
    samplePatients.forEach(p => queue.addPatient(p.name, p.age, p.severity));
```

- [ ] **Step 3: Console smoke test**

```javascript
// Paste into browser console after reload:
console.log(queue.showQueue().map(p => `${p.name} (${p.severity})`));
// Expected order: Aisha Patel (10), Maria Santos (9), Emma Chen (7), James Lee (6), Carlos Rivera (4)
```

---

### Task 5: UI Controller — Render + Events

**Files:**
- Modify: `index.html` — add the `UI` object and `init()` call after the queue instantiation

- [ ] **Step 1: Add severity badge helper and toast helper**

```javascript
    function severityBadge(s) {
      let cls = 'badge--low';
      if (s >= 9) cls = 'badge--crit';
      else if (s >= 7) cls = 'badge--high';
      else if (s >= 4) cls = 'badge--mid';
      return `<span class="badge ${cls}">${s}</span>`;
    }

    let _toastTimer = null;
    function showToast(msg) {
      const el = document.getElementById('toast');
      el.textContent = msg;
      el.classList.add('show');
      clearTimeout(_toastTimer);
      _toastTimer = setTimeout(() => el.classList.remove('show'), 3000);
    }
```

- [ ] **Step 2: Add `renderQueue()` function**

```javascript
    function renderQueue() {
      const patients = queue.showQueue();
      const tbody = document.getElementById('queueBody');
      const emptyMsg = document.getElementById('emptyMsg');
      const treatBtn = document.getElementById('treatBtn');

      treatBtn.disabled = patients.length === 0;
      emptyMsg.style.display = patients.length === 0 ? 'block' : 'none';

      tbody.innerHTML = patients.map((p, i) => `
        <tr>
          <td class="${i === 0 ? 'rank-1' : ''}">${i + 1}</td>
          <td>${escHtml(p.name)}</td>
          <td>${p.age}</td>
          <td>${severityBadge(p.severity)}</td>
          <td>#${p.arrivalIndex + 1}</td>
        </tr>
      `).join('');
    }

    function escHtml(str) {
      return str.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
    }
```

- [ ] **Step 3: Add `renderHistory()` function**

```javascript
    function renderHistory() {
      const list = document.getElementById('historyList');
      if (queue.treatmentHistory.length === 0) {
        list.innerHTML = '<li class="history-empty">No patients treated yet.</li>';
        return;
      }
      list.innerHTML = queue.treatmentHistory.map(({ patient: p, treatedAt }) => `
        <li class="history-item">
          <div>
            <div class="history-item__name">${escHtml(p.name)}</div>
            <div class="history-item__meta">Age ${p.age} &nbsp;|&nbsp; ${severityBadge(p.severity)}</div>
          </div>
          <div class="history-item__time">${treatedAt.toLocaleTimeString()}</div>
        </li>
      `).join('');
    }
```

- [ ] **Step 4: Add form validation and "Add Patient" submit handler**

```javascript
    document.getElementById('severity').addEventListener('input', e => {
      document.getElementById('severityLabel').textContent = e.target.value;
    });

    document.getElementById('patientForm').addEventListener('submit', e => {
      e.preventDefault();
      const nameEl     = document.getElementById('name');
      const ageEl      = document.getElementById('age');
      const severityEl = document.getElementById('severity');
      const nameErr    = document.getElementById('nameError');
      const ageErr     = document.getElementById('ageError');

      let valid = true;
      nameErr.textContent = '';
      ageErr.textContent  = '';

      const name = nameEl.value.trim();
      const age  = parseInt(ageEl.value, 10);

      if (!name) { nameErr.textContent = 'Name is required.'; valid = false; }
      if (!ageEl.value || isNaN(age) || age < 1 || age > 120) {
        ageErr.textContent = 'Age must be between 1 and 120.'; valid = false;
      }
      if (!valid) return;

      const severity = parseInt(severityEl.value, 10);
      queue.addPatient(name, age, severity);
      renderQueue();
      showToast(`Added: ${name} (severity ${severity})`);

      nameEl.value = '';
      ageEl.value  = '';
      severityEl.value = 5;
      document.getElementById('severityLabel').textContent = '5';
    });
```

- [ ] **Step 5: Add "Treat Next Patient" click handler**

```javascript
    document.getElementById('treatBtn').addEventListener('click', () => {
      const patient = queue.treatNext();
      if (!patient) return;
      renderQueue();
      renderHistory();
      showToast(`Now treating: ${patient.name}`);
    });
```

- [ ] **Step 6: Call initial render on page load**

```javascript
    renderQueue();
    renderHistory();
```

- [ ] **Step 7: Reload browser — verify full app works end to end:**
  - Queue shows 5 patients sorted Aisha (10) → Maria (9) → Emma (7) → James (6) → Carlos (4)
  - Severity slider label updates live as you drag
  - Add a new patient → appears in correct position in queue
  - Submit with empty fields → inline validation errors appear, no patient added
  - Click "Treat Next Patient" → top patient disappears from queue, appears in history with timestamp
  - Treat all patients → button becomes disabled, "No patients in queue." message appears
  - Toast notifications appear and fade after 3 seconds

---

### Task 6: Final Polish Pass

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add `<meta>` description and favicon emoji for browser tab polish**

In the `<head>`, after the `<title>` tag, add:
```html
  <meta name="description" content="Hospital Emergency Priority System — triage queue powered by a MaxHeap." />
  <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🏥</text></svg>" />
```

- [ ] **Step 2: Verify the full checklist one more time in the browser:**
  - [ ] MaxHeap correctly orders by severity desc, arrival asc on ties
  - [ ] "Treat Next" button is disabled when queue is empty
  - [ ] History log is newest-first
  - [ ] Severity badges use correct colors (1–3 green, 4–6 yellow, 7–8 orange, 9–10 red)
  - [ ] Form clears after successful submit
  - [ ] Responsive layout stacks on narrow viewport (resize window below 700px)

---

## Self-Review Checklist

**Spec coverage:**
| Requirement | Covered in |
|-------------|-----------|
| `Patient` class with name, age, severity | Task 3 Step 1 |
| `MaxHeap` from scratch | Task 3 Step 2 |
| `addPatient()` | Task 4 Step 1 |
| `treatNext()` | Task 4 Step 1 |
| `showQueue()` | Task 4 Step 1 |
| Form (name, age, severity) | Task 1 + Task 5 Step 4 |
| "Treat Next Patient" button | Task 1 + Task 5 Step 5 |
| Live queue table sorted by severity | Task 5 Step 2 |
| Treatment history log | Task 5 Step 3 |
| 5 sample patients on startup | Task 4 Step 2 |
| Single index.html, no frameworks | All tasks |
| Tie-breaking: first-come first-served | Task 3 Step 2 (`_compare`) |

All requirements covered. No gaps.

**Type consistency:** `Patient(name, age, severity, arrivalIndex)` used identically in Tasks 3 and 4. `MaxHeap` methods (`insert`, `extractMax`, `toSortedArray`, `size`, `peek`) referenced consistently across Tasks 3, 4, and 5. `HospitalQueue` methods (`addPatient`, `treatNext`, `showQueue`, `size`) used identically in Tasks 4 and 5.

**No placeholders:** All steps contain complete code. No TBD, TODO, or "similar to above" references.
