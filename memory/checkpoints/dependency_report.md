# Dependency Report: ServiceNow Platform Analytics Migrator (sn_platform_analytics_migrator)

**Product:** sn_platform_analytics_migrator
**Author:** Vladimir Kapustin
**License:** AGPL-3.0
**Last Updated:** 2026-06-02

---

## 1. ServiceNow Platform Dependencies (Plugins)

| Plugin ID | Plugin Name | Required | Purpose |
|-----------|-------------|----------|---------|
| `com.snc.pa` | Performance Analytics | Yes (runtime) | Core PA data model and APIs |
| `com.snc.platform_analytics` | Platform Analytics | Yes (runtime) | Next Experience PA framework |
| `com.glide.rest.api` | REST API Services | Yes (required) | Table API access for migration |
| `com.glide.app-repo` | Application Repository | Optional | Source control integration for updates |

---

## 2. ServiceNow Tables

| Table | Scope | Read/Write | Purpose |
|-------|-------|------------|---------|
| `sys_report` | Global | Read | Source: legacy Core UI reports |
| `pa_dashboards` | Global | Read | Source: Performance Analytics dashboards |
| `v_plugin` | Global | Read | Plugin activation status check |
| `sys_log` | Global | Write (log) | Audit trail for all migration operations |
| `sys_properties` | Global | Read | System property lookup |
| `sys_auth_profile` | Global | Read | Encrypted credential store (future) |
| `x_sn_platform_analytics_migrator_config` | Scoped | Read/Write | Application configuration (planned) |
| `x_sn_platform_analytics_migrator_log` | Scoped | Write | Scoped audit log (planned) |

---

## 3. ServiceNow Roles

| Role | Privilege Level | Purpose |
|------|-----------------|---------|
| `x_sn_platform_analytics_migrator.admin` | Full access | Configuration, deployment, report management |
| `x_sn_platform_analytics_migrator.user` | Read-only | Report viewing, health-check access |
| `x_sn_platform_analytics_migrator.api` | API service account | CI/CD pipeline integration |
| `snc_internal` | Inherited | Internal ServiceNow access (inheritable) |
| `admin` | Inherited | Global admin (inheritable) |

---

## 4. ServiceNow System Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x_sn_platform_analytics_migrator.logging.level` | Choice | INFO | Log level: DEBUG, INFO, WARN, ERROR |
| `x_sn_platform_analytics_migrator.scan.chunk_size` | Integer | 500 | Records per batch for large tables |
| `x_sn_platform_analytics_migrator.scan.delta.enabled` | Boolean | true | Enable incremental/delta scanning |
| `x_sn_platform_analytics_migrator.output.format.default` | Choice | md | Default export format: md, json, csv |
| `x_sn_platform_analytics_migrator.api.timeout_seconds` | Integer | 30 | REST call timeout |
| `x_sn_platform_analytics_migrator.encryption.key_id` | String | — | Reference to sys_encryption_context (future) |
| `x_sn_platform_analytics_migrator.healthcheck.enabled` | Boolean | true | Enable health endpoint |

---

## 5. Python Dependencies (Runtime)

| Package | Version | Required | Purpose |
|---------|---------|----------|---------|
| `requests` | >=2.28.0 | Yes | HTTP REST client for ServiceNow Table API |
| `dataclasses` | Built-in | Yes | Data model classes (stdlib, Python 3.7+) |
| `json` | Built-in | Yes | JSON serialization/deserialization |
| `pathlib` | Built-in | Yes | Filesystem path handling |
| `datetime` | Built-in | Yes | UTC timestamp generation |
| `typing` | Built-in | Yes | Type hints (List, Optional, Dict) |

---

## 6. Python Dependencies (Test)

| Package | Version | Required | Purpose |
|---------|---------|----------|---------|
| `pytest` | >=7.0.0 | Yes | Test runner (alternative to unittest) |
| `unittest` | Built-in | Yes | Standard test framework (primary) |
| `os` | Built-in | Yes | Environment variable access |
| `sys` | Built-in | Yes | sys.path manipulation for imports |

---

## 7. Version Matrix

| Component | Min Version | Recommended | Max Tested |
|-----------|------------|-------------|------------|
| Python | 3.10 | 3.12 | 3.12 |
| ServiceNow Instance | Utah (Q1 2023) | Washington DC (Q1 2024) | Xanadu (Q3 2024) |
| requests | 2.28.0 | 2.32.0 | 2.32.3 |
| pytest | 7.0.0 | 8.3.0 | 9.0.3 |
| Platform Analytics Plugin | Washington DC baseline | Washington DC Patch 2 | Xanadu |

---

## 8. External Integration Dependencies

| Integration | Protocol | Required | Purpose |
|-------------|----------|----------|---------|
| ServiceNow Table REST API | HTTPS (TLS 1.2+) | Yes | Primary data source |
| GitHub Actions | YAML workflow | Optional | CI/CD pipeline |
| Grafana | HTTP dashboard | Optional | Monitoring dashboard template |
| Power BI / Tableau | JSON import | Optional | Export consumption |
| Splunk / Datadog / Elastic | Log ingestion | Optional | Structured log collection |

---

## 9. Python Environment Setup

```bash
python3 -m venv venv
source venv/bin/activate
pip install requests pytest
# Or if requirements.txt exists:
pip install -r requirements.txt
```

---

## 10. Dependency Graph

```
sn_platform_analytics_migrator
├── [Python Runtime]
│   ├── requests (HTTP client)
│   ├── dataclasses (stdlib)
│   ├── json (stdlib)
│   ├── pathlib (stdlib)
│   ├── datetime (stdlib)
│   └── typing (stdlib)
├── [ServiceNow Platform]
│   ├── com.snc.pa (Performance Analytics plugin)
│   ├── com.snc.platform_analytics (Platform Analytics plugin)
│   ├── com.glide.rest.api (REST API plugin)
│   ├── sys_report (Table: legacy reports)
│   ├── pa_dashboards (Table: PA dashboards)
│   └── v_plugin (Table: plugin registry)
└── [Test Infrastructure]
    ├── pytest >=7.0.0
    ├── unittest (stdlib)
    └── Docker (optional, for isolated test environments)
```

---

Copyright (C) 2026 Vladimir Kapustin
SPDX-License-Identifier: AGPL-3.0
