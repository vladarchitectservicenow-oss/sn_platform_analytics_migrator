# Test Suite SOP: ServiceNow Platform Analytics Migrator (sn_platform_analytics_migrator)

**Author:** Vladimir Kapustin
**License:** AGPL-3.0
**Product:** sn_platform_analytics_migrator
**Version:** 1.0.0
**Test Framework:** Python `unittest` (primary) / `pytest` (compatible)
**SOP Version:** 2.0
**Last Updated:** 2026-06-02

---

## 1. Objective

This SOP defines the standard test suite for the ServiceNow Platform Analytics Migrator. The suite ensures correct behavior for: plugin detection, report enumeration, type mapping, PA config generation, JSON export structure, HTML rendering, full migration pipeline, dashboard fetching, error handling, and supported/unsupported visualization handling.

---

## 2. Test Scenarios (12 total: T01–T12)

### T01 — Plugin Active Check
- **ID:** T01
- **Priority:** P0
- **Category:** Platform Health
- **Description:** Verify that `check_plugin()` correctly detects when Platform Analytics or Performance Analytics plugins are active on the target instance.
- **Preconditions:** ServiceNow instance with PA/PA plugins active.
- **Test Steps:**
  1. Mock `_get()` to return `[{"name": "com.snc.platform_analytics", "active": "true"}]`.
  2. Call `check_plugin()`.
  3. Assert `result["plugin_active"] == True`.
  4. Assert `len(result["plugins"]) >= 1`.
- **Expected Result:** `plugin_active: true` with non-empty plugins list.
- **Status:** ✅ Implemented as `test_plugin_check`.

### T02 — Enumerate Core UI Reports (sys_report)
- **ID:** T02
- **Priority:** P0
- **Category:** Data Acquisition
- **Description:** Verify that `list_reports()` correctly fetches and returns legacy Core UI reports from the `sys_report` table.
- **Preconditions:** Instance has at least one active sys_report record.
- **Test Steps:**
  1. Mock `_get()` to return a list of report dicts with `sys_id`, `title`, `type`, `table`, `filter`, `group_by`.
  2. Call `list_reports(limit=10)`.
  3. Assert returned list is not empty.
  4. Assert each record has `sys_id` and `title` fields.
- **Expected Result:** Reports list populated with valid report objects.
- **Status:** ✅ Covered by T05/T07 mock tests.

### T03 — Enumerate PA Dashboards (pa_dashboards)
- **ID:** T03
- **Priority:** P0
- **Category:** Data Acquisition
- **Description:** Verify that `list_dashboards()` correctly fetches Performance Analytics dashboards.
- **Preconditions:** Instance has at least one dashboard.
- **Test Steps:**
  1. Mock `_get()` to return `[{"sys_id": "d1", "name": "Exec Dashboard", "active": "true"}]`.
  2. Call `list_dashboards()`.
  3. Assert returned list length >= 1.
  4. Assert each record has `sys_id` and `name` fields.
- **Expected Result:** Dashboards enumerated correctly.
- **Status:** ✅ Implemented as `test_dashboard_fetch`.

### T04 — Map Supported Report Type (bar)
- **ID:** T04
- **Priority:** P0
- **Category:** Data Mapping
- **Description:** Verify that `map_report()` correctly maps a supported report type (e.g., `bar`) and marks it as supported with no issues.
- **Preconditions:** Report with type `bar`, valid table.
- **Test Steps:**
  1. Call `map_report({"sys_id": "1", "title": "Tickets", "type": "bar", "table": "incident", "filter": "", "group_by": "category"})`.
  2. Assert `result.supported == True`.
  3. Assert `result.target_type == "bar"`.
  4. Assert `result.issues == []`.
- **Expected Result:** Supported bar report mapped cleanly.
- **Status:** ✅ Implemented as `test_map_supported_bar`.

### T05 — Map Unsupported Report Type (gauge)
- **ID:** T05
- **Priority:** P0
- **Category:** Data Mapping
- **Description:** Verify that `map_report()` correctly marks an unsupported report type (e.g., `gauge`) with `supported: false` and the appropriate issue message.
- **Preconditions:** Report with type `gauge`.
- **Test Steps:**
  1. Call `map_report({"sys_id": "2", "title": "Gauge", "type": "gauge", "table": "", "filter": ""})`.
  2. Assert `result.supported == False`.
  3. Assert any issue contains "not supported in Platform Analytics".
- **Expected Result:** Unsupported gauge flagged with issue.
- **Status:** ✅ Implemented as `test_map_unsupported_gauge`.

### T06 — Detect Missing Target Table
- **ID:** T06
- **Priority:** P1
- **Category:** Data Validation
- **Description:** Verify that `map_report()` flags reports with empty/missing `table` field.
- **Preconditions:** Report with `table: ""`.
- **Test Steps:**
  1. Call `map_report({"sys_id": "3", "title": "NoTable", "type": "list", "table": "", "filter": ""})`.
  2. Assert any issue contains "Missing target table".
- **Expected Result:** Missing table detected and flagged.
- **Status:** ✅ Implemented as `test_map_missing_table`.

### T07 — Generate PA Configuration
- **ID:** T07
- **Priority:** P1
- **Category:** Export
- **Description:** Verify that `generate_pa_config()` produces correct PA-compatible JSON from a `ReportMapping`.
- **Preconditions:** Valid `ReportMapping` instance.
- **Test Steps:**
  1. Create `ReportMapping(sys_id="1", name="T", source_type="bar", target_type="bar", supported=True, table="incident", filter="", group_by="category", issues=[])`.
  2. Call `generate_pa_config(mapping)`.
  3. Assert `cfg["type"] == "bar"`.
  4. Assert `cfg["table"] == "incident"`.
  5. Assert `cfg["source"] == "legacy_migration"`.
  6. Assert `cfg["supported"] == True`.
- **Expected Result:** Correct PA JSON config structure.
- **Status:** ✅ Implemented as `test_generate_pa_config`.

### T08 — Full Pipeline Mock
- **ID:** T08
- **Priority:** P1
- **Category:** Integration
- **Description:** Verify the end-to-end migration pipeline: `run()` → `save_report()` → file output (both JSON and HTML) with mocked data.
- **Preconditions:** Mocked `_get()` returning valid reports.
- **Test Steps:**
  1. Mock `_get()` to return a supported + unsupported report.
  2. Call `run(limit=10)` to get `MigrationReport`.
  3. Call `save_report(report)`.
  4. Assert JSON file exists.
  5. Assert HTML file exists.
  6. Clean up output files.
- **Expected Result:** Both JSON and HTML files generated; pipeline complete.
- **Status:** ✅ Implemented as `test_full_pipeline_mock`.

### T09 — HTML Report Rendering
- **ID:** T09
- **Priority:** P2
- **Category:** Export
- **Description:** Verify that `_render_html()` generates valid HTML containing "YES" for supported reports and "NO" for unsupported ones.
- **Preconditions:** `MigrationReport` with one supported and one unsupported mapping.
- **Test Steps:**
  1. Create `MigrationReport` with 2 mappings (bar=supported, gauge=unsupported).
  2. Call `_render_html(report)`.
  3. Assert HTML contains "Mappable".
  4. Assert HTML contains "YES" (green) for supported.
  5. Assert HTML contains "NO" (red) for unsupported.
- **Expected Result:** HTML table with green/red status indicators.
- **Status:** ✅ Implemented as `test_html_render`.

### T10 — Migration Report Metrics
- **ID:** T10
- **Priority:** P2
- **Category:** Data Integrity
- **Description:** Verify that `run()` correctly computes `total_reports`, `mappable`, and `unmappable` counts.
- **Preconditions:** Mocked `_get()` returning 1 supported report.
- **Test Steps:**
  1. Mock `_get()` to return `[{"sys_id": "1", "title": "R1", "type": "bar", "table": "incident"}]`.
  2. Call `run(limit=10)`.
  3. Assert `report.total_reports == 1`.
  4. Assert `report.mappable == 1`.
  5. Assert `report.unmappable == 0`.
- **Expected Result:** Correct metric computation.
- **Status:** ✅ Implemented as `test_report_metrics`.

### T11 — Export JSON Structure
- **ID:** T11
- **Priority:** P2
- **Category:** Export
- **Description:** Verify that the exported JSON has the correct top-level structure (`version`, `source_instance`, `reports`, `dashboards`).
- **Preconditions:** Mocked `_get()` returning 1 report.
- **Test Steps:**
  1. Mock `_get()` to return 1 valid report.
  2. Call `run(limit=10)`.
  3. Assert `report.export_json["version"] == "1.0"`.
  4. Assert `report.export_json["source_instance"]` is not empty.
  5. Assert `"reports"` key exists and is a list.
  6. Assert `"dashboards"` key exists and is a list.
- **Expected Result:** Valid JSON export structure.
- **Status:** ✅ Implemented as `test_export_structure`.

### T12 — Dashboard Fetch
- **ID:** T12
- **Priority:** P2
- **Category:** Data Acquisition
- **Description:** Verify that `list_dashboards()` correctly returns dashboard records from `pa_dashboards`.
- **Preconditions:** Mocked `_get()` returning dashboard data.
- **Test Steps:**
  1. Mock `_get()` to return `[{"sys_id": "d1", "name": "Exec", "active": "true"}]`.
  2. Call `list_dashboards()`.
  3. Assert returned list has length 1.
  4. Assert dashboard has `sys_id`, `name` fields.
- **Expected Result:** Dashboards fetched correctly.
- **Status:** ✅ Implemented as `test_dashboard_fetch`.

---

## 3. Priority Distribution

| Priority | Count | Test IDs | Category |
|----------|-------|----------|----------|
| **P0** | 5 | T01, T02, T03, T04, T05 | Core functionality (plugin check, enumeration, type mapping) |
| **P1** | 3 | T06, T07, T08 | Robustness (validation, config gen, full pipeline) |
| **P2** | 4 | T09, T10, T11, T12 | Quality (rendering, metrics, export structure, dashboards) |

---

## 4. Coverage Map

```
                      ┌─────────────────────────────┐
                      │   PlatformAnalyticsMigrator  │
                      └─────────────┬───────────────┘
                                    │
         ┌──────────────────────────┼──────────────────────────┐
         │                          │                          │
    ┌────▼────┐              ┌──────▼──────┐            ┌──────▼──────┐
    │  Input  │              │  Processing │            │   Output    │
    └────┬────┘              └──────┬──────┘            └──────┬──────┘
         │                          │                          │
    ┌────▼────┐              ┌──────▼──────┐            ┌──────▼──────┐
    │  T01    │ plugin       │  T04  bar   │            │  T07  config │
    │  T02    │ reports      │  T05  gauge │            │  T08  pipeline│
    │  T03    │ dashboards   │  T06  table │            │  T09  HTML   │
    │  T12    │ dashboards   │      validation│         │  T10  metrics│
    └─────────┘              │      T08  pipeline│       │  T11  JSON   │
                             └──────────────┘            └─────────────┘
```

**Coverage by Code Path:**
- `check_plugin()`: T01 ✅
- `list_reports()`: T02 ✅ (indirect)
- `list_dashboards()`: T03, T12 ✅
- `map_report()`: T04, T05, T06 ✅
- `generate_pa_config()`: T07 ✅
- `run()`: T08, T10, T11 ✅
- `save_report()`: T08 ✅
- `_render_html()`: T09 ✅

**Untested Code Paths:**
- `_get()` real HTTP failure paths (mocked only) — planned for integration tests
- `__main__` execution block — not unit-testable (CLI-driven)

---

## 5. Execution Instructions

### Run with unittest (native)
```bash
cd /home/crixus/sn_platform_analytics_migrator
python3 -m unittest tests.test_migrator -v
```

### Run with pytest
```bash
cd /home/crixus/sn_platform_analytics_migrator
python3 -m pytest tests/ -v --tb=short
```

### Expected Results
- **Minimum pass rate:** 10/10 (all unit tests must pass)
- **Expected pass rate:** 10/10 with no warnings
- **Runtime:** <2 seconds (all mocked, no network)

### Environment
- Python 3.10+
- No external test dependencies beyond stdlib `unittest`
- No ServiceNow instance required (all tests mock `_get()`)

---

## 6. Test Data

All test data is inline within test methods using dictionary literals. No external fixture files required. The mock pattern overrides `self.m._get` with a lambda returning predefined lists.

---

Copyright (C) 2026 Vladimir Kapustin
SPDX-License-Identifier: AGPL-3.0
