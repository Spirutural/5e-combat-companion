# Talent Tracker Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build `talent-tracker.html` — a combat tracker for The Talent (MCDM psionic class) branched from `tracker.html`, replacing spellcasting with a full psionics system.

**Architecture:** Single-file HTML app. Copy `tracker.html`, strip spell slots / prepared spells, add strain tracker + powers system. State managed with `update(mutatorFn)` → save → `render()`. Powers library is a hardcoded JS constant (not in localStorage).

**Tech Stack:** Vanilla HTML/CSS/JS. No build tools. No test framework — verify each step by running `start talent-tracker.html` in the project folder and checking in browser.

**Design doc:** `docs/plans/2026-02-27-talent-tracker-design.md`

---

## Task 1: Create Branch and Base File

**Files:**
- Create: `talent-tracker.html` (copy of `tracker.html`)

**Step 1: Create the git branch**
```bash
cd C:\Users\archa\Desktop\Claude\5e-combat-companion
git checkout -b talent-tracker
```
Expected: `Switched to a new branch 'talent-tracker'`

**Step 2: Copy the base file**
```bash
cp tracker.html talent-tracker.html
```

**Step 3: Update the page title**
In `talent-tracker.html`, find line 6:
```html
  <title>Material Companion</title>
```
Change to:
```html
  <title>Talent Companion</title>
```

**Step 4: Update the storage key**
Find (around line 1550):
```js
const STORAGE_KEY = '5e_tracker_v1';
```
Change to:
```js
const STORAGE_KEY = 'talent_tracker_v1';
```

**Step 5: Verify in browser**
```
start talent-tracker.html
```
Expected: Tracker loads normally, title reads "Talent Companion".

**Step 6: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: scaffold talent-tracker branch from base tracker"
```

---

## Task 2: Add New State Fields

**Files:**
- Modify: `talent-tracker.html` — DEFAULTS block (~line 1487) and migration guards (~line 1574)

**Step 1: Add fields to DEFAULTS**

In the DEFAULTS object, find the line `activeSpells: [],` (~line 1492). Add these fields directly after the `activeSpells` line:
```js
    strain: { body: 0, mind: 0, soul: 0 },
    strainMax: 16,
    manifestDie: 'd6',
    knownPowers: [],
    concentration: [],
    strainOpen: true,
    powersOpen: true,
```

**Step 2: Remove spell-specific DEFAULTS fields**

In DEFAULTS, remove these lines (they'll be replaced by psionics equivalents):
```js
    slots: [4, 3, 3, 3, 3, 2, 2, 1, 1],
    spellcastingAbility: 'wis',
    spellDCOverride: null,
    spellAtkOverride: null,
    slotsVisible: true,
    spellsOpen: true,
    preparedSpells: [],
    preparedSpellsOpen: true,
```

**Step 3: Add migration guards**

After the existing migration guards block (~line 1625), add:
```js
  if (!state.strain) state.strain = { body: 0, mind: 0, soul: 0 };
  if (state.strainMax === undefined) state.strainMax = 16;
  if (!state.manifestDie) state.manifestDie = 'd6';
  if (!state.knownPowers) state.knownPowers = [];
  if (!state.concentration) state.concentration = [];
  if (state.strainOpen === undefined) state.strainOpen = true;
  if (state.powersOpen === undefined) state.powersOpen = true;
```

**Step 4: Verify in browser**
Open DevTools console, run:
```js
JSON.parse(localStorage.getItem('talent_tracker_v1'))
```
Expected: Object with `strain`, `strainMax`, `manifestDie`, `knownPowers` fields present (or null if first load — reload page and check again).

**Step 5: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: add strain and powers state fields to DEFAULTS and migrations"
```

---

## Task 3: Build POWERS_DATA Constant

**Files:**
- Modify: `talent-tracker.html` — add `POWERS_DATA` constant above DEFAULTS

**Step 1: Extract power descriptions from PDF**

Run this Python script to extract pages 34–53 (the power descriptions):
```bash
/c/Users/archa/AppData/Local/Programs/Python/Python312/python -c "
import pdfplumber, sys
sys.stdout.reconfigure(encoding='utf-8')
path = r'C:\Users\archa\Desktop\Dnd Content\Content Books\The Talent and Psionics - Color.pdf'
with pdfplumber.open(path) as pdf:
    for i in range(33, 54):
        text = pdf.pages[i].extract_text()
        if text and text.strip():
            print(f'\n=== PAGE {i+1} ===')
            print(text)
" > C:/Users/archa/Desktop/Claude/powers_descriptions.txt 2>&1
```

Read `powers_descriptions.txt` to extract manifestation time, duration, and concentration flag for each power.

**Step 2: Add POWERS_DATA constant to the file**

Find the line `const DEFAULTS = {` (~line 1487). Directly above it, insert the full constant. Use this structure — fill in all 103 powers using data from the extracted text:

```js
  // === POWERS LIBRARY (not persisted to localStorage) ===
  const POWERS_DATA = [
    // CHRONOPATHY
    { id: 'glimpse', name: 'Glimpse', order: 1, school: 'Chronopathy', schoolKey: 'chr',
      manifestTime: 'bonus', concentration: true, duration: '1 round',
      description: 'Pick one willing creature within 5 ft. The next attack roll against them before end of your next turn has disadvantage.' },
    { id: 'time-thief', name: 'Time Thief', order: 1, school: 'Chronopathy', schoolKey: 'chr',
      manifestTime: 'action', concentration: false, duration: 'Instantaneous',
      description: 'Extract from the PDF.' },
    { id: 'again', name: 'Again', order: 2, school: 'Chronopathy', schoolKey: 'chr',
      manifestTime: 'reaction', concentration: false, duration: 'Instantaneous',
      description: 'Extract from the PDF.' },
    // ... all 103 powers ...
  ];
```

> **Note:** Use the extracted text from `powers_descriptions.txt` to fill in every power's `manifestTime`, `concentration`, `duration`, and `description`. The full roster is in the design doc. For manifestTime use: `'action'` | `'bonus'` | `'reaction'` | `'special'` (for minutes/hours).

**Step 3: Verify constant loads**

Open DevTools console:
```js
POWERS_DATA.length
```
Expected: `103`

```js
POWERS_DATA.filter(p => p.order === 1).map(p => p.name)
```
Expected: Array of all 1st-order power names.

**Step 4: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: add POWERS_DATA constant with all 103 psionic powers"
```

---

## Task 4: Remove Spellcasting CSS and Sections

**Files:**
- Modify: `talent-tracker.html` — remove spell slot CSS (~lines 138–632), prepared spells CSS (~lines 633–783), and spell slot JS render functions (~lines 2190–2416)

**Step 1: Remove spell slot CSS block**

Find the comment `/* === SPELL SLOTS === */` (~line 138). Delete everything from that comment through to just before `/* === PREPARED SPELLS === */` (~line 633).

**Step 2: Remove prepared spells CSS block**

Find `/* === PREPARED SPELLS === */` (~line 633, now shifted). Delete from there through to just before `/* === ACTIVE SPELLS === */`.

**Step 3: Remove spell JS functions**

Find `// === SPELL SLOTS ===` (now around line 2190 after CSS removal). Delete from that comment through to the end of `renderSRModal()` function — i.e., everything up to `// === STAT BLOCKS ===`.

This removes: `renderSlotsBody()`, `renderSlots()`, `renderPreparedSpells()`, `renderSRModal()`, and all their helper functions and event handlers.

**Step 4: Remove renderSlots() call from render()**

In `function render()` (near end of file), find and remove:
```js
    renderSlots();
```

**Step 5: Remove spell-related event handlers**

Search for any remaining references to `spellcastingAbility`, `spellDCOverride`, `spellAtkOverride`, `preparedSpells`, `renderSlots` and remove them. Key spots:
- `saveSection()` function: remove the `else if (key === 'spells')` branch
- `setEditSection()` section in render: remove the spells edit panel HTML

**Step 6: Verify no JS errors**
```
start talent-tracker.html
```
Open DevTools console. Expected: No red errors. Tracker loads, left column shows HP + character stats, no spell section visible.

**Step 7: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: remove spellcasting sections to make room for psionics"
```

---

## Task 5: Add Strain Tracker

**Files:**
- Modify: `talent-tracker.html` — add CSS (~after line 784), add `renderStrain()` JS function, wire into `render()`

**Step 1: Add Strain Tracker CSS**

After the `/* === ACTIVE SPELLS === */` CSS block, insert:
```css
    /* === STRAIN TRACKER === */
    .strain-section { margin-bottom: 20px; }
    .strain-row { display: flex; align-items: center; gap: 8px; margin-bottom: 8px; }
    .strain-label {
      font-size: 0.65rem; letter-spacing: 0.1em; text-transform: uppercase;
      width: 40px; font-weight: 600;
    }
    .strain-label.body { color: #FF4DA6; }
    .strain-label.mind { color: #4DD9FF; }
    .strain-label.soul { color: #9B59FF; }
    .strain-pips { display: flex; gap: 3px; flex: 1; }
    .strain-pip {
      font-size: 0.9rem; cursor: pointer; opacity: 0.2;
      transition: opacity 0.15s, filter 0.15s;
      user-select: none;
    }
    .strain-pip.filled { opacity: 1; }
    .strain-pip.filled.body  { filter: drop-shadow(0 0 4px #FF4DA6); }
    .strain-pip.filled.mind  { filter: drop-shadow(0 0 4px #4DD9FF); }
    .strain-pip.filled.soul  { filter: drop-shadow(0 0 4px #9B59FF); }
    .strain-adj { background: none; border: 1px solid #1a3a4a; color: #4a7a8a;
      border-radius: 3px; width: 20px; height: 20px; font-size: 0.75rem;
      cursor: pointer; display: flex; align-items: center; justify-content: center; }
    .strain-adj:hover { border-color: #a8d8e8; color: #a8d8e8; }
    .strain-total {
      font-size: 0.75rem; text-align: center; margin-bottom: 8px;
      color: #4a7a8a; letter-spacing: 0.05em;
    }
    .strain-total.warning { color: #d04040; animation: pulse 1s infinite; }
    .strain-total strong { color: #c8dde8; }
    .manifest-die-display {
      font-size: 0.7rem; color: #4a7a8a; text-align: center; margin-top: 4px;
    }
    .manifest-die-display span { color: #FF4DA6; font-weight: 600; }
```

**Step 2: Add `renderStrain()` function**

After `// === ACTIVE SPELLS ===` in the JS section, add:
```js
  // === STRAIN TRACKER ===
  function renderStrain() {
    const { strain, strainMax, manifestDie, strainOpen } = state;
    const total = strain.body + strain.mind + strain.soul;
    const isWarning = total >= strainMax - 2;

    const pipRow = (type, color) => {
      const val = strain[type];
      const pips = Array.from({ length: 8 }, (_, i) =>
        `<span class="strain-pip ${i < val ? 'filled ' + type : ''}"
          onclick="adjustStrain('${type}', ${i + 1})" title="${type} strain ${i + 1}">🧠</span>`
      ).join('');
      return `
        <div class="strain-row">
          <span class="strain-label ${type}">${type}</span>
          <div class="strain-pips">${pips}</div>
          <button class="strain-adj" onclick="adjustStrain('${type}', -1)" title="Remove ${type} strain">−</button>
          <button class="strain-adj" onclick="adjustStrain('${type}', 1)" title="Add ${type} strain">+</button>
        </div>`;
    };

    const section = document.getElementById('strain-section');
    if (!section) return;
    section.innerHTML = sectionTitle('Strain', 'strainOpen', () => update(s => s.strainOpen = !s.strainOpen), strainOpen) + `
      ${strainOpen ? `
        <div class="strain-total ${isWarning ? 'warning' : ''}">
          Total Strain: <strong>${total}</strong> / <strong>${strainMax}</strong>
          ${total >= strainMax ? ' ⚠ MAXIMUM REACHED' : ''}
        </div>
        <div class="strain-section">
          ${pipRow('body', '#FF4DA6')}
          ${pipRow('mind', '#4DD9FF')}
          ${pipRow('soul', '#9B59FF')}
        </div>
        <div class="manifest-die-display">Manifestation Die: <span>${manifestDie}</span></div>
      ` : ''}`;
  }

  function adjustStrain(type, delta) {
    update(s => {
      if (delta === -1) {
        s.strain[type] = Math.max(0, s.strain[type] - 1);
      } else if (delta > 0 && delta <= 8) {
        // Clicking pip N sets that type to N (toggle off if already at N)
        s.strain[type] = s.strain[type] === delta ? delta - 1 : delta;
      } else {
        s.strain[type] = Math.min(8, s.strain[type] + 1);
      }
    });
    renderAutoStrainEffects();
  }

  function renderAutoStrainEffects() {
    // Called after any strain change — updates Effects section
    // See Task 8 for implementation
    renderStatus();
  }
```

**Step 3: Add strain section to HTML left column**

Find the HP section `<div id="hp-section">` in the HTML body. After its closing `</div>`, add:
```html
    <div id="strain-section"></div>
```

**Step 4: Wire into `render()`**

In `function render()`, add:
```js
    renderStrain();
```
right after `renderHP();`.

**Step 5: Add `strainMax` and `manifestDie` as editable header fields**

In `renderHeader()`, find where HP max is shown as editable. Add adjacent fields for Strain Max and Die. Follow the same `editableVal` pattern used for HP max.

**Step 6: Verify in browser**

Open `talent-tracker.html`. Expected:
- Strain section appears below HP bar
- Three rows: BODY (pink), MIND (cyan), SOUL (violet)
- 8 brain pips per row, unfilled
- Clicking a pip fills it and adjacent ones glow
- +/− buttons adjust count
- Total updates live
- "Manifestation Die: d6" shown at bottom

**Step 7: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: add strain tracker with brain pips and type adjustments"
```

---

## Task 6: Rename Active Spells → Effects + Auto-Strain Conditions

**Files:**
- Modify: `talent-tracker.html` — update `renderStatus()` (~line 3516) and its CSS

**Step 1: Update section title in renderStatus()**

Find `renderStatus()` (~line 3557). Change the section heading from "Active Spells" / "Status" to "Effects". Update all UI labels.

**Step 2: Add auto-strain effects rendering**

In `renderStatus()`, before rendering manual effects, compute and prepend the auto-strain effects:

```js
  const STRAIN_EFFECTS = {
    body: [
      { threshold: 1, text: 'Disadv. on STR/DEX checks' },
      { threshold: 3, text: 'Speed halved' },
      { threshold: 5, text: 'Disadv. on STR/DEX saves' },
      { threshold: 7, text: 'HP max halved' },
    ],
    mind: [
      { threshold: 1, text: "Can't Dash, Disengage, or Dodge" },
      { threshold: 3, text: 'No skill proficiency' },
      { threshold: 5, text: '−5 penalty to AC' },
      { threshold: 7, text: 'No save proficiency' },
    ],
    soul: [
      { threshold: 1, text: 'Disadv. on WIS/CHA checks' },
      { threshold: 3, text: 'Disadv. on death saves' },
      { threshold: 5, text: 'Disadv. on WIS/CHA saves' },
      { threshold: 7, text: 'Half healing from supernatural effects' },
    ],
  };

  const STRAIN_COLORS = { body: '#FF4DA6', mind: '#4DD9FF', soul: '#9B59FF' };
```

Then in the HTML render, output active strain effects before manual effects:

```js
  const autoEffects = ['body', 'mind', 'soul'].flatMap(type =>
    STRAIN_EFFECTS[type]
      .filter(e => state.strain[type] >= e.threshold)
      .map(e => `
        <div class="active-spell" style="border-left: 3px solid ${STRAIN_COLORS[type]}">
          <span class="spell-name" style="color:${STRAIN_COLORS[type]};font-size:0.65rem;text-transform:uppercase;letter-spacing:0.05em">${type}</span>
          <span class="spell-name">${e.text}</span>
        </div>`)
  ).join('');
```

**Step 3: Rename "Active Spells" CSS comment and any IDs**

Search for `active-spells` or `activeSpells` in HTML IDs/classes and rename to `effects` where they're section containers (not the `activeSpells` state field — keep that as is for now, rename later).

**Step 4: Verify in browser**

Set body strain to 3 in DevTools:
```js
state.strain.body = 3; render();
```
Expected: Effects section shows two pink badges — "Disadv. on STR/DEX checks" and "Speed halved". Setting it back to 0 removes them.

**Step 5: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: rename Active Spells to Effects, add auto-strain condition badges"
```

---

## Task 7: Add Active Powers List

**Files:**
- Modify: `talent-tracker.html` — add CSS, `renderActivePowers()` JS function, HTML slot, wire into `render()`

**Step 1: Add Active Powers CSS**

After the Strain Tracker CSS block, add:
```css
    /* === ACTIVE POWERS === */
    .powers-section { margin-bottom: 20px; }
    .power-card {
      background: #07131c; border: 1px solid #0f2535;
      border-radius: 4px; padding: 8px 10px; margin-bottom: 6px;
      display: flex; align-items: center; gap: 8px;
    }
    .power-card:hover { border-color: #1a3a4a; }
    .power-order-badge {
      font-size: 0.6rem; background: #0a1e2a; border: 1px solid #1a3a4a;
      border-radius: 3px; padding: 2px 5px; color: #4a7a8a;
      white-space: nowrap; font-family: 'Cinzel', serif;
    }
    .power-name { flex: 1; font-size: 0.8rem; color: #c8dde8; }
    .power-school { font-size: 0.6rem; color: #4a7a8a; text-transform: uppercase; letter-spacing: 0.08em; }
    .power-conc-toggle {
      font-size: 0.65rem; padding: 2px 6px; border-radius: 3px; cursor: pointer;
      border: 1px solid #1a3a4a; background: none; color: #4a7a8a;
    }
    .power-conc-toggle.active { border-color: #FF4DA6; color: #FF4DA6; }
    .power-manifest-btn {
      font-size: 0.65rem; padding: 3px 8px; border-radius: 3px; cursor: pointer;
      background: #0a1e2a; border: 1px solid #1a4a6a; color: #a8d8e8;
    }
    .power-manifest-btn:hover { background: #1a3a4a; }
    .powers-manage-btn {
      width: 100%; font-size: 0.7rem; padding: 6px; margin-top: 4px;
      background: none; border: 1px dashed #1a3a4a; border-radius: 4px;
      color: #4a7a8a; cursor: pointer; letter-spacing: 0.05em;
    }
    .powers-manage-btn:hover { border-color: #a8d8e8; color: #a8d8e8; }
```

**Step 2: Add `renderActivePowers()` function**

After `renderStrain()`, add:
```js
  // === ACTIVE POWERS ===
  function renderActivePowers() {
    const { knownPowers, concentration, powersOpen } = state;
    const activePowers = POWERS_DATA.filter(p => knownPowers.includes(p.id));

    const cards = activePowers.map(p => {
      const isConc = concentration.includes(p.id);
      const ordinal = ['', '1st', '2nd', '3rd', '4th', '5th', '6th'][p.order];
      return `
        <div class="power-card">
          <span class="power-order-badge">${ordinal}</span>
          <span class="power-name">
            ${p.name}
            <span class="power-school">${p.school}</span>
          </span>
          ${p.concentration ? `
            <button class="power-conc-toggle ${isConc ? 'active' : ''}"
              onclick="toggleConcentration('${p.id}')" title="Toggle concentration">C</button>
          ` : ''}
          <button class="power-manifest-btn" onclick="openManifestModal('${p.id}')">Manifest</button>
        </div>`;
    }).join('');

    const section = document.getElementById('powers-section');
    if (!section) return;
    section.innerHTML = sectionTitle('Active Powers', 'powersOpen', () => update(s => s.powersOpen = !s.powersOpen), powersOpen) + `
      ${powersOpen ? `
        <div class="powers-section">
          ${activePowers.length === 0
            ? `<div style="font-size:0.75rem;color:#2a5a6a;text-align:center;padding:12px">No powers selected — use Manage Powers to add them</div>`
            : cards}
          <button class="powers-manage-btn" onclick="openManagePowers()">⚙ Manage Powers</button>
        </div>
      ` : ''}`;
  }

  function toggleConcentration(id) {
    update(s => {
      const idx = s.concentration.indexOf(id);
      if (idx >= 0) s.concentration.splice(idx, 1);
      else s.concentration.push(id);
    });
  }

  function openManifestModal(id) {
    // Placeholder — implemented in Task 9
    const power = POWERS_DATA.find(p => p.id === id);
    if (!power) return;
    document.getElementById('manifest-modal').style.display = 'flex';
    renderManifestModal(power);
  }

  function openManagePowers() {
    document.getElementById('manage-powers-modal').style.display = 'flex';
    renderManagePowersModal();
  }
```

**Step 3: Add HTML slots**

After `<div id="strain-section"></div>` in the left column HTML, add:
```html
    <div id="powers-section"></div>
```

Also at end of `<body>`, before the closing `</body>` tag, add placeholder modals (implemented in Tasks 8 & 9):
```html
    <div id="manage-powers-modal" style="display:none"></div>
    <div id="manifest-modal" style="display:none"></div>
```

**Step 4: Wire into `render()`**

Add after `renderStrain()`:
```js
    renderActivePowers();
```

**Step 5: Verify in browser**

Open DevTools and run:
```js
update(s => s.knownPowers = ['glimpse', 'caress-of-fire', 'again']);
```
Expected: Three power cards appear below strain tracker. "Manage Powers" button visible at bottom.

**Step 6: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: add active powers list with concentration toggle and Manifest button"
```

---

## Task 8: Manage Powers Modal

**Files:**
- Modify: `talent-tracker.html` — add modal CSS, `renderManagePowersModal()` function

**Step 1: Add Manage Powers modal CSS**

After the Active Powers CSS, add:
```css
    /* === MANAGE POWERS MODAL === */
    .modal-overlay {
      position: fixed; inset: 0; background: rgba(0,0,0,0.8);
      display: flex; align-items: center; justify-content: center;
      z-index: 1000;
    }
    .modal-box {
      background: #060f18; border: 1px solid #1a3a4a; border-radius: 8px;
      padding: 20px; max-width: 600px; width: 95vw;
      max-height: 80vh; overflow-y: auto;
    }
    .modal-title {
      font-family: 'Cinzel', serif; font-size: 1rem; color: #a8d8e8;
      margin-bottom: 16px; padding-bottom: 8px; border-bottom: 1px solid #0f2535;
      display: flex; justify-content: space-between; align-items: center;
    }
    .modal-close {
      background: none; border: none; color: #4a7a8a; font-size: 1.2rem;
      cursor: pointer; padding: 0 4px;
    }
    .modal-close:hover { color: #d04040; }
    .order-group { margin-bottom: 16px; }
    .order-group-title {
      font-family: 'Cinzel', serif; font-size: 0.7rem; color: #4a7a8a;
      text-transform: uppercase; letter-spacing: 0.1em;
      margin-bottom: 8px; padding-bottom: 4px; border-bottom: 1px solid #071320;
    }
    .power-pick-row {
      display: flex; align-items: center; gap: 8px; padding: 4px 0;
      border-bottom: 1px solid #071320;
    }
    .power-pick-row:last-child { border-bottom: none; }
    .power-pick-checkbox { accent-color: #FF4DA6; width: 14px; height: 14px; cursor: pointer; }
    .power-pick-name { flex: 1; font-size: 0.8rem; color: #c8dde8; }
    .power-pick-school { font-size: 0.65rem; color: #4a7a8a; text-transform: uppercase; }
    .power-pick-time { font-size: 0.65rem; color: #2a6a7a; }
    .modal-search {
      width: 100%; background: #07131c; border: 1px solid #1a3a4a; color: #c8dde8;
      border-radius: 4px; padding: 6px 10px; font-size: 0.8rem; margin-bottom: 12px;
    }
    .school-filter { display: flex; gap: 4px; flex-wrap: wrap; margin-bottom: 12px; }
    .school-btn {
      font-size: 0.65rem; padding: 3px 8px; border-radius: 12px; cursor: pointer;
      border: 1px solid #1a3a4a; background: none; color: #4a7a8a;
    }
    .school-btn.active { background: #0a1e2a; border-color: #FF4DA6; color: #FF4DA6; }
```

**Step 2: Add `renderManagePowersModal()` function**

```js
  // === MANAGE POWERS MODAL ===
  let managePowersFilter = { search: '', school: '' };

  function renderManagePowersModal() {
    const modal = document.getElementById('manage-powers-modal');
    const { search, school } = managePowersFilter;
    const schools = ['Chronopathy', 'Metamorphosis', 'Pyrokinesis', 'Resopathy', 'Telekinesis', 'Telepathy'];

    const filtered = POWERS_DATA.filter(p =>
      (!search || p.name.toLowerCase().includes(search.toLowerCase())) &&
      (!school || p.school === school)
    );

    const byOrder = [1, 2, 3, 4, 5, 6].map(order => {
      const powers = filtered.filter(p => p.order === order);
      if (!powers.length) return '';
      const ordinal = ['', '1st', '2nd', '3rd', '4th', '5th', '6th'][order];
      const rows = powers.map(p => `
        <div class="power-pick-row">
          <input type="checkbox" class="power-pick-checkbox"
            ${state.knownPowers.includes(p.id) ? 'checked' : ''}
            onchange="toggleKnownPower('${p.id}', this.checked)">
          <span class="power-pick-name">${p.name}</span>
          <span class="power-pick-school">${p.school}</span>
          <span class="power-pick-time">${p.manifestTime}</span>
        </div>`).join('');
      return `<div class="order-group"><div class="order-group-title">${ordinal} Order</div>${rows}</div>`;
    }).join('');

    modal.className = 'modal-overlay';
    modal.innerHTML = `
      <div class="modal-box">
        <div class="modal-title">
          Manage Powers
          <button class="modal-close" onclick="closeManagePowers()">✕</button>
        </div>
        <input class="modal-search" placeholder="Search powers..."
          value="${search}" oninput="managePowersSearch(this.value)">
        <div class="school-filter">
          <button class="school-btn ${!school ? 'active' : ''}" onclick="managePowersSchool('')">All</button>
          ${schools.map(s => `<button class="school-btn ${school === s ? 'active' : ''}" onclick="managePowersSchool('${s}')">${s}</button>`).join('')}
        </div>
        ${byOrder || '<div style="color:#2a5a6a;text-align:center;padding:20px">No powers match</div>'}
      </div>`;
  }

  function toggleKnownPower(id, checked) {
    update(s => {
      if (checked && !s.knownPowers.includes(id)) s.knownPowers.push(id);
      if (!checked) {
        s.knownPowers = s.knownPowers.filter(p => p !== id);
        s.concentration = s.concentration.filter(p => p !== id);
      }
    });
    renderActivePowers();
  }

  function managePowersSearch(val) {
    managePowersFilter.search = val;
    renderManagePowersModal();
  }

  function managePowersSchool(val) {
    managePowersFilter.school = val;
    renderManagePowersModal();
  }

  function closeManagePowers() {
    document.getElementById('manage-powers-modal').style.display = 'none';
    managePowersFilter = { search: '', school: '' };
  }
```

**Step 3: Verify in browser**

Click "Manage Powers" button. Expected:
- Modal overlays the page
- 103 powers listed, grouped by order
- Search filters by name in real time
- School buttons filter by specialization
- Checking a power adds it to Active Powers list
- ✕ closes the modal

**Step 4: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: add Manage Powers modal with search, school filter, and selection"
```

---

## Task 9: Manifestation Resolver Modal

**Files:**
- Modify: `talent-tracker.html` — add resolver CSS and `renderManifestModal()` multi-step logic

**Step 1: Add Resolver CSS**

After the Manage Powers modal CSS, add:
```css
    /* === MANIFEST RESOLVER === */
    .resolver-box { max-width: 400px; }
    .resolver-step { margin-bottom: 16px; }
    .resolver-score {
      font-size: 1.4rem; text-align: center; font-family: 'Cinzel', serif;
      color: #a8d8e8; margin: 8px 0;
    }
    .resolver-score small { font-size: 0.7rem; color: #4a7a8a; display: block; }
    .resolver-roll-btn {
      width: 100%; padding: 12px; font-size: 1rem; border-radius: 4px;
      background: #0a1e2a; border: 1px solid #1a4a6a; color: #a8d8e8;
      cursor: pointer; font-family: 'Cinzel', serif; letter-spacing: 0.1em;
    }
    .resolver-roll-btn:hover { background: #1a3a4a; }
    .resolver-result {
      text-align: center; padding: 12px; border-radius: 4px; margin: 12px 0;
      font-size: 0.9rem;
    }
    .resolver-result.success { background: #0a2a0a; border: 1px solid #1a6a1a; color: #4ada4a; }
    .resolver-result.partial { background: #2a2a0a; border: 1px solid #6a6a1a; color: #dada4a; }
    .resolver-result.failure { background: #2a0a0a; border: 1px solid #6a1a1a; color: #da4a4a; }
    .resolver-result.death-warning {
      background: #1a0a0a; border: 1px solid #d04040; color: #d04040;
    }
    .strain-alloc { display: flex; gap: 8px; justify-content: center; margin: 12px 0; }
    .strain-alloc-btn {
      padding: 8px 16px; border-radius: 4px; border: 1px solid; cursor: pointer;
      font-size: 0.8rem; background: none;
    }
    .strain-alloc-btn.body { border-color: #FF4DA6; color: #FF4DA6; }
    .strain-alloc-btn.mind { border-color: #4DD9FF; color: #4DD9FF; }
    .strain-alloc-btn.soul { border-color: #9B59FF; color: #9B59FF; }
    .strain-alloc-btn:hover { opacity: 0.8; }
    .alloc-tally { text-align: center; font-size: 0.8rem; color: #4a7a8a; margin-bottom: 8px; }
    .resolver-confirm-btn {
      width: 100%; padding: 10px; border-radius: 4px; cursor: pointer;
      background: #1a3a4a; border: 1px solid #4a7a8a; color: #a8d8e8;
      font-size: 0.85rem;
    }
    .resolver-confirm-btn:disabled { opacity: 0.4; cursor: not-allowed; }
```

**Step 2: Add `renderManifestModal()` with step state**

```js
  // === MANIFESTATION RESOLVER ===
  let manifestState = { power: null, step: 1, roll: null, strainGained: 0, allocated: { body: 0, mind: 0, soul: 0 } };

  function renderManifestModal(power) {
    manifestState = { power, step: 1, roll: null, strainGained: 0, allocated: { body: 0, mind: 0, soul: 0 } };
    _drawManifestModal();
  }

  function _drawManifestModal() {
    const modal = document.getElementById('manifest-modal');
    const { power, step, roll, strainGained, allocated } = manifestState;
    const { concentration, strain, strainMax } = state;

    // Manifestation score: order + 1 per extra concentrating power
    const concCount = concentration.filter(id => id !== power.id).length;
    const score = power.order + concCount;
    const total = strain.body + strain.mind + strain.soul;
    const allocTotal = allocated.body + allocated.mind + allocated.soul;
    const dieLabel = state.manifestDie.toUpperCase();

    // 1st-order powers skip the test entirely
    if (power.order === 1) {
      modal.className = 'modal-overlay';
      modal.innerHTML = `
        <div class="modal-box resolver-box">
          <div class="modal-title">${power.name} <button class="modal-close" onclick="closeManifestModal()">✕</button></div>
          <div style="text-align:center;padding:16px">
            <div style="font-size:0.8rem;color:#4a7a8a;margin-bottom:8px">1st-order power — no manifestation test required.</div>
            <div class="resolver-result success">✓ Power manifested. No strain.</div>
            <button class="resolver-confirm-btn" style="margin-top:12px" onclick="closeManifestModal()">Done</button>
          </div>
        </div>`;
      return;
    }

    let body = '';

    if (step === 1) {
      const concNames = concentration.filter(id => id !== power.id)
        .map(id => POWERS_DATA.find(p => p.id === id)?.name).filter(Boolean);
      body = `
        <div class="resolver-step">
          <div class="resolver-score">
            Score: ${score}
            <small>Order ${power.order}${concCount > 0 ? ` + ${concCount} (concentrating on ${concNames.join(', ')})` : ''}</small>
          </div>
          <div style="font-size:0.75rem;color:#4a7a8a;text-align:center;margin-bottom:12px">
            Roll your ${dieLabel}. If roll > ${score}: no strain. If = ${score}: +1 strain. If < ${score}: +${power.order} strain.
          </div>
          <button class="resolver-roll-btn" onclick="manifestRollForMe()">🎲 Roll ${dieLabel} for me</button>
          <div style="text-align:center;margin:8px 0;font-size:0.7rem;color:#4a7a8a">— or enter your roll —</div>
          <div style="display:flex;gap:6px;justify-content:center">
            ${Array.from({ length: parseInt(state.manifestDie.slice(1)) }, (_, i) =>
              `<button class="strain-adj" onclick="manifestSetRoll(${i + 1})" style="width:28px">${i + 1}</button>`
            ).join('')}
          </div>
        </div>`;
    } else if (step === 2) {
      let resultClass, resultText;
      const wouldExceed = total + strainGained > strainMax;
      if (roll > score)       { resultClass = 'success'; resultText = `✓ ${roll} > ${score} — Success! No strain.`; }
      else if (roll === score) { resultClass = 'partial'; resultText = `~ ${roll} = ${score} — Success + 1 strain`; }
      else                     { resultClass = 'failure'; resultText = `✗ ${roll} < ${score} — Success + ${strainGained} strain (order ${power.order})`; }

      if (wouldExceed && strainGained > 0) {
        body = `
          <div class="resolver-result death-warning">
            ⚠ Gaining ${strainGained} strain would exceed your maximum (${total} + ${strainGained} > ${strainMax}).
            <div style="margin-top:8px;font-size:0.8rem">Choose:</div>
          </div>
          <div style="display:flex;gap:8px;margin-top:8px">
            <button class="resolver-confirm-btn" onclick="manifestAndDie()">Manifest & Die</button>
            <button class="resolver-confirm-btn" onclick="manifestCancel()">Cancel (drop to 0 HP)</button>
          </div>`;
      } else if (strainGained === 0) {
        body = `
          <div class="resolver-result ${resultClass}">${resultText}</div>
          <button class="resolver-confirm-btn" style="margin-top:8px" onclick="closeManifestModal()">Done ✓</button>`;
      } else {
        body = `
          <div class="resolver-result ${resultClass}">${resultText}</div>
          <div style="margin:12px 0;font-size:0.8rem;color:#4a7a8a;text-align:center">Distribute ${strainGained} strain:</div>
          <div class="alloc-tally">${allocTotal} / ${strainGained} allocated</div>
          <div class="strain-alloc">
            <button class="strain-alloc-btn body" onclick="manifestAlloc('body')">+Body (${allocated.body})</button>
            <button class="strain-alloc-btn mind" onclick="manifestAlloc('mind')">+Mind (${allocated.mind})</button>
            <button class="strain-alloc-btn soul" onclick="manifestAlloc('soul')">+Soul (${allocated.soul})</button>
          </div>
          <button class="resolver-confirm-btn" ${allocTotal < strainGained ? 'disabled' : ''} onclick="manifestConfirmStrain()">
            Confirm Strain
          </button>`;
      }
    }

    modal.className = 'modal-overlay';
    modal.innerHTML = `
      <div class="modal-box resolver-box">
        <div class="modal-title">
          Manifest: ${power.name}
          <button class="modal-close" onclick="closeManifestModal()">✕</button>
        </div>
        <div style="font-size:0.7rem;color:#4a7a8a;margin-bottom:12px">
          ${power.school} &bull; ${['','1st','2nd','3rd','4th','5th','6th'][power.order]} Order
          ${power.concentration ? ' &bull; <span style="color:#FF4DA6">Concentration</span>' : ''}
        </div>
        ${body}
      </div>`;
  }

  function manifestRollForMe() {
    const sides = parseInt(state.manifestDie.slice(1));
    manifestSetRoll(Math.ceil(Math.random() * sides));
  }

  function manifestSetRoll(roll) {
    const { power } = manifestState;
    const concCount = state.concentration.filter(id => id !== power.id).length;
    const score = power.order + concCount;
    let strainGained = 0;
    if (roll === score) strainGained = 1;
    if (roll < score)  strainGained = power.order;
    manifestState = { ...manifestState, step: 2, roll, strainGained, allocated: { body: 0, mind: 0, soul: 0 } };
    _drawManifestModal();
  }

  function manifestAlloc(type) {
    const { strainGained, allocated } = manifestState;
    const total = allocated.body + allocated.mind + allocated.soul;
    if (total >= strainGained) return;
    manifestState.allocated[type]++;
    _drawManifestModal();
  }

  function manifestConfirmStrain() {
    const { allocated } = manifestState;
    update(s => {
      s.strain.body = Math.min(8, s.strain.body + allocated.body);
      s.strain.mind = Math.min(8, s.strain.mind + allocated.mind);
      s.strain.soul = Math.min(8, s.strain.soul + allocated.soul);
    });
    closeManifestModal();
    renderStrain();
    renderStatus();
  }

  function manifestAndDie() {
    const { allocated } = manifestState;
    update(s => {
      s.strain.body = Math.min(8, s.strain.body + allocated.body);
      s.strain.mind = Math.min(8, s.strain.mind + allocated.mind);
      s.strain.soul = Math.min(8, s.strain.soul + allocated.soul);
      s.hp.current = 0;
    });
    closeManifestModal();
    render();
  }

  function manifestCancel() {
    update(s => s.hp.current = 0);
    closeManifestModal();
    render();
  }

  function closeManifestModal() {
    document.getElementById('manifest-modal').style.display = 'none';
    manifestState = { power: null, step: 1, roll: null, strainGained: 0, allocated: { body: 0, mind: 0, soul: 0 } };
  }
```

**Step 3: Verify in browser**

Select a 2nd-order power. Click Manifest. Expected:
- Modal shows: power name, score calculation (e.g. "Score: 2"), roll buttons
- Click "Roll d6 for me" → result shown with color coding
- If strain gained: allocation buttons appear, confirm disabled until allocated
- Confirming updates the strain tracker and closes modal
- 1st-order powers skip to "No strain" immediately

**Step 4: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: add manifestation resolver modal with step-by-step strain allocation"
```

---

## Task 10: Add Power Save DC and Power Attack to Formulas

**Files:**
- Modify: `talent-tracker.html` — update the stats/formulas display section

**Step 1: Locate the stat block render area**

Find `renderStatPanel()` or the formulas display in `renderSlots()` area (which was removed). The Power Save DC and Power Attack are now standalone — add them to the character stats panel.

Find where Spell Save DC was rendered (~around where `getSpellDC()` was called). Replace those entries with:

```js
  function getPowerSaveDC() {
    const intMod = Math.floor((state.stats.int - 10) / 2);
    return 8 + state.profBonus + intMod;
  }

  function getPowerAtk() {
    const intMod = Math.floor((state.stats.int - 10) / 2);
    return state.profBonus + intMod;
  }
```

**Step 2: Add to the stats display**

In the stats panel HTML (inside `renderSlots()` or wherever the DC/Atk were displayed), add:
```js
  `<div class="stat-row">
    <span class="stat-label">Power Save DC</span>
    <span class="stat-value">${getPowerSaveDC()}</span>
  </div>
  <div class="stat-row">
    <span class="stat-label">Power Attack</span>
    <span class="stat-value">+${getPowerAtk()}</span>
  </div>`
```

These auto-update whenever INT or proficiency bonus changes — no override needed (unlike spell DC which had a manual override option).

**Step 3: Verify in browser**

Change INT to 18 in character stats. Expected: Power Save DC = 8 + profBonus + 4, Power Attack = profBonus + 4. Values update immediately.

**Step 4: Commit**
```bash
git add talent-tracker.html
git commit -m "feat: add Power Save DC and Power Attack to stats panel"
```

---

## Task 11: Final Polish and Verification

**Files:**
- Modify: `talent-tracker.html` — colors, edge cases, title, CHANGELOG

**Step 1: Verify color palette**

Confirm strain type colors appear correctly throughout:
- Brain pips glow correct color when filled
- Effects section badges use correct border colors
- Allocation buttons in resolver use correct colors

**Step 2: Verify collapsibility**

All new sections (Strain, Active Powers) collapse/expand via chevron. State persists across reload.

**Step 3: Verify strain death threshold**

In DevTools:
```js
update(s => { s.strainMax = 5; s.strain = { body: 2, mind: 2, soul: 0 }; });
```
Then click Manifest on a 3rd-order power. Enter a roll of 1.
Expected: "Manifest & Die" / "Cancel" choice appears.

**Step 4: Verify Effects auto-populate**

```js
update(s => s.strain.body = 3);
```
Expected: "Disadv. on STR/DEX checks" and "Speed halved" appear in Effects section in pink.

**Step 5: Update CHANGELOG.md**

Add an entry for the `talent-tracker` branch:
```markdown
## [1.0.0-talent] — 2026-02-XX
### Added
- Full psionics system replacing spellcasting
- Strain tracker: Body/Mind/Soul pips (8 each), total vs. maximum
- Auto-strain conditions in Effects section, color-coded by type
- Powers library: all 103 MCDM Talent powers, organized by order and school
- Manage Powers modal: search, school filter, checkbox selection
- Manifestation resolver: step-by-step test, roll integration, strain allocation
- Power Save DC and Power Attack auto-calculated from INT
```

**Step 6: Final commit**
```bash
git add talent-tracker.html CHANGELOG.md
git commit -m "feat: polish, verify edge cases, update CHANGELOG for talent-tracker"
```

---

## Quick Reference: Key Locations in tracker.html

| What | Where |
|---|---|
| `POWERS_DATA` constant | Add above line 1487 |
| `DEFAULTS` object | Line 1487 |
| Migration guards | Line ~1574 |
| New CSS sections | After line ~820 (after Active Spells CSS) |
| `renderStrain()` | Add after `// === ACTIVE SPELLS ===` (~3516) |
| `renderActivePowers()` | After `renderStrain()` |
| Modal functions | After `renderActivePowers()` |
| `function render()` | Line 3909 |
| HTML left column | Find `<div id="hp-section">`, add new divs below |

---

## Color Reference (do not use in main tracker)

| Type | Hex |
|---|---|
| Body strain | `#FF4DA6` |
| Mind strain | `#4DD9FF` |
| Soul strain | `#9B59FF` |
