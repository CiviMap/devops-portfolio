# CiviMap platform engineering

CiviMap runs as a containerized web platform on K3s. I built the deployment
setup around a simple rule: application pipelines produce artifacts, while Git
and Argo CD control what runs in each environment.

## Platform responsibilities

- provision the host and K3s with Ansible;
- validate application, infrastructure, and Helm changes in GitHub Actions;
- publish commit-addressed container images to GHCR;
- manage Kubernetes resources and environment values with Helm;
- reconcile staging and production through Argo CD;
- monitor availability and resource health with Prometheus and Alertmanager;
- automate database and file backups and verify the recovery procedure.

## Architecture

The runtime and delivery flows are documented in
[docs/architecture.md](docs/architecture.md).

The application is represented as a single workload boundary in the diagrams.
The focus here is the deployment platform: edge routing, cluster reconciliation,
persistence, monitoring, backups, and promotion between environments.

The same document includes the planned platform evolution: shared background
processing, selectively deployable cloud resources, and a security-focused
software supply chain. Planned components are kept separate from the current
architecture so their implementation status remains clear.

## Release flow

1. A source change runs repository-specific tests and container builds.
2. Successful builds publish immutable images tagged from the commit.
3. A GitOps change updates the staging image references.
4. Helm rendering and the Argo CD diff are reviewed before synchronization.
5. The staging release is checked through readiness probes and monitoring.
6. The same image references are promoted to production through a separate
   reviewed change and manual synchronization.

Application CI does not hold cluster credentials. It stops after publishing the
artifact; Argo CD is responsible for applying the desired state.

## Operations

- Prometheus collects application and platform health signals.
- Alertmanager routes actionable alerts to the operator.
- Kubernetes readiness checks and Argo CD health are part of release verification.
- Scheduled workflows back up the database and persistent files.
- Restore checks use an isolated database before a backup is accepted as usable.
- Application rollback is a Git revert to previously verified image references.

Database recovery is treated separately from application rollback. Migrations
are kept backward-compatible where possible, with forward fixes preferred over
reversing production data changes.

## Tooling

GitHub Actions · Docker · GHCR · Ansible · K3s · Kubernetes · Helm · Argo CD ·
Prometheus · Alertmanager · Cloudflare · PostgreSQL/PostGIS · Bash · PowerShell
