# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a macOS-compatible demo for running **Spring Application Advisor (SAA)** upgrade pipelines via **Concourse CI**. It automatically discovers Java/Spring Boot repos across GitHub organizations, detects version drift, and creates pull requests with upgrades.

## Running the Demo

```bash
# Prerequisites: Docker Desktop, vendir, httpie
brew install vendir httpie

# Configure secrets in .envrc (uses Bitwarden CLI)
source .envrc

# Run the full demo
./demo.sh
```

`demo.sh` orchestrates everything: syncs vendored deps, starts Concourse + Nexus via Docker Compose, installs the `fly` CLI, configures teams, and sets up the spawner pipeline.

## Key Commands

- **Start infrastructure**: `./demo.sh` (Concourse at `localhost:8080`, Nexus at `localhost:8081`)
- **Build/push runner image**: `cd docker && docker build -t scpd-runner:latest .` (then tag/push to Docker Hub)
- **Manage demo repos**: `./repo-management.sh` (deletes all repos in target org, forks source repos, sets up notifications)
- **Sync vendored deps**: `vendir sync`
- **Fly CLI** (after demo starts): `./upgrade-example/fly -t advisor-demo <command>`

## Architecture

### Pipeline System (two-tier Concourse design)

1. **Spawner pipeline** (`pipelines/spawner-pipeline.yml`): Runs every 15 minutes. Crawls GitHub orgs (from `GITHUB_ORGS` env var), discovers non-archived repos, and dynamically creates a per-repo pipeline for each one using Concourse's `set_pipeline` + `across` steps.

2. **Repo pipeline** (`pipelines/repo-pipeline.yml`): Per-repo pipeline running every 12 hours. Uses the custom `scpd-runner` Docker image. Checks for open PRs first (skips if any exist), then runs Spring Application Advisor to detect and apply upgrades. Falls back to OpenRewrite for patch-level Spring Boot upgrades when SAA finds no upgrade plan.

### Custom Runner Image (`docker/`)

Ubuntu-based image with SDKMAN (Java 21), Maven settings for Spring Enterprise repos (`packages.broadcom.com`), `gh` CLI, and standard build tools. The `settings.xml` has placeholder `MAVEN_USERNAME`/`MAVEN_PASSWORD` tokens that get replaced at pipeline runtime via `sed`.

### Secrets Management

All secrets are sourced from **Bitwarden CLI** via `.envrc`. Required secrets: GitHub token, Maven credentials (Broadcom Spring Enterprise), Docker Hub credentials. These are passed to Concourse pipelines as `fly` variables.

## macOS Compatibility Notes

- All `sed -i` commands use BSD syntax: `sed -i ''` (not GNU `sed -i`)
- Use `grep -oE` (POSIX ERE) instead of `grep -oP` (Perl regex)
- Docker Compose: Linux-specific settings (`privileged: true`, `cgroup: host`) are stripped at runtime
- Local Java version is 8 (`.sdkmanrc`), but pipeline runner uses Java 21
