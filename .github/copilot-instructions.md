# Copilot Instructions — Jenkins Images Mono-repo

This file gives focused, repo-specific guidance for AI coding agents working in this repository. Keep instructions concise and actionable; reference the files below for examples.

## Big picture
- Two primary image families live side-by-side:
  - Controller images: [docker/](docker/)
  - Agent images: [docker-agents/](docker-agents/)
- The repo produces Docker images (multi-platform via `docker bake`/Buildx), shell and PowerShell entrypoints, and test suites (BATS + PowerShell). See platform folders (e.g., [docker/alpine/hotspot/Dockerfile](docker/alpine/hotspot/Dockerfile)).

## Key locations (examples)
- Entrypoints & helpers: [docker/jenkins.sh](docker/jenkins.sh), [docker/jenkins.ps1](docker/jenkins.ps1), [docker/jenkins-support](docker/jenkins-support)
- Controller Dockerfiles: [docker/alpine/](docker/alpine/), [docker/debian/](docker/debian/), [docker/rhel/](docker/rhel/)
- Agent Dockerfiles & READMEs: [docker-agents/](docker-agents/) and [docker-agents/README_agent.md](docker-agents/README_agent.md)
- Tests: [docker/tests/](docker/tests/) and [docker-agents/tests/](docker-agents/tests/) — BATS for *nix, PowerShell tests under `*.Tests.ps1`.
- Update automation: [*/updatecli/updatecli.d/*.yaml](docker/updatecli/updatecli.d/) and [docker-agents/updatecli/updatecli.d/]

## Build & test quick commands (concrete)
- Build a controller locally (single-platform):

```bash
cd docker
docker build -t jenkins/jenkins:local-lts -f alpine/hotspot/Dockerfile .
```

- Build an agent image:

```bash
cd docker-agents
docker build -t jenkins/inbound-agent:local -f alpine/Dockerfile .
```

- Multi-platform / canonical builds: use the repository `docker-bake.hcl` files (`docker/docker-bake.hcl`, `docker-agents/docker-bake.hcl`) and `docker buildx bake`.
- Run tests:

```bash
# BATS (Linux/macOS)
bats docker/tests/

# PowerShell tests
pwsh -File docker/tests/functions.Tests.ps1
```

## Project-specific conventions (must follow)
- Plugins: installed via the `jenkins-plugin-cli`; packaged plugins live under `/usr/share/jenkins/ref/plugins/` in the image. See plugin-related folders in `docker/tests/plugins-cli/`.
- Initialization Groovy: runtime controller init scripts go in `/usr/share/jenkins/ref/init.groovy.d/` — use this pattern for baked-in configuration.
- Persistent data: always assume Jenkins home is `/var/jenkins_home` and prefer named volumes over host bind-mounts in CI/tests.
- Exposed ports: inbound TCP agents expect `50000` by default; verify with `JENKINS_SLAVE_AGENT_PORT` in the Dockerfile/entrypoint if changed.
- Golden tests: golden expectations live under `tests/golden/` in each component (e.g., [docker/tests/golden/](docker/tests/golden/)). Use `update-golden-file.sh` to refresh intentionally.

## Integration & automation notes
- `updatecli` jobs and YAML describe automatic bumps for base images, JDKs, and remoting. Edit `updatecli/updatecli.d/*.yaml` to add rules.
- CI uses GitHub Actions in subfolders (`docker-agents/.github/workflows/`); check workflows for release and DockerHub description updates (they reference `README_agent.md`).
- JDK downloads and installation helper scripts live under `docker/` and `docker-agents/tools` (e.g., `jdk-download.sh`, `adoptium-install-jdk.sh`); prefer those scripts during image builds.

## When modifying images
- Always run the relevant test suite (BATS/PowerShell) in the same subfolder where the Dockerfile lives (`docker/` or `docker-agents/`).
- If you change behavior that affects user-facing docs or README snippets, update the corresponding `README.md` near the Dockerfile and the `updatecli` YAML if it impacts automatic bumps.

## Examples of precise patterns to follow
- To add a new platform variant: copy an existing platform folder (e.g., `docker/alpine/hotspot`), update Dockerfile, update `docker-bake.hcl`, add tests under `docker/tests/`, and add updatecli rules.
- To change a plugin set: modify the plugins list file used with `jenkins-plugin-cli` (see `docker/tests/plugins-cli/pluginsfile/`) and validate with the plugins-cli test suite.

## Helpful files to open when confused
- [docker/.github/copilot-instructions.md](docker/.github/copilot-instructions.md) — existing, controller-focused guidance
- [docker/README.md](docker/README.md) and [docker-agents/README.md](docker-agents/README.md) — higher-level usage and examples
- [docker/tests/test_helpers.bash](docker/tests/test_helpers.bash) — test helper patterns and conventions

If any of these paths are unclear or you want the agent to follow additional constraints (commit style, CI gating rules, or release steps), tell me which area to expand. Ready to iterate on wording or add CI-specific rules.
