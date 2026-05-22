# Hospital Emergency Priority System — Design Spec

**Date:** 2026-05-23  
**Status:** Approved

---

## Overview

A single `index.html` file implementing a Hospital Emergency Priority System. No frameworks, no libraries. Pure HTML, CSS, and vanilla JavaScript. Patients are prioritized by severity (10 = most critical); ties broken by arrival order (first-come, first-served).

---

## Data Layer

### `Patient`
Properties: `name` (string), `age` (number), `severity` (1–10 integer), `arrivalIndex` (monotonically increasing integer, assigned at insert time).

### `MaxHeap`
Array-backed binary max-heap. No external dependencies.

**Comparison rule:** Primary sort by `severity` descending; secondary sort by `arrivalIndex` ascending (lower index = higher priority among equals).

**Public API:**
- `insert(patient)` — push to array, sift up
- `extractMax()` — swap root↔last, pop, sift down, return extracted patient
- `peek()` — return root without mutation
- `toSortedArray()` — returns a sorted snapshot (copy of heap, repeated extractMax) without mutating the live heap
- `size()` — current patient count

### `HospitalQueue`
Thin controller wrapping `MaxHeap`. Owns `arrivalCounter` and `treatmentHistory[]`.

**Public API:**
- `addPatient(name, age, severity)` — creates `Patient`, inserts into heap
- `treatNext()` — calls `extractMax()`, appends result to history with timestamp, returns treated patient or null if empty
- `showQueue()` — returns `heap.toSortedArray()`

---

## UI Layout

Single page, responsive two-column desktop / stacked mobile.

### Left Panel — "Add Patient"
- Text input: patient name (required)
- Number input: age (1–120, required)
- Range slider: severity 1–10 with live numeric label
- Button: "Add Patient" (validates all fields before inserting)

### Right Panel
**Top:** "Treat Next Patient" button — red, disabled when queue is empty.

**Middle:** Live queue table, re-rendered after every add/treat.

| Rank | Name | Age | Severity | Status |
|------|------|-----|----------|--------|
| 1 | … | … | badge | Waiting |

Severity badge color ranges:
- 1–3: green
- 4–6: yellow/orange
- 7–8: orange
- 9–10: red

**Bottom:** Treatment history log — newest entry on top. Each entry shows timestamp, patient name, age, and severity.

---

## Behavior

| Event | Action |
|-------|--------|
| Page load | Insert 5 sample patients; render queue |
| Add Patient (valid) | Insert into heap; re-render table; clear form; show toast |
| Add Patient (invalid) | Show inline validation error; do not insert |
| Treat Next | Extract max; append to history; re-render table; show toast "Now treating: [name]" |
| Queue empty + Treat Next | Button disabled; toast if edge-case triggered |

History persists in memory for the session. Displayed in reverse-chronological order.

---

## Sample Patients (startup)

| Name | Age | Severity |
|------|-----|----------|
| Maria Santos | 67 | 9 |
| James Lee | 34 | 6 |
| Aisha Patel | 52 | 10 |
| Carlos Rivera | 78 | 4 |
| Emma Chen | 45 | 7 |

---

## Implementation Constraints

- Single file: `index.html`
- No external scripts, stylesheets, or fonts
- Classes: `Patient`, `MaxHeap`, `HospitalQueue`
- All logic in `<script>` block; all styles in `<style>` block
