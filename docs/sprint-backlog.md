# Sprint 2: API Integration & Vehicle Lookup

> **Goal:** Replace URL-guessing with proper API data fetching. Add vehicle lookup by VIN and registration number.
> **Sprint dates:** 2026-03-25 — 2026-04-08
> **References:** Phase 2 in `buildplan.md`, FR-006, FR-010, FR-011, FR-015, FR-016

---

## Tasks

### Task 2.1: Integrate andmed.eesti.ee OpenData API
**Status:** DONE
**Priority:** P0
**Files:** `parse.py`

**Changes made:**
- Added `try_opendata_api()` function that searches avaandmed.eesti.ee for infoleht dataset and downloads it
- API key configured via `OPENDATA_API_KEY` environment variable (register at avaandmed.eesti.ee)
- `fetch_month()` tries API first, falls back to URL-guessing if API unavailable/unconfigured
- Existing URL-guessing logic preserved as reliable fallback
- Note: API requires registration for API key — documented in code comments

---

### Task 2.2: VIN decode logic (client-side)
**Status:** DONE
**Priority:** P1
**Files:** `index.html`

**Changes made:**
- Added `WMI_MAP` with 70+ WMI codes covering top 20+ Estonian-market makes (BMW, VW, Mercedes, Audi, Toyota, Volvo, Skoda, Tesla, etc.)
- Added `YEAR_CODES` mapping for model year decode (position 10)
- `decodeVIN(vin)` function returns `{ isValid, vin, wmi, vds, make, modelYear, yearCode, plant, serial }`
- Validates: 17 chars, no I/O/Q, alphanumeric only
- German, Japanese, Korean, French, Italian, British, American, Czech, Swedish, Romanian makes covered

---

### Task 2.3: Vehicle lookup via mntstat.ee scraping
**Status:** DONE
**Priority:** P1
**Files:** new `scrape_vehicle.py`

**Changes made:**
- Created `scrape_vehicle.py` with registration number lookup via mntstat.ee
- Discovered actual form fields: `make[]`, `model[]`, `from`, `to` (not `automarg`/`aasta_min`)
- Table columns: Staatus, Kokku, Mark, Mudel, Keretüüp, Aasta, Värv, Mootoritüüp, Käigukast, Kw, CC, Kg, Maakond
- `search_by_reg(reg)` returns structured vehicle data dict
- `search_by_filters(make, model, year_from, year_to)` for filtered searches
- `--json` flag for machine-readable output
- Tested: `python scrape_vehicle.py 100BMW` returns correct BMW IX XDRIVE40 data
- Handles not-found and errors cleanly

---

### Task 2.4: Vehicle detail panel UI
**Status:** DONE
**Priority:** P2
**Files:** `index.html`

**Changes made:**
- Added "Vehicle Lookup" page (id: `page-vehicle`) with sidebar nav button
- 3 input tabs: VIN, Reg Number (disabled/coming soon), Make/Model selector
- VIN tab: text input, decode button, calls `decodeVIN()` and shows market data for decoded make
- Reg Number tab: placeholder UI with "Coming soon" message
- Make/Model tab: reuses `createCombobox()` pattern with 3-field selector (Maker/Model/Configuration)
- Vehicle info card with grid layout showing decoded/selected vehicle details
- Market context section with:
  - Stats: total transactions, market share, popularity rank, avg/month
  - Monthly transaction volume line chart
  - Production year distribution bar chart
- CSS: `.vlookup-tab`, `.vehicle-info-grid`, `.vehicle-info-item` styles
- `showPage()` and `pageTitlesForCategory()` updated for vehicle page

---

### Task 2.5: Update documentation
**Status:** DONE
**Priority:** P1
**Files:** `docs/sprint-backlog.md`

**Changes made:**
- Sprint backlog updated with all task statuses and implementation details

---

## Data Source Reference

**avaandmed.eesti.ee API:**
- Base URL: `https://avaandmed.eesti.ee/api/v1/`
- Auth: API key (register at avaandmed.eesti.ee), pass as `?apiKey=KEY`
- Search: `GET /datasets/search?q=infoleht&apiKey=KEY`
- Download: `GET /datasets/{id}/download?format=xlsx&apiKey=KEY`

**mntstat.ee:**
- Public vehicle database: https://www.mntstat.ee/
- Search: `GET /search.php?reg_nr=XXX` or `GET /search.php?make[]=BMW&from=2020&to=2025`
- Model dropdown: `POST /data.php` with `aid=MAKE&type=model`
- Table columns: Staatus, Kokku, Mark, Mudel, Keretüüp, Aasta, Värv, Mootoritüüp, Käigukast, Kw, CC, Kg, Maakond
- 829K+ vehicles, data as of March 2025
