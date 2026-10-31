# DevOps VPS Deploy skill

A reusable agent skill for preparing, deploying, securing, updating, troubleshooting, backing up, restoring, and safely removing web applications on self-managed Linux VPSs.

It emphasizes Docker Compose, GitHub Actions, immutable releases, least-privilege access, HTTPS, database migration safety, off-host backups, restore rehearsals, rollback, and shared-resource protection.

## Install with `npx`

Install globally for a supported agent:

```powershell
npx skills add Qaphael/devops-vps-deploy --global
```

Or install into the current project so the skill can be shared with that project:

```powershell
npx skills add Qaphael/devops-vps-deploy
```

The installer can prompt you to choose an agent. To select one explicitly, add its agent identifier, for example:

```powershell
npx skills add Qaphael/devops-vps-deploy --global --agent codex
```

Update installed skills with:

```powershell
npx skills update devops-vps-deploy
```

## Contents

- `SKILL.md` — workflow, decision points, and reporting format.
- `references/` — detailed host, CI, runtime, operations, and GatePass case-study guides.
- `evals/` — realistic scenarios and expected behaviors used to review the skill.

The GatePass case study is anonymized and records both successful deployment steps and launch gaps. Re-check the target repository and VPS before applying any case-study detail to another environment.

## Scope

This repository distributes an agent skill through the `skills` CLI. It is not an npm library or a VPS deployment script. The skill guides an agent through an application's deployment workflow and requires inspection before changes to shared infrastructure.
