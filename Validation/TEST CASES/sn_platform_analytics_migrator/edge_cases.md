# Edge Cases: ServiceNow Platform Analytics Migrator (sn_platform_analytics_migrator)

**Author:** Vladimir Kapustin
**License:** AGPL-3.0
**Product:** sn_platform_analytics_migrator
**Version:** 1.0.0
**Last Updated:** 2026-06-02

---

## Overview

This document defines 8 edge case scenarios (E01–E08) that test the boundaries and limits of the Platform Analytics Migrator. Edge cases target inputs at the extremes of valid ranges, unexpected data shapes, and resource exhaustion scenarios.

---

## Edge Case Scenarios

### E01 — Empty Table: Zero Records
- **ID:** E01
- **Priority:** P0
- **Category:** Data Boundary
- **Description:** The `sys_report` table returns zero records (instance has no reports). The migrator should produce a valid, empty `MigrationReport` without errors.
- **Test Steps:**
  1. Mock `_get()` to return `[]` for all endpoints.
  2. Call `run()`.
  3. Assert `report.total_reports == 0`.
  4. Assert `report.mappable == 0`.
  5. Assert `report.unmappable == 0`.
  6. Assert `report.export_json["reports"] == []`.
  7. Assert `report.export_json["dashboards"] == []`.
  8. Call `save_report()` and verify files are created with empty tables.
- **Expected Result:** Clean, empty report with no errors or crashes.
- **Status:** ⬜ To implement.

### E02 — Maximum Batch Size Exceeded (50k+ Records)
- **ID:** E02
- **Priority:** P0
- **Category:** Resource Boundary
- **Description:** An instance has 50,000+ reports, exceeding the default `limit=500`. Only the first 500 are fetched; remaining reports are silently missed.
- **Test Steps:**
  1. (Simulated) Create mock returning 500 report dicts.
  2. Call `list_reports(limit=500)`.
  3. Assert exactly 500 reports returned.
  4. Verify no pagination logic to fetch beyond limit 500.
- **Expected Result:** Only first 500 reports processed; pagination gap identified.
- **Status:** 🟡 Known limitation — pagination enhancement planned for v1.1.

### E03 — Null/Missing Configuration Properties
- **ID:** E03
- **Priority:** P1
- **Category:** Data Integrity
- **Description:** A report dict has `null` or missing values for optional fields (`filter`, `group_by`, `chart_size`, `aggregation`, `sum_field`, `order_by`).
- **Test Steps:**
  1. Mock `_get()` to return `[{"sys_id": "1", "title": "NullFields", "type": "bar", "table": "incident"}]` (no filter/group_by keys).
  2. Call `map_report(record)`.
  3. Assert `result.filter == ""` (default).
  4. Assert `result.group_by is None`.
  5. Assert no KeyError raised.
- **Expected Result:** Graceful handling of missing keys; defaults applied.
- **Status:** ✅ Covered by existing tests (dict.get with defaults).

### E04 — Missing Dependency: Plugin Not Activated
- **ID:** E04
- **Priority:** P1
- **Category:** Platform Integration
- **Description:** The `com.snc.platform_analytics` plugin is not activated on the target instance. The `check_plugin()` method returns `plugin_active: false`.
- **Test Steps:**
  1. Mock `_get()` to return `[]` for plugin query.
  2. Call `check_plugin()`.
  3. Assert `result["plugin_active"] == False`.
  4. Assert `result["plugins"] == []`.
  5. Verify no exception raised.
- **Expected Result:** Clean detection of missing plugin; no crash.
- **Status:** ✅ Covered by existing test structure (mocked empty).

### E05 — REST API Timeout
- **ID:** E05
- **Priority:** P1
- **Category:** Network Resilience
- **Description:** The ServiceNow REST API takes >30 seconds to respond, triggering `requests.exceptions.ReadTimeout`.
- **Test Steps:**
  1. (Integration test) Point at a slow/mock HTTP server that delays 35 seconds.
  2. Call `list_reports()`.
  3. Assert `requests.exceptions.Timeout` is raised.
  4. Verify no partial data corruption.
- **Expected Result:** Clean timeout exception propagation; no hang.
- **Status:** ⬜ Planned for integration test suite.

### E06 — Unicode/Special Characters in Field Names
- **ID:** E06
- **Priority:** P2
- **Category:** Data Encoding
- **Description:** Report titles or field values contain Unicode characters (e.g., `日本語`, `émojis 🎉`, right-to-left text).
- **Test Steps:**
  1. Mock `_get()` to return report with title `"日本語レポート"`.
  2. Call `map_report()`, then `generate_pa_config()`, then `save_report()`.
  3. Assert JSON output correctly encodes Unicode (`ensure_ascii=False`).
  4. Assert HTML output renders Unicode properly.
- **Expected Result:** Unicode preserved in all outputs; no encoding errors.
- **Status:** ✅ Covered by `ensure_ascii=False` in JSON and UTF-8 HTML charset.

### E07 — Concurrent/Race Condition Prevention
- **ID:** E07
- **Priority:** P2
- **Category:** Concurrency Safety
- **Description:** Two instances of `PlatformAnalyticsMigrator` running simultaneously against the same instance could produce conflicting report files (same filename pattern).
- **Test Steps:**
  1. Run two migrator instances in parallel threads.
  2. Observe filename collision: both write `pa_migrator_YYYY-MM-DD_host.json`.
  3. Second write overwrites first (no lock).
- **Expected Result:** 🟡 Overwrite detected — no data corruption, but last-write-wins behavior.
- **Status:** 🟡 Known limitation — add PID/microsecond to filename in v1.1.

### E08 — Malformed JSON Response from API
- **ID:** E08
- **Priority:** P2
- **Category:** API Robustness
- **Description:** The ServiceNow instance returns a malformed JSON response (e.g., truncated response, non-JSON content-type).
- **Test Steps:**
  1. Mock `_get()` to return `{"result": "not-a-list"}` (dict instead of list).
  2. Call `list_reports()`.
  3. Assert either a parse error or the dict is treated as a single-item list.
- **Expected Result:** 🟡 Current code does `r.json().get("result", [])` — if result is not a list, downstream iteration may break.
- **Status:** 🟡 Known limitation — add type validation in v1.1.

---

## Edge Case Priority Distribution

| Priority | Count | Test IDs |
|----------|-------|----------|
| **P0** | 2 | E01, E02 |
| **P1** | 3 | E03, E04, E05 |
| **P2** | 3 | E06, E07, E08 |

---

## Edge Case Coverage Status

| Status | Count | IDs |
|--------|-------|-----|
| ✅ Covered | 4 | E03, E04, E06, (R07 from regression) |
| ⬜ To Implement | 2 | E01, E05 |
| 🟡 Known Limitation | 3 | E02, E07, E08 |

---

## Execution

```bash
cd sn_platform_analytics_migrator
pytest tests/ -v -k "edge" --tb=short
```

---

Copyright (C) 2026 Vladimir Kapustin
SPDX-License-Identifier: AGPL-3.0
