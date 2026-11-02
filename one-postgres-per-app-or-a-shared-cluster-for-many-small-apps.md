# One Postgres per app or a shared cluster for many small apps

When a server runs ten small apps, each one needs a database. There are two common layouts: a Postgres container per app, or one Postgres instance holding a database per app. Both work. The right choice depends on memory, isolation and how you upgrade.

## One container per app

**Pros:** full isolation. Each app can use a different Postgres major version, extensions and settings. Upgrading or restoring one app touches nothing else. Removing an app removes its database cleanly.

**Cons:** each instance has its own shared buffers, background processes and connection overhead. Ten instances idling can use well over a gigabyte between them. You also run ten backup jobs and watch ten sets of logs.

## One shared instance

**Pros:** memory is shared, so one instance with sensible settings serves all apps with less total memory. One backup job, one upgrade, one place to tune.

**Cons:** a noisy app can slow the others, since they share CPU, IO and connection slots. A major version upgrade affects every app at once. Mistakes with permissions can let one app read another's data.

## If you share, isolate properly

```
CREATE ROLE notes LOGIN PASSWORD '...';
CREATE DATABASE notes OWNER notes;
REVOKE CONNECT ON DATABASE notes FROM PUBLIC;
```

Repeat per app. Revoking `CONNECT` from `PUBLIC` stops other app roles from even connecting. Set per role limits too:

```
ALTER ROLE notes CONNECTION LIMIT 20;
ALTER ROLE notes SET statement_timeout = '30s';
```

## A middle ground

Share one instance for small, low traffic apps that can tolerate the same upgrade schedule. Give a dedicated instance to anything with heavy traffic, special extensions, or stricter data separation requirements.

## Backups either way

With a shared instance, back up each database separately with `pg_dump -Fc dbname`, not only `pg_dumpall`. Restoring one app's data from a single combined dump is slow and error prone.

## Upgrades

Per app instances let you upgrade one at a time and roll back if needed. With a shared instance, test the upgrade on a copy first, then schedule a window for all apps.

Peon's documentation on [databases in Peon](https://peon.sh/docs/databases) shows how managed database services are created and attached to apps.

Disclosure: I help build [Peon](https://peon.sh/), a self-hosted PaaS.
