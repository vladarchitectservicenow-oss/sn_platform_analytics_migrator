# Regression Cases: ServiceNow Platform Analytics Migrator (sn_platform_analytics_migrator)

**Author:** Vladimir Kapustin
**License:** AGPL-3.0
**Product:** sn_platform_analytics_migrator
**Version:** 1.0.0
**Last Updated:** 2026-06-02

---

## Overview

This document defines 8 regression test cases (R01–R08) to be executed on every change to `src/migrator.py` or `tests/test_migrator.py`. These cases verify that existing functionality is not broken by new changes, refactoring, or dependency upgrades.

---

## Regression Test Cases

### R01 — Idempotent Execution
- **ID:** R01
- **Priority:** P0
- **Category:** Functional Integrity
- **Description:** Running `run()` twice with the same mocked data produces identical `MigrationReport` output (same total_reports, mappable, unmappable counts).
- **Test Steps:**
  1. Mock `_get()` to return fixed report set.
  2. Call `run()` twice.
  3. Assert `report1.total_reports == report2.total_reports`.
  4. Assert `report1.mappable == report2.mappable`.
  5. Assert `report1.unmappable == report2.unmappable`.
- **Expected Result:** Identical metrics on repeated execution.
- **Status:** ⬜ To implement in regression test run.

### R02 — Format Consistency Across Runs
- **ID:** R02
- **Priority:** P0
- **Category:** Output Integrity
- **Description:** Generated HTML and JSON files have consistent structure across multiple runs with identical input.
- **Test Steps:**
  1. Run `save_report()` twice with the same `MigrationReport`.
  2. Compare JSON file byte-for-byte after stripping timestamps.
  3. Compare HTML file structure (same number of `<tr>` rows, same `<th>` headers).
- **Expected Result:** Deterministic output with only timestamp variation.
- **Status:** ⬜ To implement in regression test run.

### R03 — Role Assignment Idempotency
- **ID:** R03
- **Priority:** P1
- **Category:** ServiceNow Platform
- **Description:** (Planned) Assigning `x_sn_platform_analytics_migrator.user` role to a user who already has it does not produce errors or duplicate assignments.
- **Test Steps:**
  1. Assign role to user.
  2. Attempt to assign the same role again.
  3. Verify no error is thrown and role count remains 1.
- **Expected Result:** Graceful no-op on duplicate role assignment.
- **Status:** ⬜ Planned (requires scoped app deployment on ServiceNow).

### R04 — Configuration Persistence
- **ID:** R04
- **Priority:** P1
- **Category:** ServiceNow Platform
- **Description:** (Planned) System properties (`x_sn_platform_analytics_migrator.*`) survive instance restart and retain their values.
- **Test Steps:**
  1. Set property `x_sn_platform_analytics_migrator.scan.chunk_size` to 500.
  2. Retrieve and verify value is 500.
  3. (Simulate restart or actual restart on PDI).
  4. Retrieve again and verify value is still 500.
- **Expected Result:** Properties persist across restarts.
- **Status:** ⬜ Planned (requires ServiceNow instance).

### R05 — Backward Compatibility: Python 3.10
- **ID:** R05
- **Priority:** P1
- **Category:** Runtime Compatibility
- **Description:** All 10 unit tests pass on Python 3.10, 3.11, and 3.12 without modification.
- **Test Steps:**
  1. Run `pytest tests/ -v` on Python 3.10.
  2. Run `pytest tests/ -v` on Python 3.11.
  3. Run `pytest tests/ -v` on Python 3.12.
  4. Assert 10/10 PASS on all versions.
- **Expected Result:** No Python-version-specific regressions.
- **Status:** ⬜ To implement (CI matrix build).

### R06 — Dependency Upgrade Safety
- **ID:** R06
- **Priority:** P2
- **Category:** Supply Chain
- **Description:** Upgrading `requests` to the latest minor/patch version does not break the test suite.
- **Test Steps:**
  1. Pin `requests==2.28.0`, run tests → 10/10 PASS.
  2. Upgrade to `requests==2.31.0`, run tests → 10/10 PASS.
  3. Upgrade to latest (`requests>=2.32.3`), run tests → 10/10 PASS.
- **Expected Result:** All tests pass on dependency upgrades within semver constraints.
- **Status:** ⬜ To implement (dependency upgrade test matrix).

### R07 — Empty Data Graceful Degradation
- **ID:** R07
- **Priority:** P2
- **Category:** Edge Handling
- **Description:** Running migration against an instance with zero reports produces a valid but empty report, not an error.
- **Test Steps:**
  1. Mock `_get()` to return `[]` for both reports and dashboards.
  2. Call `run()`.
  3. Assert `report.total_reports == 0`.
  4. Assert `report.mappable == 0`.
  5. Assert `report.unmappable == 0`.
  6. Assert no exception raised.
- **Expected Result:** Clean zero-count report; no crash.
- **Status:** ⬜ To implement in regression test run.

### R08 — Report File Cleanup
- **ID:** R08
- **Priority:** P3
- **Category:** Operational Hygiene
- **Description:** After test execution (especially `test_full_pipeline_mock`), no stale report files remain in the output directory.
- **Test Steps:**
  1. Run `test_full_pipeline_mock`.
  2. Verify JSON and HTML files were created.
  3. Verify cleanup code (`Path(path).unlink(missing_ok=True)`) removes them.
  4. Assert output directory has no stale files post-test.
- **Expected Result:** Test teardown successfully removes generated artifacts.
- **Status:** ✅ Verified in current test suite (unlink calls present).

---

## Regression Execution Protocol

### When to Run
- Before every commit to `src/migrator.py`
- Before every release tag
- After any dependency upgrade
- Weekly via CI cron (planned)

### Minimum Pass Threshold
- 8/8 regression cases must pass (or documented skip for ServiceNow-dependent cases)

### Execution Command
```bash
cd sn_platform_analytics_migrator
pytest tests/ -v -k "regression" --tb=short
# Or manually execute each R-test scenario
```

---

Copyright (C) 2026 Vladimir Kapustin
SPDX-License-Identifier: AGPL-3.0
