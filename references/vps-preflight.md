# VPS preflight and host safety

## Inventory before changing anything

Capture the current state in a private work log, redacting secrets:

- OS/release, kernel, uptime, time zone/time sync, CPU/RAM/swap/disk/inodes.
- Active users, sudo policy, SSH effective configuration, authorized keys ownership/modes, firewall rules, listening sockets, and recent authentication events.
- Docker Engine and Compose versions; containers, image IDs, labels, health, networks, mounts, published ports, volumes, and compose project paths.
- Reverse proxy sites/routes, upstreams, certificate names/expiry/renewal, DNS A/AAAA/CNAME, and which other applications share each resource.
- Backups, destination, age, encryption, retention, and most recent successful restore test.

Do not print private key contents, passwords, `.env` files, bearer tokens, or database URLs. Prefer commands that show variable names or file metadata only.

## Access hardening

1. Create a named admin user with sudo and verify a second SSH session works before closing the existing session.
2. Install a client-generated public key for that user; verify key login explicitly.
3. Prefer a separate deploy account with only the access needed for releases. Membership in the Docker group is effectively root; use a tightly controlled deploy mechanism if avoiding that privilege matters.
4. Only after key access is proven and an alternate recovery path exists, disable password authentication/root SSH as the host policy permits.
5. Configure a firewall with allow rules for current SSH access and intended web traffic before enabling it. Test from a second connection to avoid lockout.
6. Patch the supported OS, remove unused services carefully, configure time sync and log retention, and monitor storage. Avoid an unattended major OS upgrade during an application rollout.

Check current official OpenSSH and distribution guidance before changing daemon settings. Validate configuration before reload and keep an existing session open until a new login succeeds.

## Host compromise

Treat unexpected privileged users, unknown SSH keys, modified system binaries, cryptominers, persistence services, or unexplained outbound traffic as compromise indicators. Do not trust cleanup alone. Preserve relevant logs and disk evidence if investigation is needed; isolate the host where feasible; rebuild from a clean, trusted OS image; restore only verified data; rotate credentials that were present on or accessible from the host; then reissue host keys and validate applications. Tell the owner when live service may be interrupted. Do not deploy new secrets or claim a suspect host is clean just because a process stopped.

## Shared-resource map

Before any removal or reconfiguration, map each app to its proxy hostname, Compose project, containers, image, persistent volumes, bind mounts, networks, database/service, and backup. Inspect labels and compose files where available. Shared networks and databases are not app-owned merely because one app uses them. Never run broad `docker system prune`, volume deletion, recursive path deletion, or proxy replacement as a shortcut. Resolve the exact target paths and ownership first; back up before deletion.
