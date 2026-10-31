# Production runtime: Compose, proxy, TLS, and migrations

## Container image

Use a multi-stage build to keep compilers and development dependencies out of the runtime image. Run as a non-root UID where supported. Include only required runtime files and production dependencies. Set a clear entrypoint, handle SIGTERM, and do not bake secrets into layers or build args. Add `.dockerignore` for VCS metadata, local dependencies, build output, and secret files.

## Compose service design

- Pin image versions or digests. Use explicit service names and networks.
- Add health checks that verify readiness, not merely that a process exists. Make app startup wait for database health when needed, while keeping application retry behavior robust.
- Set restart policies and reasonable CPU/memory limits based on observed host capacity. Configure log rotation to prevent disk exhaustion.
- Persist database and user-upload data in named volumes or deliberate host paths; document backup and restore. Do not persist ephemeral caches without a reason.
- Publish only the reverse proxy’s ports publicly. Bind app ports to loopback when proxy runs on host, or keep the app private on a Docker network. Do not expose PostgreSQL publicly by default.
- Use Docker Compose secrets where supported and appropriate. For environment files, enforce ownership/mode and keep them outside source control and image build context.

## Reverse proxy and certificates

Maintain per-site proxy configuration so one app’s update cannot silently replace other sites. Validate proxy syntax before reload. Confirm the upstream is healthy, preserve correct Host and forwarded protocol headers, configure body size/timeouts for the app, and check websocket support if needed. Verify public DNS and firewall reachability before certificate issuance. Test certificate hostname/chain and automatic renewal; execute a renewal dry run where supported and make sure the proxy reload hook works.

## Database changes

Review migration SQL for locks, table rewrites, backfills, and destructive operations. Prefer expand-migrate-contract: add compatible schema first, deploy code that supports old and new forms, backfill safely, then remove old schema in a later release. Take a verified backup first. Never assume restoring an old database while newer code is running is safe. Test migrations against a production-like copy where feasible.

## Environment and health

Maintain `.env.example` with names, safe non-secret examples, and descriptions. Validate required production variables at startup and avoid echoing values. Separate liveness from readiness when useful. Health checks should not expose internal secrets or detailed system state publicly. Check proxy-to-app health and external HTTPS independently.
