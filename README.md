# Hosting decision notes

Short, practical notes on choosing regions, databases, update policies and replacements for managed platform features when you run apps on your own servers.

- [Hetzner or DigitalOcean for users in India and Southeast Asia](hetzner-or-digitalocean-for-users-in-india-and-southeast-asia.md): Region choice, latency and data transfer considerations when your users are in South and Southeast Asia.
- [Keeping hobby apps always on after Heroku Eco dynos](keeping-hobby-apps-always-on-after-heroku-eco-dynos.md): How to replace sleeping Eco dynos with always-on containers on a small VPS, and how many hobby apps fit on one server.
- [Next.js server actions and forwarded headers behind a reverse proxy](next-js-server-actions-and-forwarded-headers-behind-a-reverse-proxy.md): Fixing origin checks, client IPs and redirect URLs when a self-hosted Next.js app runs behind Caddy, Traefik or nginx.
- [Replacing Vercel Analytics and Speed Insights when you self-host](replacing-vercel-analytics-and-speed-insights-when-you-self-host.md): Self-hosted options for page views and Core Web Vitals after moving a site off Vercel, and what each one measures.
- [Keeping Dockerfile base images patched with Dependabot or Renovate](keeping-dockerfile-base-images-patched-with-dependabot-or-renovate.md): Automating pull requests for base image updates, with a PHP example, so security fixes reach production without manual checks.
- [One Postgres per app or a shared cluster for many small apps](one-postgres-per-app-or-a-shared-cluster-for-many-small-apps.md): Trade-offs between running a database container per app and one shared Postgres instance with a database per app.

These notes are maintained by people who work on [Peon](https://peon.sh/), an open-source self-hosted deployment platform. Corrections are welcome through issues.
