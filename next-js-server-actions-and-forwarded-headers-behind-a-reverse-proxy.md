# Next.js server actions and forwarded headers behind a reverse proxy

Next.js works behind a reverse proxy without much effort, but a few behaviours depend on the headers the proxy sends. When they are wrong, you see server actions rejected, redirects to `http://` or `localhost`, and every request logged from the proxy's IP.

## Server actions and origin checks

Next.js compares the `Origin` header of a server action request with the host the request was sent to. If your proxy rewrites the `Host` header, or the app is reached through more than one domain, the check can fail. Two fixes:

1. Make the proxy pass the original host. Caddy and Traefik do by default. In nginx, set `proxy_set_header Host $host;` and `proxy_set_header X-Forwarded-Host $host;`.
2. If you legitimately serve actions from other origins, list them in `next.config.js`:

```
module.exports = {
  experimental: {
    serverActions: { allowedOrigins: ['app.example.com', 'www.example.com'] },
  },
}
```

Check the docs for your Next.js version, since this option has moved between releases.

## Redirects that go to http or localhost

When the proxy terminates TLS and talks to Next.js over plain HTTP, absolute URLs built from the request can use the wrong scheme or host. Make sure the proxy sends `X-Forwarded-Proto` and `X-Forwarded-Host`, and set any auth library's base URL setting explicitly, for example `NEXTAUTH_URL` or `AUTH_URL` for Auth.js, to the public HTTPS address.

## Real client IPs

Behind a proxy, the socket address is the proxy. Read the client IP from `X-Forwarded-For` in middleware or route handlers, and only trust it when the request came from your proxy. Never expose the Next.js port publicly, or anyone can spoof the header.

## Bind address

In standalone mode, start the server with `HOSTNAME=0.0.0.0` inside the container. Otherwise it may bind to the container's hostname interface and the proxy cannot reach it, which shows up as a 502.

## Streaming and buffering

Streaming responses and server sent events need buffering turned off in nginx: `proxy_buffering off;` for those locations. Caddy streams by default.

## A quick test

```
curl -sI https://app.example.com/ | grep -i -E 'location|x-powered'
```

No `localhost` in any `Location` header means redirects are healthy.

The rest of the setup, from building the image to TLS, is in this guide to [deploying Next.js to a VPS](https://peon.sh/blogs/deploy-nextjs-to-vps).

Disclosure: I help build [Peon](https://peon.sh/), a self-hosted Vercel alternative.
