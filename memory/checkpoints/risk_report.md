# Risk Report: ServiceNow Platform Analytics Migrator (sn_platform_analytics_migrator)

**Product:** sn_platform_analytics_migrator
**Author:** Vladimir Kapustin
**License:** AGPL-3.0
**Last Updated:** 2026-06-02
**Risk Count:** 12 identified risks across P0–P3 severity levels

---

## Risk Severity Definitions

| Severity | Criteria | Response SLA |
|----------|----------|--------------|
| **P0 — Critical** | Blocks core functionality; no workaround; data loss or security breach | <4 hours |
| **P1 — High** | Severely degrades functionality; limited workaround; compliance risk | <24 hours |
| **P2 — Medium** | Degrades non-critical functionality; workaround exists | <1 week |
| **P3 — Low** | Cosmetic or future concern; no immediate impact | Next sprint |

---

## Risk Inventory

### R01 — P0: Missing Platform Analytics Plugin
- **ID:** R01
- **Severity:** P0 — Critical
- **Category:** Platform Dependency
- **Description:** If `com.snc.platform_analytics` or `com.snc.pa` plugins are not activated on the target instance, the entire migration pipeline fails. The application's `check_plugin()` method detects this, but cannot automatically activate plugins.
- **Impact:** Zero reports mappable; migration cannot proceed.
- **Likelihood:** Medium (depends on instance maturity; Washington DC+ instances typically have PA pre-installed)
- **Mitigation:**
  1. **Detection:** `check_plugin()` returns `plugin_active: false` before any migration attempt.
  2. **Documentation:** README includes explicit plugin activation prerequisites.
  3. **Admin remediation:** Instance admin must activate plugins via **System Applications > All > Platform Analytics**.
- **Status:** 🟡 Mitigation in place (detection + docs); automatic activation deferred to v1.1.

---

### R02 — P0: Hardcoded Default Credentials
- **ID:** R02
- **Severity:** P0 — Critical
- **Category:** Security
- **Description:** `src/migrator.py` contains a hardcoded password (`DEFAULT_PASS = os.environ.get("SN_PASSWORD", "7%%gXJzImsW7")`) for `dev362840.service-now.com`. If this repository is public or the instance is accessible, this constitutes a credential leak.
- **Impact:** Unauthorized access to developer instance; potential data exposure.
- **Likelihood:** High (repo is public on GitHub)
- **Mitigation:**
  1. **Immediate:** Remove hardcoded default and require env-var-only or interactive prompt.
  2. **Intermediate:** Rotate the dev instance password.
  3. **Long-term:** Use ServiceNow OAuth 2.0 or `sys_auth_profile` for credential storage.
- **Status:** 🔴 **CRITICAL — requires immediate remediation.** Remove hardcoded fallback password entirely.

---

### R03 — P1: No CI/CD Pipeline
- **ID:** R03
- **Severity:** P1 — High
- **Category:** DevOps
- **Description:** No GitHub Actions, Jenkins, or other CI/CD pipeline exists. Tests run only locally and manually. No automated linting, security scanning, or release automation.
- **Impact:** Regression bugs may be introduced; releases lack quality gates; no automated deployment.
- **Likelihood:** Certain (no pipeline exists)
- **Mitigation:**
  1. Add `.github/workflows/test.yml` with `pytest tests/ -v`.
  2. Add `.github/workflows/lint.yml` with `ruff` or `flake8`.
  3. Add `.github/workflows/security-scan.yml` with `bandit` or `CodeQL`.
  4. Add pre-release checklist to SOP.md.
- **Status:** 🔴 Planned for Q3 2026.

---

### R04 — P1: Incomplete Test Coverage
- **ID:** R04
- **Severity:** P1 — High
- **Category:** Quality
- **Description:** Current test suite has 10 unit tests, all mocking `_get()` to avoid real API calls. No integration tests against a live ServiceNow instance. No error-path tests for HTTP 401, 403, 500, or timeout scenarios in the actual `_get()` method (only mocked).
- **Impact:** Real API failures may produce unexpected behavior; error handling is untested against actual HTTP responses.
- **Likelihood:** Certain (integration tests absent)
- **Mitigation:**
  1. Add `tests/integration/` with tests against a ServiceNow developer instance.
  2. Add error-path unit tests covering `_get()` HTTP status codes: 401, 403, 404, 429, 500, 503.
  3. Add timeout simulation tests.
  4. Add `responses` or `httpretty` library for realistic HTTP mocking.
- **Status:** 🔴 Planned for Q3 2026.

---

### R05 — P1: Unsupported Visualization Data Loss
- **ID:** R05
- **Severity:** P1 — High
- **Category:** Data Integrity
- **Description:** 3 of 8 visualization types (`map`, `calendar`, `gauge`) are marked unsupported. Reports using these types are excluded from PA export JSON. Users may be unaware that their legacy reports are silently dropped.
- **Impact:** Data loss; unexpected report absence; manual intervention required.
- **Likelihood:** Medium (depends on report type distribution)
- **Mitigation:**
  1. **Explicit warning:** `MigrationReport` tracks `unmappable` count and surfaces issues per report.
  2. **HTML report:** Unsupported reports rendered with red rows and issue descriptions.
  3. **Future:** Add fallback mapping (e.g., `gauge` → `bar` or `single_score`) in v1.1.
  4. **Future:** Add `calendar` → `timeline` bridge.
- **Status:** 🟡 Partial mitigation (visibility); fallback mappings planned for v1.1.

---

### R06 — P2: Performance at Scale Untested
- **ID:** R06
- **Severity:** P2 — Medium
- **Category:** Performance
- **Description:** The application defaults to `limit=500` per API call but has not been load-tested with 10,000+ reports on a production-scale instance. No chunking/pagination beyond the default limit. No concurrency model exists.
- **Impact:** Potential timeout or memory exhaustion on large instances; O(N) processing time grows linearly.
- **Likelihood:** Medium (most instances have <500 reports; enterprise instances may have thousands)
- **Mitigation:**
  1. Implement pagination with `sysparm_offset`.
  2. Add `chunk_size` property to config.
  3. Profile with 10k, 50k, 100k synthetic reports.
  4. Add streaming JSON writer to avoid in-memory accumulation.
- **Status:** 🟡 Planned for v1.1.

---

### R07 — P2: Hardcoded Instance URL
- **ID:** R07
- **Severity:** P2 — Medium
- **Category:** Configuration
- **Description:** `DEFAULT_INSTANCE = "https://dev362840.service-now.com"` is hardcoded. Running against any other ServiceNow instance requires explicit constructor override or code modification.
- **Impact:** Friction for new users; production misconfiguration risk.
- **Likelihood:** High (most users will target their own instances)
- **Mitigation:**
  1. Add CLI argument parsing (`argparse`) with `--sn-url` parameter.
  2. Add environment variable `SN_URL` as default.
  3. Remove the hardcoded developer instance URL or make it an example in docs only.
- **Status:** 🟡 Planned for v1.1.

---

### R08 — P2: No Version Pinning for requests
- **ID:** R08
- **Severity:** P2 — Medium
- **Category:** Supply Chain
- **Description:** `requests` package is imported without version pinning in a `requirements.txt` file. No `pip freeze` or `pip-tools` lock file exists.
- **Impact:** Breaking changes in `requests` could silently break the application.
- **Likelihood:** Low (requests is mature and stable)
- **Mitigation:**
  1. Create `requirements.txt` with `requests>=2.28.0,<3.0`.
  2. Add `requirements-dev.txt` with `pytest>=7.0.0`.
  3. Use `pip-tools` for deterministic builds.
- **Status:** 🟡 Planned.

---

### R09 — P3: No requirements.txt
- **ID:** R09
- **Severity:** P3 — Low
- **Category:** Onboarding
- **Description:** No `requirements.txt` or `pyproject.toml` exists in the repository. New contributors must manually install dependencies.
- **Impact:** Slower onboarding; potential version mismatch.
- **Likelihood:** Certain (no file exists)
- **Mitigation:**
  1. Create `requirements.txt` with `requests>=2.28.0`.
  2. Optionally add `pyproject.toml` with build metadata.
- **Status:** 🟡 Planned.

---

### R10 — P3: No Type Checking
- **ID:** R10
- **Severity:** P3 — Low
- **Category:** Code Quality
- **Description:** Python type hints exist (`List[dict]`, `Optional[str]`) but no static type checker (mypy/pyright) is configured.
- **Impact:** Type errors may slip through to runtime.
- **Likelihood:** Low (codebase is small and well-typed)
- **Mitigation:**
  1. Add `mypy` to dev dependencies.
  2. Add `mypy src/ tests/` to CI pipeline.
- **Status:** 🟡 Planned.

---

### R11 — P3: No README Freshness Check
- **ID:** R11
- **Severity:** P3 — Low
- **Category:** Documentation
- **Description:** No automated check ensures README stays synchronized with code changes.
- **Impact:** README may become outdated; users follow stale instructions.
- **Likelihood:** Medium (over time)
- **Mitigation:**
  1. Add GitHub Action to check README.last_updated against latest commit date.
  2. Manual process: update README as part of release checklist.
- **Status:** 🟢 Accepted (low priority).

---

### R12 — P3: Single-Author Bus Factor
- **ID:** R12
- **Severity:** P3 — Low
- **Category:** Project Governance
- **Description:** All code and documentation authored by Vladimir Kapustin. Single point of failure for maintenance and knowledge transfer.
- **Impact:** If the author is unavailable, project may stall.
- **Likelihood:** Low (public repo with open-source community potential)
- **Mitigation:**
  1. Encourage community contributions via CONTRIBUTING.md.
  2. Add thorough documentation and inline comments.
  3. Consider co-maintainer or GitHub organization transfer if project grows.
- **Status:** 🟢 Accepted (low priority; mitigated by thorough docs).

---

## Mitigation Roadmap

| Quarter | Priority | Mitigations |
|---------|----------|-------------|
| **Q2 2026 (Now)** | P0 | R02: Remove hardcoded password; rotate credentials |
| **Q2 2026 (Now)** | P1 | R01: Add plugin prerequisite docs to README |
| **Q3 2026** | P1 | R03: Implement GitHub Actions CI/CD; R04: Add integration + error-path tests |
| **Q4 2026** | P2 | R05: Add fallback viz mappings; R06: Add pagination + load testing; R07: Add CLI args + env vars |
| **Q1 2027** | P3 | R08: Pin dependencies; R09: Create requirements.txt; R10: Add mypy; R11: Freshness check; R12: CONTRIBUTING.md |

---

## Risk Summary Graph

```
P0 ██░░░░░░░░ (2 risks — immediate remediation)
P1 ████░░░░░░ (3 risks — Q3 2026)
P2 ███░░░░░░░ (3 risks — Q4 2026)
P3 ████░░░░░░ (4 risks — Q1 2027)
```

---

Copyright (C) 2026 Vladimir Kapustin
SPDX-License-Identifier: AGPL-3.0
