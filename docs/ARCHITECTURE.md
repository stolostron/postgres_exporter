# Architecture — postgres_exporter (Global Hub)

**postgres_exporter** is a Red Hat–maintained fork of the [Prometheus Community PostgreSQL Server Exporter](https://github.com/prometheus-community/postgres_exporter), packaged for Multicluster Global Hub. The Global Hub **operator** deploys this image on the global hub cluster alongside PostgreSQL to expose database metrics in Prometheus format.

Related: [multicluster-global-hub docs/ARCHITECTURE.md](https://github.com/stolostron/multicluster-global-hub/blob/release-5.1/docs/ARCHITECTURE.md) for the full operator/manager/agent data flow.

---

## Role in Global Hub

| Aspect | Detail |
| ------ | ------ |
| **Deployed by** | `MulticlusterGlobalHub` operator reconciler |
| **Runs on** | Global hub cluster, namespace `multicluster-global-hub` |
| **Data source** | PostgreSQL (`multicluster-global-hub-postgresql`) — configured via `DATA_SOURCE_NAME` env var |
| **Metrics endpoint** | `:9187/metrics` (Prometheus scrape target) |
| **User** | `nobody` (unprivileged, rootless container) |
| **Image** | `registry.redhat.io/multicluster-globalhub/multicluster-globalhub-postgres-exporter-rhel9` (prod) / Konflux dev workload quay path |

The exporter is **read-only** — it issues `SELECT` queries against PostgreSQL system catalogs and `pg_stat_*` views. It does not write to the database or participate in the spec/status sync data flow.

---

## ACM Customizations

This fork has **no source-code patches** on top of upstream. All customizations are in the build and packaging layer:

- **CGO enabled**: `promu build --cgo` produces a CGO binary required for Konflux FIPS binary scanning (`fbc-fips-check-oci-ta`).
- **promu vendored**: The `promu/` directory vendors the Prometheus build utility source so Konflux hermetic builds do not need external network access during compilation.
- **UBI-minimal runtime**: Production image uses `registry.access.redhat.com/ubi9/ubi-minimal` for Red Hat supply-chain compliance.
- **Red Hat labels**: Component `multicluster-globalhub-postgres-exporter-rhel9-container`, CPE `cpe:/a:redhat:multicluster_globalhub:5.1::el9`.

---

## Container Image

`Containerfile.konflux` is a two-stage build:

1. **Builder** — `brew.registry.redhat.io/rh-osbs/openshift-golang-builder:rhel_9_1.25`:
   - Builds `promu` from source with `CGO_ENABLED=1`.
   - Runs `promu build --cgo` to produce the `postgres_exporter` binary.
2. **Runtime** — `registry.access.redhat.com/ubi9/ubi-minimal`:
   - Copies `/workspace/postgres_exporter` to `/bin/postgres_exporter`.
   - Runs as `nobody`, exposes port `9187`.

Local build:

```bash
podman build -f Containerfile.konflux -t postgres-exporter:local .
```

---

## CI/CD (Konflux)

| Pipeline | Trigger | File |
| -------- | ------- | ---- |
| Push | `release-5.1` branch push | `.tekton/postgres-exporter-globalhub-5-1-push.yaml` |
| Pull request | PRs to `release-5.1` | `.tekton/postgres-exporter-globalhub-5-1-pull-request.yaml` |

- **Application:** `release-globalhub-5-1`
- **Component:** `postgres-exporter-globalhub-5-1`
- **Output:** `quay.io/redhat-user-workloads/acm-multicluster-glo-tenant/postgres-exporter-globalhub-5-1:<sha>`
- **Platforms:** linux/x86_64, ppc64le, s390x, arm64
- **Pipeline:** `konflux-build-catalog/pipelines/common-base.yaml` (hermetic, prefetch gomod for `promu/`)

---

## Configuration

The exporter is configured via:

| Method | Detail |
| ------ | ------ |
| `DATA_SOURCE_NAME` env var | PostgreSQL DSN — `postgresql://user:pass@host:5432/dbname?sslmode=disable` |
| `--config.file` flag | YAML config with `auth_modules` for multi-target scraping |
| `--collector.*` flags | Enable/disable individual metric collectors |

Key collectors active by default:

- `collector.database` — per-database stats
- `collector.locks` — lock counts by mode
- `collector.replication` — replication lag and slot state
- `collector.stat_bgwriter` — background writer activity
- `collector.stat_user_tables` — table-level I/O and vacuum stats

---

## Branch Strategy

| Branch | Purpose |
| ------ | ------- |
| `release-5.1` | Current Global Hub 5.1 development and Konflux builds |
| `release-2.x` | Prior GH release tracks (maintenance; `release-2.17` = GH 1.8, `release-2.16` = GH 1.7, etc.) |

The branch naming follows the ACM release numbering (`release-2.17` = ACM 2.17 = GH 1.8), not the upstream postgres_exporter version.

---

## Dependency Management

The repo uses standard Go modules (`go.mod` / `go.sum`). Dependency updates are managed by **Mintmaker** (Konflux bot) which opens PRs against each active release branch. Mintmaker uses `fix(deps)` for security-relevant bumps and `chore(deps)` for routine updates. All Mintmaker PRs carry `do-not-merge/hold` and require human review before merge.
