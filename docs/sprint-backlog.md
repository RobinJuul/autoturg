# Sprint 2: UI Overhaul + Vehicle Lookup + API Integration

> **Goal:** Transform from admin dashboard to consumer product. Add Vehicle Lookup as hero feature. Replace URL-guessing with API. Fix critical UX issues.
> **Sprint dates:** 2026-03-25 — 2026-04-08
> **References:** Phase 2 in `buildplan.md`, `docs/ui-review.md`

---

## Priority Order

Builder should work through tasks in this order. Tasks marked ⚡ are quick wins.

1. **Task 2.1** — Fix scroll bug ⚡
2. **Task 2.2** — Move categories from sidebar to in-page segment control
3. **Task 2.3** — Add timeline preset buttons ⚡
4. **Task 2.4** — Build Vehicle Lookup page (hero feature)
5. **Task 2.5** — Integrate andmed.eesti.ee OpenData API
6. **Task 2.6** — UI polish pass
7. **Task 2.7** — Update documentation

---

## Task 2.1: Fix scroll bug ⚡

**Status:** NOT STARTED
**Priority:** P0 — blocks all page content below the fold
**Files:** `index.html` (CSS)

**The bug:** `.main` element (line ~107) lacks `overflow-y: auto` and `height: 100vh`. Content below the first viewport fold is unreachable.

**Fix:**
```css
.main {
  flex: 1;
  min-width: 0;
  overflow-x: hidden;
  overflow-y: auto;      /* ADD THIS */
  height: 100vh;         /* ADD THIS */
}
```

**Acceptance criteria:**
- All pages scroll to show full content
- Vehicle Lookup shows production year chart below the monthly chart
- Overview page scrolls through the full makes table

---

## Task 2.2: Move categories from sidebar to in-page segment control

**Status:** NOT STARTED
**Priority:** P0 — fundamental UX improvement
**Files:** `index.html` (HTML, CSS, JS)

**Problem:** Category buttons (Järelturg, Uued sõidukid, Import, Kogu turg) are in the sidebar as navigation buttons. This confuses users — they think switching category navigates somewhere. Categories should be a **data filter** within each page, not navigation.

**What to do:**

### A. Remove category section from sidebar
Remove the entire `Category` nav-section (lines 648–662). The sidebar should only have:
- Logo
- Views: Monthly Overview, Model Comparison, Vehicle Lookup
- (Move "Sync & Upload" to a small icon/link at sidebar bottom, not a full nav button)

### B. Add segment control to each page
Add a **pill-style segment control** below the timeline bar, inside each page's content area. It replaces the sidebar categories.

**HTML pattern:**
```html
<div class="category-tabs">
  <button class="cat-tab active" data-cat="jarelturg" onclick="switchCategory('jarelturg')">
    Järelturg
  </button>
  <button class="cat-tab" data-cat="newCars" onclick="switchCategory('newCars')">
    Uued sõidukid
  </button>
  <button class="cat-tab" data-cat="imports" onclick="switchCategory('imports')">
    Import
  </button>
  <button class="cat-tab" data-cat="koguTurg" onclick="switchCategory('koguTurg')">
    Kogu turg
  </button>
</div>
```

**CSS for segment control:**
```css
.category-tabs {
  display: flex;
  gap: 4px;
  background: var(--s3);
  padding: 3px;
  border-radius: 8px;
  width: fit-content;
  margin-bottom: 20px;
}
.cat-tab {
  font-family: var(--font);
  font-size: 13px;
  font-weight: 500;
  padding: 7px 16px;
  border: none;
  border-radius: 6px;
  background: transparent;
  color: var(--muted);
  cursor: pointer;
  transition: all 0.15s;
}
.cat-tab.active {
  background: var(--s1);
  color: var(--text);
  box-shadow: 0 1px 3px rgba(0,0,0,0.08);
}
.cat-tab:hover:not(.active) {
  color: var(--text);
}
```

### C. Update `switchCategory()` function
- Instead of toggling `.cat-btn` in sidebar, toggle `.cat-tab.active` on the in-page tabs
- **Do NOT clear comparison slots** when switching category (current behavior resets all 5 slots — that's destructive)
- Keep the category in sync across pages: when you switch category on Overview, the same category should be active when you navigate to Comparison

### D. Place the segment control
- **Overview page:** Place it right above the stats-grid (first thing user sees)
- **Comparison page:** Place it above the model picker grid
- **Vehicle Lookup page:** Not needed (vehicle lookup is cross-category)

**Acceptance criteria:**
- Sidebar has no "Category" section
- Each page (overview, comparison) has a pill-style category switcher at the top
- Switching category re-renders that page's data
- Category stays synced across pages
- Sidebar is narrower/cleaner with just Views + logo

---

## Task 2.3: Add timeline preset buttons ⚡

**Status:** NOT STARTED
**Priority:** P1 — quick usability win
**Files:** `index.html` (HTML, JS)

**What to do:**

Add preset buttons next to the existing "All" button in the timeline bar (line ~707):

```html
<button class="btn timeline-preset" onclick="setTimelinePreset('lastMonth')">Last month</button>
<button class="btn timeline-preset" onclick="setTimelinePreset('thisYear')">This year</button>
<button class="btn timeline-preset" onclick="setTimelinePreset('lastYear')">Last year</button>
<button class="btn timeline-preset active" onclick="resetTimeline()">All</button>
```

**JS function:**
```javascript
function setTimelinePreset(preset) {
  const months = activeMonths();
  if (!months.length) return;

  const now = new Date();
  let fromYear, fromMonth, toYear, toMonth;

  if (preset === 'lastMonth') {
    // Previous month
    const d = new Date(now.getFullYear(), now.getMonth() - 1, 1);
    fromYear = toYear = d.getFullYear();
    fromMonth = toMonth = d.getMonth() + 1;
  } else if (preset === 'thisYear') {
    fromYear = toYear = now.getFullYear();
    fromMonth = 1;
    toMonth = now.getMonth() + 1; // current month (or latest available)
  } else if (preset === 'lastYear') {
    fromYear = toYear = now.getFullYear() - 1;
    fromMonth = 1;
    toMonth = 12;
  }

  // Find closest available months in data
  timelineFrom = findClosestMonth(months, fromYear, fromMonth);
  timelineTo = findClosestMonth(months, toYear, toMonth);

  // Update select elements
  document.getElementById('timelineFrom').value = timelineFrom;
  document.getElementById('timelineTo').value = timelineTo;

  // Highlight active preset
  document.querySelectorAll('.timeline-preset').forEach(b => b.classList.remove('active'));
  event.target.classList.add('active');

  applyTimeline();
}
```

**Style the presets** with the same pattern as the existing "All" button, but slightly smaller. Active preset gets a subtle highlight.

**Acceptance criteria:**
- "Last month", "This year", "Last year", "All" buttons visible in timeline bar
- Each preset correctly filters the data
- Active preset is visually highlighted
- Works correctly even when data doesn't cover the full range (e.g., "This year" when only Jan–Feb data exists)

---

## Task 2.4: Build Vehicle Lookup page (hero feature)

**Status:** NOT STARTED
**Priority:** P0 — the core user-facing feature
**Files:** `index.html` (HTML, CSS, JS)

> **Important:** This is the most important page of the product. It should feel like getting a personalized car market report, not browsing a data table.

### A. Add the page structure

Add new page `page-vehicle` and sidebar nav button (between Model Comparison and Sync & Upload):

```html
<button class="nav-btn view-btn" onclick="showPage('vehicle')">
  <span class="nav-icon">🔍</span> Vehicle Lookup
</button>
```

### B. Input section — 3 tabs

```html
<div class="vehicle-input-section">
  <h2 class="section-title">Find your vehicle</h2>
  <p class="section-sub">Enter a VIN code, registration number, or select make & model</p>

  <div class="input-tabs">
    <button class="input-tab active" data-tab="vin">VIN Code</button>
    <button class="input-tab" data-tab="reg">Reg Number</button>
    <button class="input-tab" data-tab="manual">Make / Model</button>
  </div>

  <!-- VIN tab -->
  <div class="input-panel active" id="panel-vin">
    <input id="vinInput" placeholder="e.g. WBAPH5C55BA123456" maxlength="17" />
    <button onclick="lookupVIN()">Decode</button>
  </div>

  <!-- Reg tab -->
  <div class="input-panel" id="panel-reg">
    <input id="regInput" placeholder="e.g. 123 ABC" />
    <button onclick="lookupReg()">Look up</button>
    <p class="coming-soon">Registration lookup available Q2 2026</p>
  </div>

  <!-- Manual tab -->
  <div class="input-panel" id="panel-manual">
    <!-- Reuse combobox pattern: Make → Model → Configuration -->
  </div>
</div>
```

### C. Vehicle report section (the key part)

After the user enters a vehicle, show a **personalized report**:

```html
<div id="vehicleReport" style="display:none">

  <!-- 1. Vehicle identity card -->
  <div class="vehicle-hero">
    <div class="vehicle-hero-make">BMW</div>
    <div class="vehicle-hero-model">5 SERIES</div>
    <div class="vehicle-hero-detail">2011 · VIN: WBAPH5C55BA123456</div>
  </div>

  <!-- 2. Popularity stats — KEY FEATURE -->
  <div class="stats-grid">
    <!-- Stat 1: Make popularity -->
    <div class="stat-pill">
      <div class="stat-pill-label">Make popularity</div>
      <div class="stat-pill-value">#2</div>
      <div class="stat-pill-sub">of 1763 makes in Järelturg</div>
    </div>
    <!-- Stat 2: Model popularity within ALL makes -->
    <div class="stat-pill">
      <div class="stat-pill-label">Model rank (market)</div>
      <div class="stat-pill-value">#7</div>
      <div class="stat-pill-sub">of all models traded</div>
    </div>
    <!-- Stat 3: Model popularity within THIS make -->
    <div class="stat-pill">
      <div class="stat-pill-label">Model rank (within BMW)</div>
      <div class="stat-pill-value">#2</div>
      <div class="stat-pill-sub">of 45 BMW models</div>
    </div>
    <!-- Stat 4: Total transactions for this model -->
    <div class="stat-pill">
      <div class="stat-pill-label">Transactions</div>
      <div class="stat-pill-value">4,521</div>
      <div class="stat-pill-sub">in selected period</div>
    </div>
  </div>

  <!-- 3. Insight text — INTERPRETS the data -->
  <div class="insight-card">
    <p class="insight-text">
      <strong>BMW 5 SERIES</strong> is the 2nd most popular BMW model and ranks #7 overall
      in the Estonian used car market. With 4,521 transactions in the last 26 months,
      it's a <strong>high-liquidity</strong> choice — easy to find a good deal and easy to resell.
    </p>
  </div>

  <!-- 4. Configuration comparison (if config selected) -->
  <div id="configComparison" style="display:none">
    <h3 class="section-title">Configuration popularity</h3>
    <p class="section-sub">How does your chosen configuration compare to others?</p>
    <!-- Horizontal bar chart: each config as a bar, selected one highlighted -->
    <canvas id="chartConfigPopularity"></canvas>
  </div>

  <!-- 5. Monthly trend chart -->
  <div class="card">
    <h3 class="card-title">Monthly transaction volume</h3>
    <p class="card-sub">BMW 5 SERIES · Järelturg</p>
    <canvas id="chartVehicleTrend"></canvas>
  </div>

  <!-- 6. Production year distribution -->
  <div class="card">
    <h3 class="card-title">Production year distribution</h3>
    <p class="card-sub">Which model years are most traded?</p>
    <canvas id="chartVehicleProdYear"></canvas>
  </div>

  <!-- 7. Cross-category comparison -->
  <div class="card">
    <h3 class="card-title">This model across market segments</h3>
    <p class="card-sub">Järelturg vs New vs Import</p>
    <!-- Show this model's presence in all 3 categories -->
    <canvas id="chartVehicleCrossCategory"></canvas>
  </div>
</div>
```

### D. JavaScript implementation

**New functions to add:**

```javascript
// 1. VIN lookup flow
function lookupVIN() {
  const vin = document.getElementById('vinInput').value.trim().toUpperCase();
  const result = decodeVIN(vin);
  if (!result.isValid) { showError(result.error); return; }
  showVehicleReport({ make: result.make, model: null, variant: null, year: result.modelYear, vin });
}

// 2. Manual selection flow
function lookupManual() {
  const make = selectedMake, model = selectedModel, variant = selectedVariant;
  showVehicleReport({ make, model, variant, year: null, vin: null });
}

// 3. Core report renderer
function showVehicleReport({ make, model, variant, year, vin }) {
  document.getElementById('vehicleReport').style.display = 'block';

  // Calculate ALL popularity ranks
  const months = filteredMonths(); // use ALL categories for cross-category

  // a) Make rank (across all makes)
  const makeRanks = rankMakes(months);
  const makeRank = makeRanks.findIndex(m => m.make === make) + 1;

  // b) Model rank (across ALL models of ALL makes)
  const allModelRanks = rankAllModels(months);
  const modelRankOverall = allModelRanks.findIndex(m => m.make === make && m.model === model) + 1;

  // c) Model rank (within this make)
  const makeModelRanks = rankModelsForMake(months, make);
  const modelRankWithinMake = makeModelRanks.findIndex(m => m.model === model) + 1;

  // d) If configuration selected: rank configs within this model
  if (variant) {
    const configRanks = rankConfigsForModel(months, make, model);
    renderConfigComparison(configRanks, variant);
  }

  // e) Generate insight text
  renderInsight(make, model, makeRank, modelRankOverall, totalTxs);

  // f) Render charts
  renderVehicleTrendChart(months, make, model, variant);
  renderVehicleProdYearChart(months, make, model, variant);
  renderCrossCategoryChart(make, model);
}

// 4. Ranking helpers
function rankAllModels(months) {
  // Aggregate all rows by make+model, return sorted by count desc
  const map = {};
  months.forEach(m => m.rows.forEach(r => {
    const key = r.make + '|' + r.model;
    map[key] = (map[key] || { make: r.make, model: r.model, count: 0 });
    map[key].count += r.count;
  }));
  return Object.values(map).sort((a, b) => b.count - a.count);
}

function rankModelsForMake(months, make) {
  // Same but filtered to one make
  return rankAllModels(months).filter(m => m.make === make);
}

function rankConfigsForModel(months, make, model) {
  // Aggregate by variant within make+model
  const map = {};
  months.forEach(m => m.rows.forEach(r => {
    if (r.make !== make || r.model !== model) return;
    const v = r.variant || r.fullModel || 'Unknown';
    map[v] = (map[v] || { variant: v, count: 0 });
    map[v].count += r.count;
  }));
  return Object.values(map).sort((a, b) => b.count - a.count);
}

// 5. Configuration comparison chart
function renderConfigComparison(configRanks, selectedVariant) {
  // Horizontal bar chart
  // Selected variant highlighted in gold, others in gray
  // Shows top 10 configurations + selected one if not in top 10
  document.getElementById('configComparison').style.display = 'block';
  // ... Chart.js horizontal bar implementation
}

// 6. Insight text generator
function renderInsight(make, model, makeRank, modelRank, txCount) {
  const liquidity = txCount > 3000 ? 'high-liquidity' : txCount > 1000 ? 'moderate-liquidity' : 'niche';
  const liquidityText = {
    'high-liquidity': 'Easy to find a good deal and easy to resell.',
    'moderate-liquidity': 'Reasonable availability on the market.',
    'niche': 'Limited availability — finding the right one may take time.'
  };
  // Build natural language insight
  // ...
}

// 7. Cross-category chart
function renderCrossCategoryChart(make, model) {
  // Show this model's transaction count in jarelturg, newCars, imports
  // Stacked or grouped bar chart across months
  // Helps user understand: is this model mostly traded used or bought new?
}
```

### E. Make/Model selector in Vehicle Lookup

Reuse the combobox pattern from Model Comparison, but with ONE set of dropdowns (Make → Model → Configuration). When the user selects:
- **Make only:** Show make-level report
- **Make + Model:** Show model-level report with model rank stats
- **Make + Model + Config:** Show full report with configuration comparison

### F. Style the report

- **Vehicle hero section**: Large make name, model name below, subtle detail line
- **Insight card**: Light yellow/gold background (`var(--gold2)`), left gold border, 14px font
- **Card titles**: Use sentence case (not ALL CAPS MONOSPACE)
  - "Monthly transaction volume" not "MONTHLY TRANSACTION VOLUME"
  - "Production year distribution" not "PRODUCTION YEAR DISTRIBUTION"

**Acceptance criteria:**
- Vehicle Lookup page accessible from sidebar
- VIN decode shows make-level AND attempts model matching from data
- Manual selection works with Make → Model → Config cascade
- Stat pills show: make rank, model rank (overall), model rank (within make), total transactions
- Configuration comparison chart appears when a config is selected
- Insight text generated dynamically based on data
- Cross-category chart shows the model across järelturg, new, import
- Sentence-case headers throughout
- Reg number tab shows "Available Q2 2026" placeholder

---

## Task 2.5: Integrate andmed.eesti.ee OpenData API

**Status:** NOT STARTED
**Priority:** P1 — replaces fragile URL-guessing in parse.py
**Files:** `parse.py`

> ✅ **PM-verified API details (2026-03-25):** The API is live, public, no auth needed.

**Confirmed working API endpoints:**
- Search: `GET https://andmed.eesti.ee/api/datasets?search=infoleht`
- Get dataset: `GET https://andmed.eesti.ee/api/datasets/1401f275-5b59-4bf9-b98a-61ea15bc95ea`
- Download file: `GET https://andmed.eesti.ee/api/v2/datasets/9d2f4b4d-9eab-45f7-97b2-543600621aff/distribution/{distributionId}/file`
- **No API key required** — rate limit 2000 req/window

**Dataset details:**
- Dataset ID: `1401f275-5b59-4bf9-b98a-61ea15bc95ea`
- Dataset Identifier: `9d2f4b4d-9eab-45f7-97b2-543600621aff`
- Currently available: Nov 2025, Dec 2025, Jan 2026 (3 months)

**Implementation flow:**
```
Step 1: GET /api/datasets/1401f275-5b59-4bf9-b98a-61ea15bc95ea
Step 2: Parse response.distributions[] — each has titleEt ("Infoleht YYYY-MM") and accessUrls[0]
Step 3: Find latest month by sorting distributions on titleEt
Step 4: Download via accessUrls[0] (returns XLSX binary)
Step 5: Fallback to current URL-guessing if API fails
```

**Acceptance criteria:**
- parse.py can discover and download infoleht files via the Open Data API
- Fallback to URL-guessing still works if API is unavailable
- Data output is identical to current (same schema, same parsing)

---

## Task 2.6: UI polish pass

**Status:** NOT STARTED
**Priority:** P2 — do after core features work
**Files:** `index.html` (CSS, HTML)

**Quick wins to apply across the whole app:**

### A. Sentence-case headers everywhere
Replace all `ALL CAPS MONOSPACE` chart/section headers with sentence case:
- `TOP 5 MAKES — MONTHLY TRANSACTION VOLUME` → `Top 5 makes — Monthly transaction volume`
- `MARKET SHARE BY MAKE` → `Market share by make`
- `MONTHLY TOTAL TRANSACTIONS` → `Monthly total transactions`
- `TOP MAKES RANKED` → `Top makes ranked`
- `SELECT UP TO 5 MAKE · MODEL · CONFIGURATION COMBINATIONS` → `Select up to 5 models to compare`

Change the CSS for card titles:
```css
.card-title {
  font-family: var(--font);        /* was var(--mono) */
  font-size: 15px;                 /* was 11px */
  font-weight: 600;                /* was 700 */
  letter-spacing: 0;               /* was 0.08em */
  text-transform: none;            /* was uppercase */
}
.card-sub {
  font-family: var(--font);        /* was var(--mono) */
  font-size: 12px;
  color: var(--muted);
  text-transform: none;            /* was uppercase */
}
```

### B. Collapse Model Comparison to 2 default slots
In `buildModelSlots()`, only render 2 slots by default. Add an "Add model +" button that adds up to 3 more.

### C. Hide Sync & Upload
Move it out of the main nav. Add a small "⚙️" icon button in the sidebar bottom (next to sync status) that opens the sync page. Don't list it as a primary view.

### D. Add insight text to Overview page
Below the stats-grid, add a brief auto-generated insight:
```
"Volkswagen leads the Estonian used car market with 10.1% share,
followed by BMW (9.4%) and Toyota (7.2%)."
```

**Acceptance criteria:**
- All headers in sentence case
- Model Comparison starts with 2 slots + "Add model" button
- Sync page hidden from main nav
- At least one insight text on Overview page

---

## Task 2.7: Update documentation

**Status:** NOT STARTED
**Priority:** P1 — do after Tasks 2.1-2.6 are complete
**Files:** `docs/rpd.md`, `docs/data-schema.md`, `docs/buildplan.md`, `docs/architecture.md`

**What to do:**
1. In `docs/rpd.md`: Update FR statuses for completed features
2. In `docs/data-schema.md`: Document any new data structures (ranking functions, etc.)
3. In `docs/buildplan.md`: Check off Sprint 2 deliverables
4. In `docs/architecture.md`: Document the andmed.eesti.ee API integration
5. In `docs/ui-review.md`: Mark resolved issues

---

## How to Work

1. Pick the next NOT STARTED task **in the order listed above**
2. Read `index.html` fully to understand current code patterns
3. Implement the changes
4. Update this file: change status to DONE with notes on what was implemented
5. Commit with a descriptive message
6. Move to the next task

**Key code locations in `index.html`:**
- CSS: lines 8–636
- Sidebar HTML: lines 640–688
- Main content HTML: lines 690–871
- JS globals: lines 877–882 (`db`, `activeCategory`, `charts`, `slots`)
- `switchCategory()`: line 1009
- `showPage()`: line 1038
- `renderOverview()`: line 1094
- `buildModelSlots()`: line 1371
- Chart helper `chartOpts()`: check for responsive Chart.js options
- `activeMonths()`: line 904 (returns months for current category)
- `filteredMonths()`: line 921 (applies timeline filter)
- `allMakes()`: line 1257
- `modelsForMake()`: line 1263
- `variantsForMakeModel()`: line 1269

**Data structure:**
```javascript
db = { jarelturg: [], newCars: [], imports: [] }
// Each category array contains month objects:
{ year: 2024, month: 1, label: "Jan 2024", rows: [
  { make: "BMW", model: "5 SERIES", variant: "520D", fullModel: "5 SERIES 520D", prodYear: 2018, count: 12 }
]}
```

## Data Source Reference

**andmed.eesti.ee API (✅ VERIFIED WORKING):**
- Swagger docs: https://andmed.eesti.ee/api/dataset-docs/
- Dataset page: https://andmed.eesti.ee/datasets/infoleht-(esmaste-ja-uute-soidukite-statistika)
- **Search:** `GET https://andmed.eesti.ee/api/datasets?search=infoleht` (no auth)
- **Get dataset:** `GET https://andmed.eesti.ee/api/datasets/1401f275-5b59-4bf9-b98a-61ea15bc95ea`
- **Download:** `GET https://andmed.eesti.ee/api/v2/datasets/9d2f4b4d-9eab-45f7-97b2-543600621aff/distribution/{distributionId}/file`
- Rate limit: 2000 req/window
- Current files: INFOLEHT-112025.xlsx, INFOLEHT-122025.xlsx, INFOLEHT-012026.xlsx

**Statistikaamet API (andmed.stat.ee):**
- Table TS322: First registrations by month
- API docs: https://andmed.stat.ee/abi/api-juhend.pdf

**mntstat.ee:**
- Public vehicle database: https://www.mntstat.ee/
- 829K+ vehicle records, web interface only — no REST API

**Transpordiamet infoleht files (fallback):**
- Download pattern: `https://www.transpordiamet.ee/sites/default/files/documents/{YYYY-MM}/INFOLEHT-{MMYYYY}.xlsx`
