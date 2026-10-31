# GitHub CI and VPS delivery

## CI checks

Use a lockfile and a pinned runtime/toolchain where possible. A typical application pipeline runs install, lint/format, typecheck, unit/integration tests, production build, container build, and image scan. Use ephemeral databases and generated test-only secrets; never reuse production credentials in CI. Publish test reports and deployment identity, not secrets.

Keep workflow permissions minimal (`contents: read` by default, add only what a job needs). Pin third-party actions to reviewed full commit SHAs and update them deliberately. Avoid printing contexts containing secrets. Use protected GitHub Environments for production credentials and require review where the project’s risk warrants it. Scope secrets to the smallest environment/job. Re-check GitHub’s current secure-use guidance.

## Artifact promotion

Build an immutable image once in CI and tag it with the commit SHA; deploy by digest when possible. Promote the same artifact from validation to production. Do not rely on `latest` for rollback identity. For a private app where registry publishing is unavailable, transfer an immutable archive plus checksum/signature over authenticated SSH, then verify it on the host.

## SSH deployment

- Use a dedicated deploy identity; avoid root login and broad shell access if a safer mechanism is feasible.
- Pin the VPS host key from a fingerprint obtained through a trusted channel. Do not blindly trust a `ssh-keyscan` result gathered over the same unverified connection.
- Keep keys in protected environment secrets; never write them to the repository or image.
- Use strict host key checking, bounded connect timeouts, keepalives for long transfers, and explicit remote command status handling.
- Stream or transfer artifacts to a versioned release directory, verify checksum, activate the selected release, and preserve the previous known-good version.
- Make retries idempotent and report which commit/digest is active. A dropped SSH connection does not prove the remote command stopped; reconnect and inspect state before retrying.

## Workflow shape

Separate CI from production deployment. Gate deployment on the intended branch/tag and successful required checks. Use a protected production environment. The deploy job should identify the artifact, establish verified SSH, deploy, wait for health, run smoke checks, and fail visibly if the service is unhealthy. Avoid concurrent deploy races using a workflow concurrency group. Keep rollback as a documented job or command that selects a known prior artifact; do not automatically roll back database state.

Consult GitHub docs for current workflow/environment syntax and action pinning behavior before editing YAML.
