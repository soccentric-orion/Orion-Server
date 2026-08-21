# Orion-Server

## 1. Project Summary

Orion-Server is the on-premises C++ orchestration and evidence backend for Orion IV&V.

- The executable starts database migrations, a gRPC surface for embedded clients, and a Drogon HTTP application for the Dashboard and external tools.
- `HalServiceImpl` exposes direct HAL-oriented operations.
- `ClientServiceImpl` receives the on-device client's result stream and delivers commands.
- REST is mounted under `/api/v1` and uses session cookies or bearer tokens through an authentication filter.

Settings and structured logging configure the process.

- `DbClient` and `MigrationRunner` manage asynchronous PostgreSQL access and ten SQL migrations.
- Repositories isolate persistence for users, tokens, targets, runs, results, logs, schedules, templates, notifications, OTA, and audit data.
- Services implement authentication, run lifecycle, modules, configuration, comparisons, thresholds, anomaly handling, retention, and OTA orchestration.
- Controllers cover health, auth, target, modules, runs, logs, configuration, settings, notifications, and OTA.
- A bounded stream queue provides backpressure, and SSE exposes live logs or run events.

CMake presets, Docker Compose, a dev container, configuration examples, dependency notes, and setup scripts support development.

- GoogleTest sources cover utilities, registry behavior, stream backpressure, migration behavior, anomaly and threshold logic, result conversion, run badges, UUIDs, time, and hashing.
- The audit records 65 CTest entries with one migration test skipped without `ORION_TEST_DSN`; that is evidence, not a universal run claim.

## 2. Feature Status

| # | Name / Description | Implemented? | Test Coverage? | Manual Test |
|---:|---|---|---|---|
| 1 | CMake-built executable and settings | Implemented | Partial: config tests exist | [ ] Manual verification required |
| 2 | Drogon REST API under `/api/v1` | Implemented | Partial: controller/unit coverage | [ ] Manual verification required |
| 3 | Session and bearer-token auth filter | Implemented | Partial: utility/service tests | [ ] Manual verification required |
| 4 | HAL and Client gRPC services | Implemented | Partial: backpressure/conversion tests | [ ] Manual verification required |
| 5 | PostgreSQL access and ten migrations | Implemented | Partial: migration test requires DSN | [ ] Manual verification required |
| 6 | Run lifecycle, scheduling, retention, comparisons | Implemented | Partial: run/service tests | [ ] Manual verification required |
| 7 | Module registry and test config persistence | Implemented | Partial: `ModuleRegistryTest` | [ ] Manual verification required |
| 8 | Result/log ingestion and SSE streams | Implemented | Partial: end-to-end unverified | [ ] Manual verification required |
| 9 | OTA, notifications, audit, thresholds | Implemented | Partial: anomaly/threshold tests | [ ] Manual verification required |
| 10 | Docker, dev container, scripts, presets | Implemented | Unknown: environment-dependent | [ ] Manual verification required |

## 3. Build and Test

```bash
cmake --preset debug
cmake --build build/debug
cmake --preset test
cmake --build build/test
ctest --test-dir build/test --output-on-failure
```
