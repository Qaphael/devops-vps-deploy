# VPS operations and recovery

## Backups

Identify every stateful item: database, uploads, user-generated files, configuration, certificates only where appropriate, and required encryption keys. Define owner-approved retention and recovery objectives. Automate encrypted off-host copies with restricted credentials. Keep backup credentials separate from application credentials. Alert on failure and age. Document a restore command and perform a periodic isolated restore rehearsal; verify row counts and representative app behavior. Do not call a backup usable until restore succeeds.

## Release and rollback

Record commit, image digest, migration version, deployment time, and health status. Retain a known-good artifact. For a failed release, inspect current remote state first, then select a compatible prior artifact and verify health. Do not blindly rerun a partially completed deployment. Database rollback is a separate, riskier operation; favor compatible forward fixes unless the restore plan is rehearsed and approved.

## Monitoring and incidents

At minimum observe external HTTPS availability, container health/restarts, host disk and memory, database availability, certificate expiry/renewal, backup age/result, and application errors. Ensure logs have rotation and are not secret-bearing. For incidents, establish impact and timeline, preserve evidence, contain the issue, restore service from a trusted state, rotate affected secrets, and record follow-up work. If compromise is suspected, use the host-compromise procedure in `vps-preflight.md`.

## Removal checklist

1. Confirm hostname(s), owner, requested scope, and whether data should be retained or exported.
2. Inventory proxy routes, compose project, containers, images, volumes, mounts, networks, database/schema, cron jobs/systemd units, backups, and external dependencies.
3. Mark shared resources; leave them in place unless separately authorized and ownership is established.
4. Back up requested/valuable data and verify archive readability.
5. Disable only the target route and stop only target services; verify neighboring apps.
6. Remove app-owned resources by explicit names/paths. Avoid broad prune commands.
7. Verify DNS/proxy no longer routes to the removed app, other apps remain healthy, and backups are retained according to the agreed policy.
8. Report exactly which containers, files, volumes, schemas, and backups were retained or removed.
