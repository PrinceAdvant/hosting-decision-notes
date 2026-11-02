# Replacing Vercel Analytics and Speed Insights when you self-host

Moving a Next.js or static site off Vercel is mostly about builds and hosting, but teams often forget two small integrations: Vercel Web Analytics for page views and Speed Insights for real user performance data. Both stop working once the site is served from your own server. Here is how to replace each.

## Page views: privacy friendly analytics

Self-hosted Plausible and Umami are the usual replacements. Both count page views, referrers, countries and devices without cookies, which is close to what Vercel Web Analytics shows.

- **Umami** runs on Postgres or MySQL and is light. A small app and database fit easily next to your other containers.
- **Plausible Community Edition** uses Postgres plus ClickHouse. ClickHouse needs more memory, so plan for at least a couple of gigabytes for the analytics stack.

Swap the `@vercel/analytics` component for the tool's script tag in your root layout. If you track custom events, map each `track()` call to the equivalent call in the new tool.

## Real user performance: Web Vitals

Speed Insights reports Core Web Vitals from real visitors. To keep that data:

1. Use the `web-vitals` library, or Next.js's `useReportWebVitals` hook, to capture LCP, INP and CLS in the browser.
2. Send each metric with `navigator.sendBeacon` to a small endpoint you control.
3. Store them in Postgres or forward them as custom events to your analytics tool.

```
import { onLCP, onINP, onCLS } from 'web-vitals'
const send = (m) => navigator.sendBeacon('/api/vitals', JSON.stringify({ name: m.name, value: m.value, page: location.pathname }))
onLCP(send); onINP(send); onCLS(send)
```

Report the 75th percentile per page, which is how Google evaluates Core Web Vitals.

## Lab data still matters

Run Lighthouse in CI on key pages to catch regressions before they ship. Real user data tells you what happened; lab data tells you why.

## Ad blockers

Self-hosted scripts on your own domain are blocked less often than third-party ones, but some lists still block common analytics paths. Serving the script and endpoint under a first-party path improves coverage.

## Check your consent setup

Even cookieless tools may need a mention in your privacy policy. Review it with whoever handles compliance.

For the bigger picture of what replaces Vercel's hosting itself, see this guide to [self-hosted Vercel alternatives](https://peon.sh/blogs/vercel-alternative-self-hosted).

Disclosure: I help build [Peon](https://peon.sh/), an open-source platform for deploying Next.js and other apps to your own servers.
