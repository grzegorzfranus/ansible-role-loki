# Ansible Role: Loki

|Source|Version|CI|License|
|------|-------|--|-------|
|[![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-loki)|[![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-loki)](https://github.com/grzegorzfranus/ansible-role-loki/releases)|[![CI](https://github.com/grzegorzfranus/ansible-role-loki/actions/workflows/ci.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-loki/actions/workflows/ci.yml)|[![Repository License](https://img.shields.io/badge/license-apache2.0-brightgreen.svg)](LICENSE)|

This Ansible role installs, configures, and manages Grafana Loki log aggregation system in single-binary mode for enterprise infrastructure. It supports dual deployment modes (`loki_deployment_method: docker | binary`): Docker Compose stack (`community.docker.docker_compose_v2`) and native binary with hardened systemd unit.

## ✨ Features

- 📦 **Dual Deployment Modes**: Docker Compose stack (`community.docker.docker_compose_v2`) or native binary + systemd unit
- 🛡️ **Container Security Hardening**: Non-root UID/GID 10001, `read_only` rootfs, `no-new-privileges:true`, `/tmp` tmpfs mount, and memory limits
- 🔒 **Systemd Sandboxing**: `ProtectSystem=strict`, `ProtectHome=true`, `PrivateTmp=true`, `CapabilityBoundingSet=`, `NoNewPrivileges=true`
- 🧹 **Retention & Compacting**: TSDB index schema v13, 7-day retention (168h) with compactor-enforced deletion
- 🔍 **Preflight System Validation**: Memory (1024 MB min), free disk space (2048 MB min on data partition), and OS distribution verification
- 🔐 **Supply-Chain Verification**: SHA256 release checksum validation with automatic `SHA256SUMS` URL fallback
- 📝 **Optional Logging Integrations**: Optional rsyslog redirection and logrotate rules with safe `backup: true` templating
- 🧪 **Comprehensive CI Matrix**: Validated via Molecule across Rocky Linux 9, Ubuntu 24.04, and Debian 12

## 🎯 Architecture

The role provides a single-binary Grafana Loki log collector architecture supporting both Docker Compose and native binary deployment modes:

```
Agents (Alloy/Vector) ──[HTTP 3100 / Push]──> Grafana Loki (Single Binary) ──> TSDB / Chunks Store (/var/lib/loki)
                                                    │
                                                    └──[Compactor (168h)]──> Purges Expired Chunks
```

## 📋 Requirements

- **Ansible**: 2.15 or higher
- **Python**: 3.9 or higher on target hosts
- **Network**: Port 3100 open on target host (Tailscale magicDNS or LAN)
- **Privileges**: Privilege escalation enabled (`loki_become: true`, default)

### Supported Operating Systems

| OS Family | Version | Status |
|-----------|---------|---------|
| Ubuntu | 26.04 (Resolute) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Ubuntu | 24.04 (Noble) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Ubuntu | 22.04 (Jammy) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Debian | 13 (Trixie) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Debian | 12 (Bookworm) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Debian | 11 (Bullseye) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| EL (RHEL, Rocky, Alma, Oracle) | 9 | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |

### Ansible Version

Ansible >= 2.15

### Python Version

Python >= 3.9

### Setup Module

The role relies on Remote Facts gathered by Ansible setup. Disabling fact gathering will break variable cascading and validation.

### Root Access

Privilege escalation (`loki_become: true`) is required to write configuration files, create system accounts, and manage systemd or Docker services.

## 🚀 Quick Start

### 1. Basic Docker Compose Deployment (Default)

```yaml
---
- name: Deploy Grafana Loki via Docker Compose
  hosts: logging_servers
  roles:
    - role: grzegorzfranus.loki
      vars:
        loki_deployment_method: "docker"
        loki_bind_address: "100.64.0.31"
```

### 2. Native Binary & Hardened Systemd Unit Deployment

```yaml
---
- name: Deploy Grafana Loki via Native Binary
  hosts: log_collectors
  roles:
    - role: grzegorzfranus.loki
      vars:
        loki_deployment_method: "binary"
        loki_bind_address: "192.168.10.31"
```

### 3. Run the Playbook

```bash
ansible-playbook -i inventory site.yml
```

## ⚙️ Configuration

### Default Configuration

The role ships with production-ready defaults tuned for single-binary log aggregation:

```yaml
loki_state: "present"
loki_deployment_method: "docker"
loki_version: "3.6.13"
loki_image: "grafana/loki"
loki_image_tag: "3.6.13"
loki_http_port: 3100
loki_grpc_port: 9096
loki_bind_address: "127.0.0.1"
loki_storage_backend: "filesystem"
loki_retention_period: "168h"
loki_compactor_enabled: true
loki_ingestion_rate_mb: 4
loki_ingestion_burst_size_mb: 6
```

### Real-World Configuration (Tailnet Interface & Custom Retention)

```yaml
---
- name: Production Log Aggregation Node
  hosts: log-mgt-01.comlia.app
  roles:
    - role: grzegorzfranus.loki
      vars:
        loki_deployment_method: "docker"
        loki_bind_interface: "tailscale0"
        loki_retention_period: "720h"
        loki_filesystem_retention_acknowledged: true
        loki_ingestion_rate_mb: 16
        loki_ingestion_burst_size_mb: 32
        loki_compose_memory_limit: "4g"
        loki_extra_config:
          limits_config:
            max_global_streams_per_user: 50000
```

## 📊 Variables

### Service State & Deployment Mode

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_state` | Role desired state (`present` or `absent`) | `"present"` |
| `loki_remove_data` | Purge data directory on uninstallation | `false` |
| `loki_deployment_method` | Deployment mode (`docker` or `binary`) | `"docker"` |

### Image & Binary Version

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_version` | Software version tag for download and image tagging | `"3.6.13"` |
| `loki_image` | Docker image repository | `"grafana/loki"` |
| `loki_image_tag` | Docker container tag (cannot be `latest`) | `"3.6.13"` |
| `loki_binary_checksum` | Expected SHA256 checksum string for native binary (auto-fetches `SHA256SUMS` URL if empty) | `""` |

### General Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_become` | Enable task-level privilege escalation (`sudo`). Set `false` in container CI test runners | `true` |
| `loki_service_enabled` | Enable service on boot and start running | `true` |
| `loki_manage_service_restart` | Restart service on configuration template changes | `true` |

### System Account & Paths

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_user` | System user name under which Loki runs | `"loki"` |
| `loki_group` | System group name under which Loki runs | `"loki"` |
| `loki_uid` | Fixed numeric User ID | `10001` |
| `loki_gid` | Fixed numeric Group ID | `10001` |
| `loki_data_dir` | Directory for storing chunk files and database index | `"/var/lib/loki"` |
| `loki_config_dir` | Directory for configuration files | `"/etc/loki"` |
| `loki_compose_dir` | Directory for Docker Compose files | `"/etc/loki/compose"` |
| `loki_install_dir` | Binary installation destination | `"/usr/local/bin"` |

### Network & Listeners

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_http_port` | HTTP listener port for Loki API and health endpoints | `3100` |
| `loki_grpc_port` | gRPC internal listener port | `9096` |
| `loki_bind_address` | Listening IP address (`127.0.0.1` default for loopback isolation) | `"127.0.0.1"` |
| `loki_bind_interface` | Optional network interface name to bind to (e.g. `tailscale0`). Overrides `loki_bind_address` when set | `""` |

### Server Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_auth_enabled` | Enable multi-tenant authentication (`X-Scope-OrgID` requirement) | `false` |
| `loki_log_level` | Logging verbosity (`debug`, `info`, `warn`, `error`) | `"info"` |
| `loki_server_config` | Custom dictionary overrides merged into `server:` YAML block | `{}` |

### Storage & Retention

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_storage_backend` | Storage engine backend | `"filesystem"` |
| `loki_schema_version` | TSDB index schema version | `"v13"` |
| `loki_schema_from` | Schema effective start date | `"2024-01-01"` |
| `loki_schema_index_period` | Index partition duration | `"24h"` |
| `loki_retention_period` | Global log retention duration | `"168h"` |
| `loki_filesystem_retention_acknowledged` | Safety waiver for retention > 30 days on filesystem storage | `false` |
| `loki_compactor_enabled` | Enable retention compactor process | `true` |
| `loki_compactor_working_dir` | Compactor temporary processing directory | `"/var/lib/loki/compactor"` |
| `loki_delete_request_store` | Backend for tombstones | `"filesystem"` |

### Ingestion & Stream Limits

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_ingestion_rate_mb` | Ingestion rate limit per stream (MB/s) | `4` |
| `loki_ingestion_burst_size_mb` | Ingestion burst capacity (MB) | `6` |
| `loki_max_streams_per_user` | Maximum active streams per tenant | `10000` |
| `loki_max_global_streams_per_user` | Maximum global active streams | `25000` |
| `loki_max_query_series` | Maximum time series returned per query | `500` |
| `loki_chunk_target_size` | Target compressed chunk size in bytes | `1572864` |
| `loki_chunk_idle_period` | Maximum chunk idle duration before flushing | `"30m"` |

### Docker Compose Hardening

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_compose_restart_policy` | Container restart policy (`unless-stopped`, `always`, `on-failure`, `no`) | `"unless-stopped"` |
| `loki_compose_read_only` | Mount root filesystem as read-only | `true` |
| `loki_compose_log_driver` | Container logging driver | `"local"` |
| `loki_compose_memory_limit` | Container RAM limit | `"1g"` |

### Systemd Hardening

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_systemd_hardening_enabled` | Enable strict systemd security sandboxing | `true` |
| `loki_systemd_extra_options` | Additional systemd unit directives list | `[]` |

### Logging & Archival

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_configure_rsyslog` | Enable rsyslog file redirection | `false` |
| `loki_configure_logrotate` | Enable logrotate configuration | `false` |
| `loki_logrotate_options` | Logrotate parameters dictionary (`frequency`, `count`, etc.) | *See defaults/main.yml* |
| `loki_extra_config` | Raw dictionary deep-merged into rendered `loki-config.yml` | `{}` |

### System Validation Parameters

| Variable | Description | Default |
|----------|-------------|---------|
| `loki_validate_system` | Execute preflight system checks | `true` |
| `loki_min_free_disk_mb` | Minimum required disk space on data partition | `2048` |
| `loki_min_memory_mb` | Minimum required RAM | `1024` |

## 📌 Role Properties

| Property | Value | Description |
|----------|-------|-------------|
| **Idempotent** | ✅ Yes | Executing the role multiple times produces zero changes after initial convergence. |
| **Atomic** | ❌ No | Roles may leave rendered configuration files on disk if an error occurs mid-execution. |
| **Check Mode** | ✅ Supported | Read-only syntax and template validation runs cleanly under `--check`. |
| **Diff Mode** | ✅ Supported | Configuration and systemd unit changes render inline diffs under `--diff`. |

## 📤 Role Output

This role does not set any public facts on target hosts. Internal computed facts use double-underscore prefix `__loki_*` (e.g. `__loki_effective_bind_address`, `__loki_arch`).

## 🔍 Verification

### 1. HTTP Readiness & API Probing

```bash
# Verify Loki HTTP /ready status endpoint (returns 200 OK ready)
curl -s -i http://127.0.0.1:3100/ready

# Inspect Loki metrics endpoint
curl -s http://127.0.0.1:3100/metrics | head -n 20
```

### 2. Service & Process Inspection

```bash
# Binary mode: inspect systemd service status
sudo systemctl status loki --no-pager

# Docker mode: inspect compose container status
sudo docker compose -f /etc/loki/compose/docker-compose.yml ps
```

### 3. Log Inspection

```bash
# Binary mode: view systemd journal logs
sudo journalctl -u loki -n 50 --no-pager

# Docker mode: view container logs
sudo docker compose -f /etc/loki/compose/docker-compose.yml logs --tail 50
```

## 🛡️ Security Features

- ✅ **Least-Privilege Execution**: Dedicated system account `loki:loki` with fixed numeric UID/GID `10001`.
- ✅ **Strict Permission Triads**: Configurations set to `0640` (owned by `root:loki`), binary to `0755`, data directories to `0750`.
- ✅ **Container Sandboxing**: `read_only: true`, `no-new-privileges:true`, memory limits, and `/tmp` tmpfs mount.
- ✅ **Systemd Isolation**: `ProtectSystem=strict`, `ProtectHome=true`, `PrivateTmp=true`, `CapabilityBoundingSet=`, `NoNewPrivileges=true`.
- ✅ **Integrity Checking**: SHA256 checksum verification for native binary downloads.

## 🧪 Check Mode Behavior

Under `--check` (Check Mode):
- OS distribution, RAM, and free disk space preflight assertions run normally.
- Template rendering and file creation tasks report planned file changes without modifying disk state.
- Systemd service start/restart and Docker Compose container operations are safely skipped.

## 🏷️ Tags

All tags are prefixed with `loki_` to avoid namespace collisions.

| Tag | Description |
|-----|-------------|
| `always` | Tasks that run on every execution (OS variable gathering, bind address resolution) |
| `loki_setup` | Core setup pipeline (assertions, prerequisites, user creation, directories) |
| `loki_validate` | System preflight RAM, disk space, and OS distribution validation |
| `loki_remove` | Uninstallation and data purge tasks |
| `loki_install` | Installation tasks (binary download or Docker image pull) |
| `loki_configure` | Configuration rendering, systemd unit, and Compose stack setup |
| `loki_service` | Running service state management and HTTP `/ready` readiness probing |

## 🌐 Network Resilience

Loki listens on HTTP port `3100` (API/Ingest) and gRPC port `9096` (internal inter-component). Ensure firewalls allow incoming TCP 3100 traffic from authorized log forwarder agents (Alloy/Vector) or Tailscale network interfaces.

## 🔧 Troubleshooting

### Port Binding Failure

```bash
# Check if port 3100 is already bound by another process
sudo ss -tulpn | grep 3100

# Verify bind address IP exists on host
ip addr show
```

### Permission Denied on Data Directory

```bash
# Verify UID/GID 10001 ownership on data directory
ls -ld /var/lib/loki

# Fix ownership manually if modified externally
sudo chown -R loki:loki /var/lib/loki
```

### Compactor Retention Not Purging Logs

```bash
# Verify retention_enabled is set to true in rendered config
grep -A 5 "compactor:" /etc/loki/loki-config.yml

# Inspect compactor working directory
ls -la /var/lib/loki/compactor
```

## 📁 File Structure

```
ansible-role-loki/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml
│   │   ├── config.yml
│   │   ├── feature_request.yml
│   │   └── task.yml
│   ├── PULL_REQUEST_TEMPLATE/
│   │   └── pull_request_template.md
│   └── workflows/
│       ├── ci.yml                     # Reusable CI workflow integration
│       └── release.yml                # Release Please workflow integration
├── defaults/
│   └── main.yml                       # Role default configuration variables
├── handlers/
│   └── main.yml                       # Service restart and systemd reload handlers
├── meta/
│   ├── argument_specs.yml             # Native argument specification schema
│   └── main.yml                       # Role metadata and dependencies
├── molecule/
│   ├── config-override/              # Custom port, retention, and extra_config test scenario
│   ├── default/                      # Dual binary live + docker template pass scenario
│   └── uninstall/                    # Service removal and data purge test scenario
├── tasks/
│   ├── main.yml                       # Main task orchestration
│   ├── assert.yml                     # Variable assertion ladder
│   ├── compose.yml                    # Docker Compose configuration rendering
│   ├── configure.yml                  # Main loki-config.yml rendering
│   ├── directories.yml                # System directory creation
│   ├── install.yml                    # Installation dispatcher
│   ├── install_binary.yml             # Binary download & checksum extraction
│   ├── install_docker.yml             # Docker image pull & role inclusion
│   ├── logging.yml                    # Optional rsyslog and logrotate configuration
│   ├── prerequisites.yml              # Base package installation
│   ├── remove.yml                     # Uninstallation tasks
│   ├── service.yml                    # Service state management & readiness probe
│   ├── systemd.yml                    # Systemd service unit setup
│   ├── user.yml                       # System user and group creation
│   └── validate.yml                   # System preflight checks
├── templates/
│   ├── compose/
│   │   └── docker-compose.yml.j2      # Docker Compose stack template
│   ├── logrotate/
│   │   └── loki.j2                    # Logrotate configuration template
│   ├── loki/
│   │   └── loki-config.yml.j2         # Grafana Loki main configuration template
│   ├── rsyslog/
│   │   └── loki.conf.j2               # Rsyslog configuration template
│   └── systemd/
│       └── loki.service.j2            # Systemd service unit template
├── vars/
│   ├── debian_11.yml                  # Debian 11 specific variables
│   ├── debian_12.yml                  # Debian 12 specific variables
│   ├── debian_13.yml                  # Debian 13 specific variables
│   ├── redhat_9.yml                   # Enterprise Linux 9 specific variables
│   ├── ubuntu_22.04.yml               # Ubuntu 22.04 specific variables
│   ├── ubuntu_24.04.yml               # Ubuntu 24.04 specific variables
│   └── ubuntu_26.04.yml               # Ubuntu 26.04 specific variables
├── .ansible-lint                      # Ansible lint configuration
├── .ansible-vars-validate.yml         # Variable consistency check waivers
├── .yamllint                          # YAML lint configuration
├── LICENSE                            # Apache-2.0 License
└── README.md                          # Role documentation
```

## CI/CD Pipeline

This repository uses centralized, reusable GitHub Actions workflows from [grzegorzfranus/github-workflows](https://github.com/grzegorzfranus/github-workflows) (`v3.1.1`) for quality assurance, security scanning, and release automation.

### CI Pipeline (`ansible-ci.yml@v3.1.1`)

Runs on every Pull Request in a multi-tier gate pattern:
1. **PR Title & Commit Lint** — enforces [Conventional Commits](https://www.conventionalcommits.org/) format (`feat:`, `fix:`, etc.)
2. **YAML Syntax Lint** — validates YAML formatting via `yamllint`
3. **Ansible Lint** — checks Ansible best practices and role standards
4. **Galaxy Metadata & Variable Validation** — verifies `meta/main.yml` schema and 4-surface variable consistency
5. **Security Scanning** — TruffleHog secret detection and Trivy IaC scanning
6. **Molecule Integration Tests** — executes Molecule test scenarios (`default`, `config-override`, `uninstall`)
7. **Merge Check Gate** — single authoritative status check aggregating all workflow checks

### Release Pipeline (`ansible-publish.yml@v3.1.1`)

Automated via [Release Please](https://github.com/googleapis/release-please):
1. **Push to `main`** → Release Please creates or updates a Release PR with automated changelog generation
2. **Merge Release PR** → creates Git version tag and GitHub Release automatically

## Example Playbooks

### Docker Compose Mode Playbook

```yaml
---
- name: Deploy Grafana Loki Stack
  hosts: logging_servers
  roles:
    - role: grzegorzfranus.loki
      vars:
        loki_deployment_method: "docker"
        loki_bind_address: "10.0.0.5"
        loki_retention_period: "168h"
```

### Native Binary Mode Playbook

```yaml
---
- name: Deploy Grafana Loki Native Binary
  hosts: log_collectors
  roles:
    - role: grzegorzfranus.loki
      vars:
        loki_deployment_method: "binary"
        loki_bind_address: "127.0.0.1"
        loki_retention_period: "72h"
```

### Uninstall Playbook

```yaml
---
- name: Uninstall Grafana Loki
  hosts: logging_servers
  roles:
    - role: grzegorzfranus.loki
      vars:
        loki_state: "absent"
        loki_remove_data: true
```

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your feature branch from `main`
- Use [Conventional Commits](https://www.conventionalcommits.org/) format for commit messages (`feat: ...`, `fix: ...`)
- Ensure code passes all quality checks (`yamllint .`, `ansible-lint`, `molecule test -s default`)
- Open a Pull Request referencing your target issue (`Closes #1` in PR body)

## 📝 License

This project is licensed under the Apache-2.0 License - see the [LICENSE](LICENSE) file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).