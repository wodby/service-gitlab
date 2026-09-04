# GitLab Community Edition service

Wodby 2 service manifest for GitLab Community Edition, based on the official
[GitLab Helm chart](https://docs.gitlab.com/charts/).

## Runtime contract

The service provides GitLab's web application and API, Git over HTTPS and SSH, Sidekiq background jobs, a Toolbox
workload, and persistent Gitaly repository storage. It requires links to:

- PostgreSQL with the GitLab-required `amcheck`, `btree_gist`, and `pg_trgm` extensions
- a Redis-compatible datastore such as Valkey
- an SMTP relay

It also requires an S3-compatible object-storage integration. The integration must provide the connection YAML
expected by the GitLab chart and distinct, pre-created bucket names for artifacts, LFS objects, uploads, packages,
backups, and temporary restore data.

The initial administrator username is `root`. Wodby generates its password as the `root_password` service token and
stores it in the service environment Secret consumed by the GitLab migrations job.

## Deliberately external or disabled capabilities

- GitLab Runner is not installed. Register a separate Runner to execute CI/CD jobs; GitLab recommends separate runners
  for production deployments.
- Container Registry, GitLab Pages, the Kubernetes Agent Server, incoming email, and the bundled monitoring stack are
  disabled.
- The service does not define a complete GitLab backup or restore operation. GitLab backups must include the database,
  repositories, object-storage data, and a separately protected copy of the Rails secrets.

The bundled Gitaly workload is a single repository-storage instance. Production or high-availability installations
should follow GitLab's
[Cloud Native Hybrid guidance](https://docs.gitlab.com/administration/reference_architectures/cloud_native/).

## Upgrades

GitLab upgrades must follow the upstream
[required upgrade stops](https://docs.gitlab.com/update/upgrade_paths/). Complete background migrations and create a
tested backup before advancing to the next required stop.
