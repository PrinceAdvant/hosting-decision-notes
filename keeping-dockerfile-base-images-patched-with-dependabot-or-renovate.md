# Keeping Dockerfile base images patched with Dependabot or Renovate

Most container vulnerabilities are fixed upstream long before they are fixed in your image. A PHP image built six months ago still runs the PHP patch release and OS packages from six months ago. Automated update pull requests close that gap with very little effort.

## Pin precisely, then let a bot move the pin

`FROM php:8.3-fpm` floats, which sounds convenient but makes builds unreproducible and hides when an update happened. Pin a patch version, and optionally the digest:

```
FROM php:8.3.13-fpm-bookworm@sha256:<digest>
```

A bot then opens a pull request when a newer patch or digest appears, your CI builds and tests it, and you merge.

## Dependabot

Add `.github/dependabot.yml`:

```
version: 2
updates:
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
```

Dependabot reads `FROM` lines and opens pull requests for newer tags. Set the `directory` for each Dockerfile location.

## Renovate

Renovate handles Dockerfiles, Compose files and digests, and can group updates. A minimal `renovate.json`:

```
{
  "extends": ["config:recommended"],
  "packageRules": [
    { "matchDatasources": ["docker"], "matchUpdateTypes": ["patch", "digest"], "automerge": true }
  ]
}
```

Auto merging patch and digest updates, while keeping minor and major updates for review, is a sensible default if your tests are decent.

## Rebuild even when nothing changed

A pinned tag can be rebuilt upstream with patched OS packages. Digest pinning catches this. If you do not pin digests, schedule a weekly rebuild with `--pull --no-cache` so you pick up fixed packages anyway.

## PHP specifics

- Extensions compiled with `docker-php-ext-install` are rebuilt with the image, so a base update needs a full rebuild, not just a restart.
- Composer dependencies are separate. Add the `composer` ecosystem to Dependabot or let Renovate handle `composer.json`.
- Track the PHP branch support dates. A branch that only receives security fixes is fine; a branch past end of life gets nothing.

## Scan what you ship

Run a scanner such as Trivy or Grype in CI against the built image and fail on high severity issues that have a fix available.

For the current PHP vulnerability picture and what to patch first, see this overview of [PHP vulnerabilities in 2026](https://peon.sh/blogs/php-vulnerabilities-2026).

Disclosure: I help build [Peon](https://peon.sh/), a self-hosted platform for deploying containerized apps.
