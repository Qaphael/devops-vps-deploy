# Anonymized case study: KSP GatePass

This case study records both what worked and what remained incomplete. It deliberately omits real hostnames, addresses, usernames, key fingerprints, and credentials. Treat it as a small single-VPS example, not a universal production blueprint.

## Application and path

The application is a Next.js/Prisma modular monolith with PostgreSQL. It was packaged with Docker Compose and served through Nginx with Certbot-managed HTTPS. DNS was configured to the VPS; the application port was bound to loopback, while the reverse proxy handled public traffic. GitHub Actions ran CI and gated deployment. The initial delivery streamed a Git archive over SSH to a host release directory and ran a Compose build/start on the VPS.

CI caught real issues before deployment: a missing explicit React `LayoutProps` type and tests failing because `.env.example` contained a placeholder token-encryption key. The type was corrected; CI generated an ephemeral valid test key instead of treating the example placeholder as a secret. The production image initially omitted runtime setup-admin files and `tsx`, which was corrected. A test was added for an event spanning midnight (18:00 to 06:00 next day).

The first long deploy passed CI but its SSH connection dropped during image export. The remote job’s state had to be checked before retrying. Keepalives improved reliability. The lasting improvement is to build once in CI, publish an immutable image, and deploy that exact digest; keepalives address a fragile connection but do not make host-side builds ideal.

## Host and workflow lessons

- The deployment used a root SSH key. Prefer a dedicated least-privilege deploy identity and verified host key for future deployments.
- Build time was unusually long while an unrelated database container consumed substantial CPU and memory. Inventory resource pressure and shared services before blaming application compilation.
- A separate app and database shared Docker networking with other sites. Cleanup therefore required mapping project labels, routes, networks, mounts, and data ownership. Preserve shared networks/databases unless ownership is clear.
- A deployment host had previously shown evidence of compromise, including an unexpected miner/persistence service. Stopping the service was not a clean rebuild. A trusted production posture still requires rebuilding from a known-good OS image and rotating credentials accessible from the host.
- A one-time production admin bootstrap was not confirmed complete. Do not assume a successful deploy means operational setup is complete.

## What was verified and what was open

At one point, GitHub Actions reported CI and deployment success and the public HTTPS site returned HTTP 200. That established availability at that time, not all launch readiness. Still-open checks included a target-host backup schedule and isolated restore rehearsal, real Android scanning from paper tickets over HTTPS, and completion/verification of the one-time admin bootstrap. The real-paper QR scan and backup-restore checks are launch gates for this ticketing application.

When reusing this case, re-inspect the present repository and host state. Do not copy old CI run IDs, deployment assumptions, IP addresses, domains, credentials, or claims of current health.
