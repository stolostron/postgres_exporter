# CLAUDE.md — postgres_exporter

AI assistant context for **postgres_exporter** — the Red Hat fork of the Prometheus Community PostgreSQL Server Exporter, packaged for Multicluster Global Hub to expose database metrics for monitoring.

Onboarded per [Fleet Engineering Agentic SDLC — repo onboarding](https://github.com/OpenShift-Fleet/agentic-sdlc/blob/main/practices/repo-onboarding.md). Day-to-day workflow: [ai-dev-workflow.md](https://github.com/OpenShift-Fleet/agentic-sdlc/blob/main/practices/ai-dev-workflow.md).

---

## Build, Test, and Lint Commands

```bash
# Build the binary (uses promu)
make build

# Run unit tests
make test

# Run only short tests (no live PostgreSQL required)
make common-test-short

# Lint (golangci-lint)
make lint

# Check formatting
make style

# All checks + build + test (CI equivalent)
make common-all

# Konflux / production container image
podman build -f Containerfile.konflux -t postgres-exporter:local .
```

> Konflux builds use `Containerfile.konflux`: builds `promu` from source with `CGO_ENABLED=1`, then calls `promu build --cgo` to produce the exporter binary with CGO support for FIPS compliance. Published multi-arch images via `.tekton/`.

---

## Repo Layout

```text
Containerfile.konflux         Production multi-stage container build (Konflux / brew)
Makefile                      Thin wrapper — sets DOCKER_ARCHS/REPO, includes Makefile.common
Makefile.common               Prometheus-community shared build rules (promu, lint, test, style)
.tekton/                      Konflux PipelineRuns (push + pull-request for release-5.0)
cmd/postgres_exporter/        Exporter main package
collector/                    Metric collectors for PostgreSQL subsystems
config/                       Config file parser and examples (auth_modules)
postgres_mixin/               Grafana dashboards and alerting rules (upstream)
promu/                        Build-tool source vendored for hermetic Konflux builds
```

---

## Architecture

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for Global Hub integration, deployment context, Konflux CI, and configuration details.

Upstream Prometheus Community docs remain in [README.md](README.md).

---

## Personal Config

Read `.claude/user.local.md` at the start of any task that needs an assignee, email, or project key. If the file does not exist, fall back to Claude memory (`user-config`), then placeholders. Run `make personalize` in [OpenShift-Fleet/agentic-sdlc](https://github.com/OpenShift-Fleet/agentic-sdlc) to generate it.

---

## Fleet Engineering Skills

Use the Fleet plugin in Claude Code (`make install-claude` in agentic-sdlc) when available. In Cursor or other agents without the plugin, fetch and apply the relevant skill URL when the task matches its domain.

| Skill | When to use |
| ----- | ----------- |
| [start-work](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/sdlc/start-work/SKILL.md) | Begin work on a Jira ticket — creates sub-task, transitions status |
| [finish-work](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/sdlc/finish-work/SKILL.md) | Commit, push, open PR, update Jira |
| [pr-review](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/sdlc/pr-review/SKILL.md) | Review a GitHub PR with worktree isolation and inline comments |
| [pr-fix](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/sdlc/pr-fix/SKILL.md) | Fix blocked PRs: merge conflicts, CI failures, review comments |
| [jira-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira/jira-specialist/SKILL.md) | Triage, search, link, or transition Jira issues |
| [bug-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira/bug-specialist/SKILL.md) | Create a well-structured bug report with reproduction steps |
| [story-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira/story-specialist/SKILL.md) | Create a user story with acceptance criteria |
