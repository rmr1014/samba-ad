# Copilot Instructions for Samba AD & File Server Docker Environment

## Overview
This project provides a modern, secure, and modular Docker-based environment for running Samba Active Directory (AD) domain controllers and file servers. It is designed for home servers and small businesses, with a focus on reliability, upgradability, and clear separation of concerns.

## Architecture
- **Two main containers:**
  - `dc1`: Samba AD Domain Controller
  - `fs1`: Samba File Server
- **Separation:** Each container is built from a shared Dockerfile with distinct build targets (`dc`, `fs`).
- **Configuration:** All persistent data and configuration are stored outside containers via bind-mounted volumes.
- **Networking:** Custom Docker network (`sambanet`) with static IPs and DNS forwarding between host and containers.

## Key Files & Directories
- `docker-compose.yml`: Defines services, volumes, environment variables, and networking.
- `dockerfile/`: Contains Dockerfile and initialization scripts for provisioning and joining AD domains.
  - `Dockerfile`: Multi-stage build for DC and FS roles.
  - `init-dc.sh`, `init-fs.sh`, `init.sh`: Entrypoints for container startup.
  - `samba-provision.sh`, `samba-join.sh`: Scripts for AD domain provisioning and joining.
  - `vars.sh`: Centralized environment variable definitions.

## Developer Workflows
- **Build & Start:**
  - Use `docker-compose up --build` to build and launch containers.
- **Provision AD Domain:**
  - If not provisioned, exec into DC container and run `samba-provision.sh`.
- **Join File Server to Domain:**
  - Exec into FS container and run `samba-join.sh`.
- **Configuration Files:**
  - Samba config (`smb.conf`) and Kerberos config (`krb5.conf`) are managed by scripts and symlinked to expected locations.

## Patterns & Conventions
- **Scripts source `vars.sh`** for environment setup.
- **Config files are symlinked** to ensure Docker host and container paths match.
- **Provisioning checks**: Scripts check for config existence before running setup.
- **Custom user mapping**: `user.map` is referenced in Samba config for username mapping.
- **POSIX and Windows ACLs**: File server supports both, with special config for Windows ACLs if running unprivileged.

## Integration Points
- **External DNS**: Forwarding between AD DC and main DNS server is required.
- **IAM Tools**: AD can be used for SSO with tools like Authelia.

## Examples
- To provision a new domain:
  ```bash
  docker exec -it <dc-container> bash
  /usr/helpers/samba-provision.sh
  ```
- To join a file server:
  ```bash
  docker exec -it <fs-container> bash
  /usr/helpers/samba-join.sh
  ```

## References
- See [README.md](../README.md) for detailed features and blog post links.
- Key scripts: `dockerfile/init-dc.sh`, `dockerfile/init-fs.sh`, `dockerfile/samba-provision.sh`, `dockerfile/samba-join.sh`

---
**Feedback requested:** Please review and suggest clarifications or additions for unclear or incomplete sections.