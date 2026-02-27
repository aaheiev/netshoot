# Netshoot Project Instructions for GitHub Copilot

You are an AI assistant working on the `netshoot` project, a comprehensive Docker and Kubernetes network troubleshooting container. Your goal is to help maintain and enhance this "swiss-army knife" for network engineers and developers.

## Project Overview
- **Name**: `netshoot`
- **Purpose**: Provide a single container image pre-loaded with powerful networking tools to troubleshoot complex Docker and Kubernetes networking issues.
- **Base Image**: Uses `alpine` Linux (specifically edge/testing repositories are often enabled for latest tools).
- **Maintainer**: nicolaka

## Core Principles & Coding Standards

### 1. Dockerfile Best Practices
- **Base Image**: Always base new stages or images on `alpine` to keep the footprint small, unless a specific tool requires `debian`/`ubuntu` (use multi-stage builds in that case).
- **Package Management**:
  - Use `apk add --no-cache` to avoid caching index locally.
  - Group packages alphabetically in `RUN` instructions for readability.
  - Clean up temporary files in the same layer where they are created.
- **Multi-Stage Builds**:
  - Use a `fetcher` or `builder` stage (e.g., `FROM debian:stable-slim as fetcher`) to download binaries or compile tools that are not in Alpine repositories.
  - `COPY` only the necessary artifacts (binaries, configs) to the final Alpine stage.
- **Layers**: Combine commands using `&&` and `\` to minimize the number of layers.

### 2. Shell Scripting (Bash)
- **Shebang**: Use `#!/usr/bin/env bash` for portability.
- **Strict Mode**: Always start scripts with `set -euo pipefail` to catch errors early.
- **External Downloads**:
  - When fetching binaries (like in `build/fetch_binaries.sh`), assume `curl` or `wget` is available.
  - Dynamically fetch the latest version using GitHub API if possible (e.g., `get_latest_release` function), or allow version pinning via variables.
  - Handle architecture differences (`amd64` vs `arm64`) explicitly using `uname -m`.

### 3. Makefile
- **Targets**: Use simple, standard targets (`build`, `push`, `all`).
- **Phony Targets**: Always declare `.PHONY` for non-file targets.
- **Variables**: Use variables for image names and versions (e.g., `IMAGENAME`, `VERSION`).
- **Silence**: Use `@` to silence command output where appropriate.
- **Multi-Arch**: Support `docker buildx` for multi-architecture builds (linux/amd64, linux/arm64).

## Tooling Awareness
Copilot should be aware that the following tools are (or should be) available in the `netshoot` container. When suggesting troubleshooting steps or enhancements, prefer using these tools:

- **Network Analysis**: `tcpdump`, `tshark`, `termshark`, `ngrep`
- **Connectivity Testing**: `curl`, `wget`, `httpie`, `netcat-openbsd`, `socat`, `websocat`
- **Routing & Stats**: `iproute2` (ip, ss, bridge), `conntrack-tools`, `iftop`, `iptraf-ng`, `ctop`
- **DNS**: `bind-tools` (dig, host, nslookup), `drill`
- **Performance**: `iperf`, `iperf3`, `speedtest-cli`, `fortio`
- **Security/Scanning**: `nmap`, `nmap-nping`, `nmap-scripts`
- **Kubernetes/Cloud**: `calicoctl`
- **Shell**: `zsh` (with oh-my-zsh and plugins), `bash`, `vim`

## Common Tasks & Scenarios
- **Updating Tools**: When asked to update a tool, check if it's installed via `apk` (update `Dockerfile` package list) or manually fetched (update `build/fetch_binaries.sh`).
- **Adding New Tools**:
  1. Check if available in Alpine `edge/testing` or `community` repos.
  2. If not, add a function to `fetch_binaries.sh` to download from official releases.
  3. Update `Dockerfile` to copy the binary from `fetcher` stage.
- **Troubleshooting Examples**: When generating documentation or examples, show how to run `netshoot` in different network namespaces:
  - Host network: `docker run -it --net host netshoot`
  - Container network: `docker run -it --net container:<id> netshoot`

