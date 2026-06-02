# Execution Plan: ServiceNow Platform Analytics Migrator (sn_platform_analytics_migrator)

**Product:** sn_platform_analytics_migrator
**Author:** Vladimir Kapustin
**License:** AGPL-3.0
**Last Updated:** 2026-06-02

---

## Phase Breakdown

### Phase 1: Discovery & Analysis
**Status:** ✅ Complete
**Duration:** 1 sprint

| Task | Owner | Status | Deliverable |
|------|-------|--------|-------------|
| Clone repository and inspect structure | Pipeline | ✅ Done | Working directory |
| Identify framework: Python 3.10+, ServiceNow REST API, unittest | Pipeline | ✅ Done | `architecture_summary.md` |
| Analyze source code: `migrator.py` (172 LOC), `test_migrator.py` (86 LOC) | Pipeline | ✅ Done | Code analysis notes |
| Detect dependencies: `requests`, stdlib only | Pipeline | ✅ Done | `dependency_report.md` |
| Map data model: `ReportMapping`, `MigrationReport` dataclasses | Pipeline | ✅ Done | Component table |
| Identify risks: 12 risks (2 P0, 3 P1, 3 P2, 4 P3) | Pipeline | ✅ Done | `risk_report.md` |

---

### Phase 2: Validation & Test Suite
**Status:** ✅ Complete
**Duration:** 1 sprint

| Task | Owner | Status | Deliverable |
|------|-------|--------|-------------|
| Create Test Suite SOP (12 scenarios T01-T12) | Pipeline | ✅ Done | `test_suite_SOP.md` |
| Create Regression Cases (8 cases R01-R08) | Pipeline | ✅ Done | `regression_cases.md` |
| Create Edge Cases (8 cases E01-E08) | Pipeline | ✅ Done | `edge_cases.md` |
| Create Validation Checklist (D/T/R/L/S/G/F codes) | Pipeline | ✅ Done | `validation_checklist.md` |
| Run existing unit tests (`pytest tests/ -v`) | Pipeline | ✅ Done | Test results |
| Add copyright headers to source files | Pipeline | ✅ Done | SPDX-compliant headers |
| Verify all Phase 2 deliverables | Pipeline | ✅ Done | Self-review |

---

### Phase 3: Integration & Playwright Testing
**Status:** 🔜 Planned (Q3 2026)
**Duration:** 2 sprints

| Task | Owner | Status | Deliverable |
|------|-------|--------|-------------|
| Set up ServiceNow PDI (Personal Developer Instance) | Operator | ⏳ Planned | PDI URL + credentials |
| Write Playwright script for PDI login validation | Operator | ⏳ Planned | `tests/e2e/test_pdi_login.py` |
| Test report enumeration against live `sys_report` table | Operator | ⏳ Planned | Integration test results |
| Test dashboard enumeration against live `pa_dashboards` | Operator | ⏳ Planned | Dashboard test results |
| Test plugin check against live `v_plugin` table | Operator | ⏳ Planned | Plugin status report |
| Validate JSON export schema against PA Migration Center format | Operator | ⏳ Planned | Schema conformance report |
| Add integration tests to `tests/integration/` | Developer | ⏳ Planned | Integration test suite |
| Set up GitHub Actions CI pipeline | Developer | ⏳ Planned | `.github/workflows/` |

---

### Phase 4: Code Refactoring
**Status:** 🔜 Planned (Q4 2026)
**Duration:** 1 sprint

| Task | Owner | Status | Deliverable |
|------|-------|--------|-------------|
| Remove hardcoded default password (R02 fix) | Developer | ⏳ Planned | `migrator.py` patch |
| Remove hardcoded default instance URL (R07 fix) | Developer | ⏳ Planned | CLI args + env var |
| Add `argparse` CLI interface (`--sn-url`, `--sn-user`, `--sn-pass`, `--limit`, `--format`) | Developer | ⏳ Planned | `src/cli.py` |
| Extract HTML renderer to separate module | Developer | ⏳ Planned | `src/render.py` |
| Add pagination support for large report sets | Developer | ⏳ Planned | `_get_paginated()` method |
| Add type-checking with `mypy` | Developer | ⏳ Planned | `mypy.ini` |
| Add `requirements.txt` and `requirements-dev.txt` | Developer | ⏳ Planned | Dependency lock files |
| Run full test suite post-refactor | Pipeline | ⏳ Planned | 100% pass rate |

---

### Phase 5: README Expansion
**Status:** ⏳ In Progress
**Duration:** 1 sprint

| Task | Owner | Status | Deliverable |
|------|-------|--------|-------------|
| Verify README ≥2000 words | Pipeline | ✅ Done (19,053 chars) | Word count check |
| Ensure README has: overview, architecture mermaid, installation, config table, ROI, troubleshooting, API ref, testing, roadmap, license, support | Pipeline | ✅ Done | Comprehensive README |
| Add multi-year ROI projection table | Pipeline | ✅ Done | 3-year ROI table |
| Add glossary of ServiceNow terms | Pipeline | ✅ Done | Glossary |
| Add copyright + SPDX header to README | Pipeline | ✅ Done | License section |

---

### Phase 6: Quality Gate
**Status:** 🔜 Planned (Q4 2026)
**Duration:** 1 sprint

| Task | Owner | Status | Deliverable |
|------|-------|--------|-------------|
| Run full test suite (unit + integration + e2e) | Pipeline | ⏳ Planned | 100% pass rate |
| Run `ruff` or `flake8` linter on all Python files | Pipeline | ⏳ Planned | Zero lint errors |
| Run `mypy` type checker | Pipeline | ⏳ Planned | Zero type errors |
| Run `bandit` security audit | Pipeline | ⏳ Planned | Zero critical/high findings |
| Validate all markdown docs have headers, consistent formatting | Pipeline | ⏳ Planned | Doc lint |
| Verify DONE.marker is present at repo root | Pipeline | ⏳ Planned | Existence check |
| Verify `.gitignore` covers all generated artifacts | Pipeline | ⏳ Planned | Coverage check |
| Cross-reference test IDs (T01-T12, R01-R08, E01-E08) across all docs | Pipeline | ⏳ Planned | Consistency report |

---

### Phase 7: Git Operations
**Status:** 🔜 Planned (per pipeline run)
**Duration:** <5 minutes

| Task | Command | Status |
|------|---------|--------|
| Stage all changes | `git add -A` | ⏳ Pending |
| Commit with standardized message | `git commit -m "fix: expand Phase 1+2 docs, add copyright headers, .gitignore, DONE.marker"` | ⏳ Pending |
| Push with force to main | `git push --force origin main` | ⏳ Pending |

---

### Phase 8: Completion
**Status:** 🔜 Planned (per pipeline run)
**Duration:** <1 minute

| Task | Owner | Status | Deliverable |
|------|-------|--------|-------------|
| Verify DONE.marker at repo root | Pipeline | ⏳ Pending | Existence check |
| Verify all Phase 1 docs are non-skeletal (>500 words each) | Pipeline | ⏳ Pending | Word count check |
| Verify all Phase 2 docs have correct ID format (T01-T12, R01-R08, E01-E08) | Pipeline | ⏳ Pending | Format check |
| Generate pipeline summary | Pipeline | ⏳ Pending | Summary report |

---

## Milestones

| Milestone | Target Quarter | Key Deliverables | Dependencies |
|-----------|---------------|-------------------|--------------|
| **M1: Analysis Complete** | Q2 2026 ✅ | architecture_summary.md, dependency_report.md, risk_report.md, execution_plan.md | Repository cloned |
| **M2: Test Suite Complete** | Q2 2026 ✅ | test_suite_SOP.md (12 scenarios), regression_cases.md (8 cases), edge_cases.md (8 cases), validation_checklist.md | M1 |
| **M3: CI/CD Active** | Q3 2026 | GitHub Actions pipeline, integration tests, Playwright E2E tests | M2 |
| **M4: Refactor Complete** | Q4 2026 | CLI interface, pagination, type checking, requirements.txt | M3 |
| **M5: Beta Release (v1.0)** | Q4 2026 | All quality gates passed, full test coverage, security audit clean | M4 |
| **M6: GA Release (v1.1)** | Q1 2027 | Fallback viz mappings, auto-remediation, Grafana dashboard | M5 |

---

## Dependency Graph

```
Phase 1 (Analysis)
    │
    ▼
Phase 2 (Validation & Docs) ─────────────────┐
    │                                         │
    ▼                                         ▼
Phase 3 (Integration Tests)              Phase 5 (README)
    │                                         │
    ▼                                         │
Phase 4 (Refactoring) ◄───────────────────────┘
    │
    ▼
Phase 6 (Quality Gate)
    │
    ▼
Phase 7 (Git Push)
    │
    ▼
Phase 8 (Completion)
```

**Critical Path:** Phase 1 → Phase 2 → Phase 3 → Phase 4 → Phase 6 → Phase 7 → Phase 8

**Parallel Track:** Phase 5 (README) can run concurrently with Phase 3/4.

---

## Risk-Adjusted Timeline

| Scenario | Total Duration | Assumption |
|----------|---------------|------------|
| **Best case** | 8 sprints (4 months) | All dependencies available; no PDI delays; no blocking issues |
| **Expected** | 10 sprints (5 months) | One PDI reprovision; minor integration issues |
| **Worst case** | 14 sprints (7 months) | PDI limitations block Phase 3; refactoring uncovers architectural debt |

---

## Resource Requirements

| Phase | Resource | Hours |
|-------|----------|-------|
| Phase 1 | Pipeline automation | <1 hour |
| Phase 2 | Pipeline automation | <1 hour |
| Phase 3 | ServiceNow PDI + Operator | 16 hours (manual testing) |
| Phase 4 | Python developer | 20 hours |
| Phase 5 | Pipeline + developer review | 2 hours |
| Phase 6 | Pipeline automation | <1 hour |
| Phase 7-8 | Pipeline automation | <10 minutes |

---

Copyright (C) 2026 Vladimir Kapustin
SPDX-License-Identifier: AGPL-3.0
