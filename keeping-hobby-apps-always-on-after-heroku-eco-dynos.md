# Keeping hobby apps always on after Heroku Eco dynos

Heroku's Eco dynos sleep after 30 minutes without web traffic, and the first request after that waits while the dyno wakes. For side projects and internal tools this is the main annoyance, and it is often the reason people look at moving. A small VPS runs the same apps always on, usually for a similar or lower total cost once you have more than a few apps.

## How many hobby apps fit on one server

A typical hobby web app idles at 80 to 250 MB of memory: Node and Go at the low end, Rails and Django with a couple of workers at the high end. A 4 GB server, after the OS, Docker and a reverse proxy take roughly 600 MB, comfortably runs 10 to 15 such apps with a shared Postgres instance. Measure your own with `docker stats --no-stream` after a day of normal use.

## Translating the pieces

- **Web dyno:** a container running the same command as the `web:` line in the Procfile.
- **Worker dyno:** a second container from the same image with the worker command.
- **Heroku Postgres Essential:** one Postgres container shared by the apps, with a separate database and user per app.
- **Heroku Scheduler:** a cron container or host cron running one-off containers.
- **Config vars:** environment variables per app.

## Set limits so one app cannot starve the rest

Always-on also means always able to misbehave. Give each app a memory limit slightly above its normal peak:

```
services:
  web:
    image: ghcr.io/me/notes-app:1.4.2
    deploy:
      resources:
        limits:
          memory: 300M
    restart: unless-stopped
```

A memory leak then restarts one container instead of pushing the whole server into swap.

## Things Heroku did that you now own

Security updates for the OS, backups for the database, and TLS certificates. Automate all three on day one: unattended upgrades, a nightly `pg_dump` copied offsite, and a reverse proxy with automatic HTTPS.

## Is it worth it for one app?

For a single app that gets a visitor a week, sleeping may be fine. The VPS approach pays off from two or three apps upward, or when the cold start delay matters to you.

For a wider look at where to move from Heroku, see this roundup of [Heroku alternatives in 2026](https://peon.sh/blogs/heroku-alternative-2026).

Disclosure: I help build [Peon](https://peon.sh/), an open-source Heroku-style platform for your own servers.
