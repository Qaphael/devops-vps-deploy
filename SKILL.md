---
name: devops-vps-deploy
description: Plan, prepare, deploy, secure, update, troubleshoot, roll back, back up, restore, or safely remove web applications on self-managed Linux VPSs. Use whenever the user mentions deploying an app to a VPS, Docker Compose, GitHub Actions deployment, a domain/reverse proxy/TLS, production updates, server hardening, backup/restore, or cleaning up hosted apps. Guide the work from repository readiness through CI, host readiness, DNS and HTTPS, live verification, and ongoing operations; inspect before changing shared infrastructure.
compatibility: Requires repository and shell access for implementation. Internet access is useful for checking current official documentation and stable versions. Deployment credentials and a reachable target host are required only for live changes.
---

# VPS application delivery

Use this skill to take an application from a code repository to a maintainable, observable, recoverable VPS deployment. Prefer a small, reproducible Docker Compose setup; do not introduce Kubernetes, managed paid services, queues, or extra platforms unless the user asks or the application requires them.

## First establish the task and target

Identify whether the user wants an assessment, first deployment, CI/CD, update, incident diagnosis, rollback, backup/restore, security hardening, or removal. Inspect the repository and the actual target before proposing changes. Read project instructions and deployment files first. Check current official documentation for version-sensitive tools and APIs.

Inventory before acting:

- Repository, branch, working tree, tests, runtime, build output, migrations, health endpoint, required environment variables, and existing Docker configuration.
- VPS OS, user and sudo access, disk/memory/CPU, Docker/Compose versions, firewall, listening ports, running services and containers, proxy configuration, certificates, DNS records, and existing sites.
- For each container, inspect image, labels, networks, mounts, ports, volumes, and database dependencies. Establish whether any resource is shared before changing or deleting it.
- Determine whether the host has evidence of compromise. If it does, treat it as untrusted: recommend preserving evidence as needed, rebuilding from a trusted image, and rotating exposed credentials. Do not characterize a cleanup of a suspect host as equivalent to rebuilding it.

Report what is known, unknown, shared, and potentially destructive. For a plan-only request, stop at a concrete plan. For authorized implementation, write a short plan before changes and work one stage at a time. Ask only when required information is missing or the action could destroy data or affect other applications. Never push, alter production data, reset a database, remove shared resources, or make an unapproved public release.

## Delivery sequence

1. **Prepare code.** Confirm reproducible install/build commands, lockfile, environment variable inventory, production start command, health endpoint, graceful shutdown, structured logs, security headers, and migration strategy. Add `.dockerignore`; use a multi-stage non-root image where practical. Keep dev seed data and bootstrap credentials out of production. Ensure secrets are not in source, image layers, build arguments, logs, or artifacts.
2. **Prove changes in CI.** Run formatting/lint, typecheck, unit/integration tests, production build, and dependency/security checks appropriate to the project. Use ephemeral test secrets and databases. GitHub Actions should have minimal `GITHUB_TOKEN` permissions, pinned third-party actions, protected production environments where available, and secrets scoped to the job/environment that needs them. Do not put deployment secrets in repository files.
3. **Build once and identify the artifact.** Prefer a CI-built container image pushed to a registry and deploy an immutable commit SHA or digest. Deploy that same artifact to production; avoid rebuilding different source on the VPS. For a small private app without a registry, transfer a reviewed immutable archive and checksum. If the existing workflow builds on the VPS, explain the tradeoff and migrate only when the user authorizes it.
4. **Prepare the host.** Use a dedicated least-privilege deploy account with narrowly scoped Docker/deploy access. Prefer SSH keys, disable password SSH after confirming key access, verify host identity using a trusted fingerprint, restrict SSH exposure where practical, and enable a firewall without locking out current access. Keep OS packages maintained, time synchronized, and disk use monitored. Do not default to root SSH deployment.
5. **Configure runtime.** Keep application configuration separate from images. Use protected environment files or Compose secrets with restrictive permissions; document variable names and rotation. Bind app/database ports to loopback or private networks; expose only the reverse proxy. Use health checks, restart policies, resource limits appropriate to the host, persistent volumes, log rotation, and explicit container networks. A database should not be publicly reachable unless specifically required and protected.
6. **Connect DNS, proxy, and HTTPS.** Verify A/AAAA records point to the intended host and that inbound 80/443 are available. Configure the selected reverse proxy with the correct upstream and forwarded headers. Issue HTTPS certificates only after DNS and reachability are correct; verify automatic renewal and reload hooks. Test redirects, certificate chain/hostname, and application origin/cookie settings.
7. **Deploy safely.** Take or verify a recent recoverable backup before schema or data changes. Apply backward-compatible migrations before or during rollout; avoid destructive schema changes in the same release that still needs old code. Pull the exact artifact, start services, wait for health checks, inspect logs, and run authenticated and unauthenticated smoke tests. Verify externally over HTTPS. Do not claim success based only on a successful SSH command or CI job.
8. **Operate and recover.** Document release identity, logs, health, disk, resource use, certificate renewal, backup destination/retention, restore procedure, rollback artifact, and incident contacts. Back up databases and user-uploaded files off-host, encrypt where appropriate, and rehearse restoration in an isolated scratch target. Define recovery point/time expectations with the owner. A backup is not proven until restored.
9. **Update or roll back.** Repeat CI for every release. Keep the previous known-good image and migration compatibility window. Roll back app code only when database changes remain compatible; otherwise use a rehearsed forward fix or tested data restore. Record the deployed commit/digest and outcome.
10. **Remove an application safely.** Map domains, proxy routes, containers, compose projects, networks, volumes, host paths, database schemas, and backup needs. Identify shared resources and preserve them. Export a final backup if requested/appropriate, disable traffic, remove only app-owned resources, then verify unrelated apps and proxy routes remain healthy. State exactly what data was deleted.

## Verification and reporting

Use the project’s requested checks; do not add or run tests without user request where that constraint applies. At minimum, report available results for build/CI, container health, service logs, database/migration status, local health endpoint, public HTTPS response, certificate, and a representative user flow. Distinguish checks actually performed from recommended but unperformed checks.

Return a concise report in this order:

1. **Findings** — app/host facts and unknowns.
2. **Plan or changes** — exact staged work and artifact/release identity.
3. **Verification** — commands/results and live URL if verified.
4. **Recovery** — backup state, rollback target, restore rehearsal status.
5. **Open risks** — only concrete remaining issues and the action needed.

Never expose private keys, passwords, tokens, `.env` values, full secret-bearing logs, or sensitive host identifiers in the report. Redact them in generated files and shell output.

## Reference guides

Read only the relevant guide before detailed work:

- `references/vps-preflight.md` — host inventory, access, isolation, and compromise handling.
- `references/github-actions.md` — CI, artifact promotion, SSH deploy patterns, and workflow security.
- `references/production-runtime.md` — Docker Compose, proxy/TLS, health, migrations, and runtime configuration.
- `references/operations.md` — backups, restore, update, rollback, monitoring, incident response, and removal.
- `references/gatepass-case-study.md` — anonymized lessons from the KSP GatePass deployment, including incomplete launch checks.

Official references to re-check as relevant: [GitHub deployment environments](https://docs.github.com/en/actions/concepts/workflows-and-actions/deployment-environments), [GitHub Actions secure use](https://docs.github.com/en/actions/reference/security/secure-use), [Docker Compose production](https://docs.docker.com/compose/how-tos/production/), [Docker Compose secrets](https://docs.docker.com/compose/how-tos/use-secrets), [Certbot usage and renewal](https://eff-certbot.readthedocs.io/en/stable/using.html), and [NIST SSDF](https://csrc.nist.gov/projects/ssdf). Recheck official sources for current details rather than treating this list as immutable.
