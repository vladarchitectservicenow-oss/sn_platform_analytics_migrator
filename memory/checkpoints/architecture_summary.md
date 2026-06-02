# Architecture Summary: ServiceNow Platform Analytics Migrator (sn_platform_analytics_migrator)

**Product:** sn_platform_analytics_migrator
**Scope:** x_sn_platform_analytics_migrator
**Author:** Vladimir Kapustin
**License:** AGPL-3.0
**Version:** 1.0.0
**Last Updated:** 2026-06-02

---

## 1. Overview

The ServiceNow Platform Analytics Migrator (`sn_platform_analytics_migrator`) is a Python 3.10+ application that scans legacy Core UI reports (`sys_report`) and Performance Analytics dashboards (`pa_dashboards`) on ServiceNow instances, maps them to Platform Analytics equivalents, and generates PA-compatible JSON configuration payloads for import via Migration Center. The application uses ServiceNow's Table REST API to enumerate, map, validate, and export report configurations.

---

## 2. Component Table

| Component | File | Language | Responsibility | Status |
|-----------|------|----------|----------------|--------|
| **PlatformAnalyticsMigrator** | `src/migrator.py` | Python 3 | Core migration engine: plugin check, report/dashboard enumeration, type mapping, PA config generation, HTML/JSON report export | Production |
| **ReportMapping** | `src/migrator.py` | Python 3 | Data class representing a single report mapping (sys_id, name, source_type, target_type, issues) | Production |
| **MigrationReport** | `src/migrator.py` | Python 3 | Data class for full migration report container (instance, timestamp, mapping summary, export_json) | Production |
| **TestMigrator** | `tests/test_migrator.py` | Python 3 | Unit test suite: 10 test cases covering map, config generation, pipeline, rendering, plugin check, dashboard fetch, export structure | Production |
| **REST API Client** | `src/migrator.py::_get()` | Python 3 | HTTP Basic Auth REST client targeting ServiceNow Table API (`/api/now/table/*`) | Production |
| **HTML Renderer** | `src/migrator.py::_render_html()` | Python 3 | Static HTML report generator with green/red status table | Production |

---

## 3. Data Flow Diagram

```
                    ┌──────────────────────────┐
                    │   ServiceNow Instance     │
                    │   (dev362840)             │
                    └───────────┬──────────────┘
                                │ REST API (Basic Auth)
                    ┌───────────▼──────────────┐
                    │  PlatformAnalyticsMigrator │
                    │                           │
                    │  1. check_plugin()        │
                    │     └─ GET /v_plugin      │
                    │                           │
                    │  2. list_reports()        │
                    │     └─ GET /sys_report    │
                    │                           │
                    │  3. list_dashboards()     │
                    │     └─ GET /pa_dashboards │
                    │                           │
                    │  4. map_report()          │
                    │     └─ REPORT_TYPE_MAP    │
                    │         lookup + validate │
                    │                           │
                    │  5. generate_pa_config()  │
                    │     └─ PA JSON schema     │
                    │                           │
                    │  6. run()                 │
                    │     └─ MigrationReport    │
                    │                           │
                    │  7. save_report()         │
                    └──────┬───────────┬────────┘
                           │           │
                    ┌──────▼──┐  ┌────▼──────────┐
                    │ JSON    │  │ HTML Report    │
                    │ Export  │  │ (pa_migrator_  │
                    │         │  │  YYYY-MM-DD_   │
                    │         │  │  host.html)    │
                    └─────────┘  └────────────────┘
```

---

## 4. API Contract

### Inbound (ServiceNow Table REST API)

| Endpoint | Method | Params | Returns |
|----------|--------|--------|---------|
| `/api/now/table/v_plugin` | GET | `sysparm_query`, `sysparm_fields`, `sysparm_limit` | `[{name, active}]` |
| `/api/now/table/sys_report` | GET | `sysparm_limit`, `sysparm_fields`, `sysparm_query` | `[{sys_id, title, type, table, filter, group_by, chart_size, aggregation, sum_field, order_by}]` |
| `/api/now/table/pa_dashboards` | GET | `sysparm_limit`, `sysparm_fields` | `[{sys_id, name, owner, active}]` |

### Outbound (PA Config Export)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | Yes | Schema version (currently "1.0") |
| `source_instance` | string | Yes | ServiceNow instance URL |
| `reports` | array | Yes | List of PA-compatible report configs |
| `report[].sys_id` | string | Yes | Original report sys_id |
| `report[].name` | string | Yes | Report title/name |
| `report[].type` | string | Yes | PA visualization type (list, bar, pie, line, pivot) |
| `report[].table` | string | Yes | Target database table |
| `report[].filter` | string | No | Encoded query filter |
| `report[].group_by` | string | No | Group-by field |
| `report[].source` | string | Yes | Always "legacy_migration" |
| `report[].supported` | boolean | Yes | Whether type is mappable |
| `dashboards` | array | Yes | List of dashboard summaries |

### Authentication

- **Scheme:** HTTP Basic Auth
- **Credentials:** Source from environment (`SN_PASSWORD`) or passed at constructor
- **Transport:** HTTPS required; `timeout=30s`

---

## 5. Report Type Mapping Matrix

| Source Type | Target Type | Supported | Notes |
|-------------|-------------|-----------|-------|
| `list` | `list` | ✅ | Full support |
| `bar` | `bar` | ✅ | Full support |
| `pie` | `pie` | ✅ | Full support |
| `line` | `line` | ✅ | Full support |
| `pivot` | `pivot` | ✅ | Full support |
| `map` | `map` | ❌ | Geo-map not supported in PA |
| `calendar` | `calendar` | ❌ | Calendar viz not in PA |
| `gauge` | `gauge` | ❌ | Gauge not supported in PA |

---

## 6. Performance Statistics

| Metric | Value | Notes |
|--------|-------|-------|
| Report enumeration limit | 500 per call | Configurable via `limit` parameter |
| REST timeout | 30 seconds | Single-request timeout |
| Test suite execution time | <2 seconds | 10 unit tests (mocked `_get`) |
| JSON export size | ~2-5 KB per 100 reports | Typical payload |
| HTML render size | ~3-8 KB per 100 reports | Self-contained HTML |
| Memory footprint | <50 MB | Dataclass-based, no heavy frameworks |

---

## 7. Error Handling Strategy

| Scenario | Behavior |
|----------|----------|
| HTTP non-200 | `RuntimeError` with URL, status code, and first 200 chars of response |
| Missing plugin | `plugin_active: false` in check result |
| Unsupported type | Mapped with `supported: false` + issue description |
| Missing table field | `issues` list populated with "Missing target table" |
| Empty report set | `total_reports: 0, mappable: 0, unmappable: 0` |
| Network timeout | `requests.exceptions.Timeout` propagates to caller |

---

## 8. Key Design Decisions

1. **Dataclass-based data model:** `ReportMapping` and `MigrationReport` use `@dataclass` and `asdict()` for clean serialization — no external ORM.
2. **Dictionary-based mock testing:** Tests override `self.m._get` with lambda returning lists of dicts — no mocking framework dependency.
3. **Dual export (JSON + HTML):** Every run generates both machine-readable JSON and human-readable HTML simultaneously.
4. **UTC timestamps:** All report timestamps use `datetime.now(timezone.utc)` for consistency.
5. **Hardcoded default instance:** `dev362840.service-now.com` as default for development; production overrides via constructor.

---

Copyright (C) 2026 Vladimir Kapustin
SPDX-License-Identifier: AGPL-3.0
