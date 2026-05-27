# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the PostgreSQL Server Exporter for Prometheus, written in Go. It exports PostgreSQL metrics for monitoring with Prometheus. The project supports PostgreSQL versions 11-16 and includes multi-target functionality for monitoring multiple PostgreSQL instances from a single exporter instance.

## Build and Development Commands

### Building
- `make build` - Build the binary using promu
- `make promu` - Install the promu build tool
- `make docker` - Build Docker image (requires promu crossbuild first)

### Testing
- `make test` - Run all tests
- `make test-short` - Run short tests only
- Integration tests: `DATA_SOURCE_NAME='postgresql://postgres:test@localhost:5432/circle_test?sslmode=disable' GOOPTS='-v -tags integration' make test`

### Code Quality
- `make lint` - Run golangci-lint
- `make style` - Check code formatting with gofmt
- `make format` - Format code with go fmt
- `make vet` - Run go vet
- `make check_license` - Verify license headers
- `make yamllint` - Check YAML files

### Dependencies
- `make deps` - Download dependencies
- `make unused` - Check for unused dependencies
- `update-go-deps` - Update Go dependencies

## Architecture

### Core Components

**Main Entry Point**: `cmd/postgres_exporter/main.go`
- Uses kingpin for CLI argument parsing
- Supports both single-target and multi-target modes
- Configuration via flags, environment variables, or YAML config file

**Collector Framework**: `collector/` package
- `collector.go` - Base collector interface and factory system
- Each collector handles specific PostgreSQL metrics (e.g., `pg_stat_database`, `pg_locks`)
- Collectors can be enabled/disabled via command-line flags
- Uses Prometheus client library for metric definitions

**Configuration**: `config/` package
- `config.go` - Main configuration structure
- `dsn.go` - Data Source Name handling for PostgreSQL connections
- Supports authentication modules for multi-target setups

### Collector Types

Individual collectors in `collector/` directory handle different PostgreSQL statistics:
- `pg_database.go` - Database-level metrics
- `pg_stat_database.go` - Database statistics
- `pg_stat_bgwriter.go` - Background writer stats
- `pg_locks.go` - Lock information
- `pg_replication.go` - Replication metrics
- And many others for specific PostgreSQL features

### Multi-Target Support

The exporter supports monitoring multiple PostgreSQL instances via the `/probe` endpoint with target parameter. Authentication modules in the config file enable secure credential management without exposing passwords in URLs.

## Key Configuration

- Default config file: `postgres_exporter.yml`
- Primary environment variable: `DATA_SOURCE_NAME` for PostgreSQL connection
- Default metrics endpoint: `:9187/metrics`
- Custom queries can be defined in `queries.yaml` (deprecated feature)

## Testing Setup

For integration tests, you need a running PostgreSQL instance:
```bash
docker run -p 5432:5432 -e POSTGRES_DB=circle_test -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=test -d postgres
```

## Development Notes

- The project uses Go modules (`go.mod`)
- Follows Prometheus exporter conventions
- All collectors implement the `Collector` interface
- Metric names use the `pg_` prefix by default
- Code must include Apache 2.0 license headers
- Uses the prometheus-community Docker repository for images