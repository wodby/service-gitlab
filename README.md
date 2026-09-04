# GitLab Community Edition service

Wodby 2 service manifest for GitLab Community Edition, based on the official GitLab Helm chart.

The service expects PostgreSQL, a Redis-compatible datastore, and an S3-compatible object-storage integration. The
object-storage integration must provide the connection YAML expected by the GitLab chart plus distinct bucket names
for artifacts, LFS objects, uploads, packages, and backups.
