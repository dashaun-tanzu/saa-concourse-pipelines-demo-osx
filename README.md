# Continuous Migration with Concourse and Spring Application Advisor (macOS)

This is a macOS-compatible version of the original [saa-concourse-pipelines-demo](../saa-concourse-pipelines-demo).

## macOS-Specific Notes

This version has been adapted to work on macOS with the following compatibility changes:

- **GNU sed**: All `sed -i` commands use BSD-compatible syntax with empty backup suffix (`sed -i ''`)
- **grep regex**: Changed `grep -oP` (Perl regex) to `grep -oE` (POSIX ERE)
- **Docker Compose**: Linux-specific settings (`privileged: true`, `cgroup: host`) are automatically removed at runtime

## Prerequisites

- macOS (tested on current version)
- [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)
- [vendir](https://carvel.dev/vendir/)
- [http](https://github.com/jkbrzt/httpie) (or `curl`)

## Setup

1. Install dependencies:
   ```bash
   brew install vendir httpie
   ```

2. Configure environment variables (see `.envrc` in parent directory)

3. Run the demo:
   ```bash
   ./demo.sh
   ```

## Differences from Linux Version

- Docker Compose runs in Docker Desktop (VM-based, not native)
- File system paths use macOS conventions
- Bash version: Use Homebrew bash (4.x+) if available, otherwise `/bin/bash` (3.2)

## Troubleshooting

### Docker Compose errors
If you see errors about `privileged` or `cgroup`:
- The script should automatically remove these settings
- Verify you have permission to run Docker commands: `docker ps`

### sed errors
If you see `sed: 1: ...: invalid command code`:
- Ensure you're using the macOS-compatible version of this repo

### Command not found
Install missing tools via Homebrew:
```bash
brew install <tool>
```

## Original Repository

The original Linux version is located at: `../saa-concourse-pipelines-demo`
