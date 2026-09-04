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
expected by the GitLab chart, an `s3cmd` configuration file for the backup utility, and distinct, pre-created bucket
names for artifacts, LFS objects, uploads, packages, backups, and temporary restore data.

The initial administrator username is `root`. Wodby generates its password as the `root_password` service token and
stores it in the service environment Secret consumed by the GitLab migrations job.

## Deliberately external or disabled capabilities

- GitLab Runner is not installed. Register a separate Runner to execute CI/CD jobs; GitLab recommends separate runners
  for production deployments.
- Container Registry, GitLab Pages, the Kubernetes Agent Server, incoming email, and the bundled monitoring stack are
  disabled.

The bundled Gitaly workload is a single repository-storage instance. Production or high-availability installations
should follow GitLab's
[Cloud Native Hybrid guidance](https://docs.gitlab.com/administration/reference_architectures/cloud_native/).

## Backups and restore

The chart runs GitLab's Toolbox `backup-utility` daily at 01:00 in the Kubernetes controller's time zone. Each backup
includes the GitLab database, Gitaly repositories and wikis, and the configured GitLab object-storage data, then
uploads the archive to `GITLAB_BACKUPS_BUCKET`. Overlapping runs are forbidden and each run has a six-hour deadline.
These archives are managed directly in that bucket and do not appear as Wodby backup records.

Backup assembly uses a dynamically provisioned, 100 GiB generic ephemeral volume. Ensure the cluster has
enough provisionable storage for the uncompressed backup working set; increase this value before stored data approaches
that size. Configure retention and independent replication on the backup bucket.

The PostgreSQL service's own backup remains useful as an additional database recovery layer, but it does not replace
the GitLab archive because it contains neither repository data nor the matching GitLab object data.

GitLab deliberately excludes Rails secrets from its backup archive. Export the chart's Rails secret to a separate,
access-restricted location and keep it with the recovery plan. Restores must use the same GitLab version, restore those
secrets, and stop webservice and Sidekiq clients while `backup-utility --restore` runs. Follow GitLab's
[Helm restore procedure](https://docs.gitlab.com/charts/backup-restore/restore/); this service does not expose an unsafe
one-click restore action.

## Upgrades

GitLab upgrades must follow the upstream
[required upgrade stops](https://docs.gitlab.com/update/upgrade_paths/). Complete background migrations and create a
tested backup before advancing to the next required stop.
