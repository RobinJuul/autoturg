# Build Plan

> Phased delivery roadmap with milestones and dependencies.

**Last updated:** 2026-03-25

---

## Gantt Chart

```mermaid
gantt
    title Autoturg Build Plan
    dateFormat YYYY-MM-DD
    axisFormat %b %Y

    section Phase 0: Foundation ✓
    Järelturg dashboard                 :done, p0a, 2024-01-01, 2026-03-23
    parse.py data pipeline              :done, p0b, 2026-03-01, 2026-03-15
    GitHub Actions monthly cron         :done, p0c, 2026-03-15, 2026-03-20
    Model comparison (5 slots)          :done, p0d, 2026-03-20, 2026-03-23

    section Phase 1: Expand Data Sources ✓
    Parse new-car registration sheet    :done, p1a, 2026-03-24, 1d
    Multi-category data.json schema     :done, p1c, 2026-03-24, 1d
    UI: category tabs & navigation      :done, p1d, 2026-03-24, 1d
    Combined cross-category overview    :done, p1e, 2026-03-24, 1d

    section Sprint 2: API & VIN Foundation
    andmed.eesti.ee OpenData API        :active, s2a, 2026-03-25, 7d
    VIN decode (client-side)            :active, s2b, 2026-03-25, 5d
    mntstat.ee vehicle scraper          :active, s2c, 2026-03-25, 7d
    Vehicle detail panel UI             :s2d, after s2b, 5d
    Sprint 2 docs update               :s2e, after s2d, 1d
    Sprint 2 done                       :milestone, s2m, 2026-04-08, 0d

    section Sprint 3: Teabevarav & Enrichment
    Research Teabevarav API             :s3a, 2026-04-08, 5d
    Build Teabevarav API client         :s3b, after s3a, 7d
    Integrate into vehicle panel        :s3c, after s3b, 3d
    Statistikaamet API (TS322)          :s3d, 2026-04-08, 7d
    Serverless proxy (CF Worker)        :s3e, after s3b, 5d
    Sprint 3 done                       :milestone, s3m, 2026-04-22, 0d

    section Sprint 4: auto24.ee Pricing
    Research auto24 data access         :s4a, 2026-04-22, 5d
    Build auto24 data collector         :s4b, after s4a, 7d
    Price normalization pipeline        :s4c, after s4b, 5d
    Price range visualizations          :s4d, after s4c, 5d
    Sprint 4 done                       :milestone, s4m, 2026-05-06, 0d

    section Sprint 5: International Pricing
    mobile.de API integration           :crit, s5a, 2026-05-06, 10d
    AutoScout24 API integration         :crit, s5b, 2026-05-06, 10d
    Cross-platform comparison           :s5c, after s5a, 5d
    autoportaal.ee collection           :s5d, 2026-05-06, 7d
    Sprint 5 done                       :milestone, s5m, 2026-05-20, 0d

    section Sprint 6: Depreciation
    Depreciation curve computation      :s6a, 2026-05-20, 7d
    Depreciation charts UI              :s6b, after s6a, 5d
    Value retention indicator           :s6c, after s6b, 3d
    Cross-model comparison              :s6d, after s6c, 3d
    Sprint 6 done                       :milestone, s6m, 2026-06-03, 0d

    section Sprint 7+: Polish & Scale
    Performance optimization            :s7a, after s6m, 14d
    Mobile responsive redesign          :s7b, after s7a, 7d
    Architecture reassessment           :milestone, s7m, after s7b, 0d
```

---

## Phase 0: Foundation (DONE)

**Objective:** Build a working dashboard for Estonian järelturg transaction data.

**Deliverables:**
- [x] Monthly overview with top 5 makes, market share donut, totals bar chart, sortable table
- [x] Model comparison with up to 5 make/model/variant slots
- [x] Production year distribution charts
- [x] Manual .xlsx upload + auto-fetch from Transpordiamet
- [x] parse.py server-side pipeline with GitHub Actions cron
- [x] 26 months of historical data (Jan 2024 – Feb 2026)

**References:** FR-001, FR-002, FR-003, FR-014

---

## Phase 1: Expand Data Sources (DONE)

**Objective:** Cover all 3 market categories — new cars, imports, and järelturg.

**Completed:** 2026-03-24 (Sprint 1)

**Deliverables:**
- [x] parse.py refactored with `find_sheet_by_category()` and `SHEET_KEYWORDS` dict
- [x] data.json schema changed to `{ jarelturg[], newCars[], imports[] }`
- [x] 26 months järelturg + 26 months newCars data populated
- [x] UI: sidebar category buttons (Järelturg, Uued sõidukid, Import, Kogu turg)
- [x] Combined "Kogu turg" view with stacked bar chart
- [x] GitHub Actions pipeline updated for all categories

**Notes:**
- Import sheets not found in infoleht files — imports array stays empty. Import data needs alternative source.
- New cars parsed from "Sõidu-uus" sheet via keyword matching on "uus"/"uued"
- Auto-migration from old `{ months[] }` format to new schema in both Python and JS

**References:** FR-004 (DONE), FR-005 (Blocked), US-16, US-17, US-18

---

## Phase 2: API Integration & Data Pipeline

**Objective:** Replace URL-guessing with proper API access, add vehicle lookup via VIN/registration number.

**Prerequisites:** Phase 1 complete.

**Tasks:**
1. Integrate andmed.eesti.ee OpenData API for programmatic infoleht access
2. Integrate Statistikaamet API (andmed.stat.ee) for TS322 first-registration data
3. Replace URL candidate guessing in parse.py with API-driven data fetching
4. Build client-side VIN decode (WMI → make, VDS → model/year)
5. Build mntstat.ee scraper for vehicle details by registration number
6. Create vehicle detail panel in UI showing specs + market context
7. Build input UI: make/model selector, VIN field, registration number field
8. Apply for AVP access from Transpordiamet (critical path, long lead time)

**Deliverables:**
- [ ] parse.py fetches infoleht data via andmed.eesti.ee API instead of URL guessing
- [ ] Statistikaamet TS322 data integrated for first-registration statistics
- [ ] VIN input decodes make, model, year client-side
- [ ] mntstat.ee scraper returns vehicle specs by registration number
- [ ] Vehicle detail panel shows specs + transaction history + market position
- [ ] Input method switcher (dropdown, VIN, reg number)

**Risks:**
- andmed.eesti.ee API may require registration for API key
- mntstat.ee scraping may break if site layout changes — need resilient selectors
- AVP application takes weeks — scraping is the interim fallback
- Rate limits on mntstat.ee unknown

**Definition of Done:** Data pipeline uses APIs instead of URL guessing. User can enter a VIN or registration number and see vehicle details plus market context.

**References:** FR-006, FR-010, FR-011, FR-015, FR-016, US-13, US-14, US-15

---

## Phase 3: Pricing Intelligence

**Objective:** Show current asking prices and price distributions from Estonian and European marketplaces.

**Prerequisites:** Phase 2 complete. API accounts for mobile.de and AutoScout24 obtained.

**Tasks:**
1. Integrate mobile.de Search API (HTTP Basic auth)
2. Integrate AutoScout24 Listing API (OAuth)
3. Build auto24.ee data collection (scraping or partnership — explore both)
4. Build price normalization pipeline (currency, make/model matching across sources)
5. Create price range visualizations (box plots or violin plots)
6. Show current asking prices in vehicle detail panel

**Deliverables:**
- [ ] Pricing data from 2+ sources ingested and stored in prices.json
- [ ] Price normalization across sources (EUR, canonical make/model names)
- [ ] Price range visualization for any make/model/year
- [ ] Current listings shown alongside market statistics
- [ ] Weekly pricing updates via GitHub Actions

**Risks:**
- mobile.de API account approval timeline unknown
- auto24.ee has no API — scraping may violate ToS, partnership may take time
- Price data volume could grow quickly — monitor storage size

**Definition of Done:** User can see price ranges for a model, compare asking prices across platforms, and understand where a specific price falls relative to the market.

**References:** FR-007, FR-008, FR-009, FR-013, US-10, US-11, US-12

---

## Phase 4: Depreciation Analysis

**Objective:** Show how vehicles lose value over time and help buyers understand long-term cost of ownership.

**Prerequisites:** Phase 3 complete (requires pricing data across production years).

**Tasks:**
1. Compute depreciation curves from historical pricing data (median price by vehicle age)
2. Build depreciation chart UI (age vs. price with confidence bands)
3. Create "value retention indicator" — simple score showing how well a model holds value
4. Enable cross-model depreciation comparison (overlay multiple models)

**Deliverables:**
- [ ] Depreciation curves for models with sufficient pricing data
- [ ] Interactive depreciation chart with age-based view
- [ ] Value retention score/indicator per model
- [ ] Side-by-side depreciation comparison for competing models

**Risks:**
- Requires sufficient pricing data across production years — some models may have sparse data
- Depreciation varies by variant/configuration — decide granularity level

**Definition of Done:** User can view depreciation curve for any model with 10+ data points, compare depreciation between models, and see a clear value retention indicator.

**References:** FR-012, US-08, US-09

---

## Phase 5: Polish & Scale

**Objective:** Optimize performance, improve mobile experience, and reassess architecture for continued growth.

**Prerequisites:** Phases 1-4 complete.

**Tasks:**
1. Performance audit: lazy loading, code splitting (if multi-file), pagination for large datasets
2. Mobile responsive improvements: collapsible sidebar, touch-friendly charts, responsive grids
3. Architecture reassessment: evaluate whether to introduce a component framework, backend, or database

**Deliverables:**
- [ ] Page load under 2s on broadband for all views
- [ ] Usable mobile experience for all core features
- [ ] Architecture decision documented: continue as-is or evolve stack

**Definition of Done:** Lighthouse performance score > 80. Core flows work on mobile. Architecture decision made and documented in ADR.

**References:** NFR-01, NFR-06

---

## Dependency Map

```
Sprint 0–1 (DONE)
    └── Sprint 2: API & VIN Foundation
            └── Sprint 3: Teabevarav API & Enrichment
                    └── Sprint 4: auto24.ee Pricing ──────┐
                        Sprint 5: International Pricing ───┤ (can run in parallel)
                                                           └── Sprint 6: Depreciation
                                                                   └── Sprint 7+: Polish
```

## Critical Path Items

| Blocker | Gates | Owner Action | Deadline |
|---------|-------|-------------|----------|
| Teabevarav API access | Sprint 3 | Check https://abi.ria.ee/teabevarav/ for registration | This week |
| auto24.ee data access | Sprint 4 | Email info@auto24.ee about partnership | This week |
| mobile.de API account | Sprint 5 | Apply at services.mobile.de | Now (weeks lead time) |
| AutoScout24 API access | Sprint 5 | Apply at developer portal | Now (weeks lead time) |

**Note:** Sprints 2–3 have NO external blockers. Sprint 5 can run in parallel with Sprint 4 if API credentials arrive. Sprint 6 needs at least one pricing source (Sprint 4 or 5).
