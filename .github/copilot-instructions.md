# Copilot Instructions for Samba AD & File Server Docker Environment

## Overview
Docker-based environment for Samba Active Directory (DC) and file server (FS), optimized for home servers/small businesses. Core design: externalized config via bind-mounted volumes, multi-stage Docker build with role-specific targets, separation of concerns between DC and FS containers.

## Critical Architecture Concepts

**Multi-Stage Build Pattern:** Single `Dockerfile` with `dc` and `fs` build targets. Both stages inherit from `base`, which installs core Samba/Kerberos packages. DC target adds `init-dc.sh` + `samba-provision.sh`; FS target adds `init-fs.sh` + `samba-join.sh`.

**Config Externalization Strategy:** All persistent data stored in bind-mounted host directories (`config-dc1/`, `data-dc1/`, `config-fs1/`, `data-fs1/`, `shares-fs1/`). Containers are stateless, allowing rebuilds without data loss. Symlink pattern: provisioning scripts create symlinks from `/etc/samba/config/smb.conf` (host-mounted) to `/etc/samba/smb.conf` (container expected path).

**Networking:** `macvlan` network on `sambanet`, static container IPs from environment variables (`DC_IP`, `FS_IP`). DNS configured with bidirectional forwarding: containers forward unknown queries to `MAIN_DNS_IP`; host DNS may forward AD zone to DC IP for domain clients.

## Script Execution Flow

**Key Pattern:** All scripts source `dockerfile/vars.sh` first. This centralizes environment setup: transforms `DOMAIN_FQDN` to lowercase/uppercase variants, defines `CONFIG_DIR` paths, constructs NetBIOS domain name from FQDN prefix.

**DC Startup Flow:** 
1. `init-dc.sh` checks if `smb.conf` exists in `config-dc1/`
2. If exists: copies Kerberos config from provisioned data, symlinks smb.conf, runs `samba --interactive --no-process-group`
3. If missing: exits with error message suggesting manual `samba-provision.sh` execution

**FS Startup Flow:** Similar check pattern; if FS already domain-joined, starts normally.

**Provisioning Workflow:** Non-automatic. Only triggers via manual `docker exec`:
```bash
docker exec -it dc1 /usr/helpers/samba-provision.sh  # Provision DC
docker exec -it fs1 /usr/helpers/samba-join.sh      # Join FS to domain
```

## Conventions & Patterns

- **Environment-driven config:** Injected via `docker-compose.yml` environment section (DOMAIN_FQDN, DC_IP, FS_IP, MAIN_DNS_IP, etc.)
- **Privileged mode for DC:** Required for `vfs_acl_xattr` module (security.* namespace access)
- **FS unprivileged by default:** Can use `acl_xattr:security_acl_name = user.NTACL` for Windows ACLs without root
- **User mapping:** `user.map` in `config-dc1/` or `config-fs1/` maps LDAP users to Unix accounts
- **Config file locations:** Samba looks for `smb.conf` at `/etc/samba/smb.conf` (symlinked); actual config on host in `config-dc1/samba/` or `config-fs1/samba/`

## When Modifying Scripts

- Update environment variable handling in `vars.sh` → affects all downstream scripts
- Add provisioning logic → modify `samba-provision.sh` (DC only) or `samba-join.sh` (FS only)
- Change startup checks → modify `init-dc.sh` or `init-fs.sh` (role-specific) or `init.sh` (common logic)
- Docker build stages → add packages to `base` stage; role-specific tools in `dc` or `fs` targets
- Volumes/config paths → update both `docker-compose.yml` AND any script references (they must match)

## Integration Points

- **AD as Auth Backend:** Can integrate with Authelia or other OIDC/LDAP clients for SSO; AD user objects created via Samba LDAP
- **DNS Forwarding:** Requires external DNS server configured to forward `DOMAIN_FQDN` zone to `DC_IP`; DC forwards root queries to `MAIN_DNS_IP`
- **Domain Member Dynamic Updates:** If FS is domain-joined member, can update DNS via Kerberos; requires DC DNS forwarding setup
- **Windows Clients:** Join domain like any Samba AD; use Kerberos tickets from `krb5.conf` created during provisioning