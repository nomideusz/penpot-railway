# Deploy and Host Penpot on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/penpot-production?utm_medium=integration&utm_source=button&utm_campaign=penpot-production)

[Penpot](https://penpot.app/) is the open-source design and prototyping platform, the self-hostable alternative to Figma: vector design, components and variants, design tokens, flex and grid layout, interactive prototypes and real-time multiplayer editing, with an inspect mode that hands developers CSS, SVG and HTML. Files use open standards (SVG, CSS) and you can import your Figma work. This template runs the official Penpot images, pinned, with Postgres, Valkey and a Railway bucket for every uploaded image, font and thumbnail.

## About Hosting Penpot

The stack is five services and a bucket: Penpot (the web app), Backend, Exporter, Postgres, Valkey, and the Assets bucket.

- **Upstream's own images, pinned.** `penpotapp/frontend`, `penpotapp/backend` and `penpotapp/exporter` at a fixed release (2.18.3), not the moving `latest` tag. Upgrading is bumping the three tags together.
- **Assets in a Railway bucket.** Upstream's compose file shares one disk between the web server and the backend; Railway services cannot share a volume, so uploads go to S3 instead. The backend stores objects in the bucket and answers each request with a short-lived signed URL, which the Penpot web server fetches and streams back. Browsers only ever talk to your Penpot domain.
- **Migrations on every boot.** The backend migrates the database at start-up and Railway's health check on `/readyz` only passes once it is connected to Postgres and Valkey.
- **Survives redeploys of any service.** The web server proxies `/api`, `/assets` and websockets to the backend and exports to the exporter over the private network, re-resolving their addresses while it runs, so redeploying the backend never leaves the web app pointing at a dead address, and the web app can start before the others.
- **Exports work out of the box.** The Exporter runs a headless browser that renders PNG, JPEG, SVG and PDF exports from the web app over the private network.
- **Generated secrets.** `PENPOT_SECRET_KEY` (sessions, invitations, the exporter's key), the database password and the Valkey password are generated for each deployment. Upstream's compose file ships a publicly known `change-this-insecure-key`.

## Common Use Cases

- A self-hosted Figma replacement for a product or agency design team
- Design systems with shared libraries, components and design tokens
- Clickable prototypes to share with clients and stakeholders by link
- Design-to-code handoff: developers inspect CSS, SVG and measurements in the browser

## Dependencies for Penpot Hosting

- Postgres 17 (included, private network only)
- Valkey 8, Redis-compatible (included, private network only, used for real-time notifications)
- A Railway Storage Bucket (included) for uploaded media, fonts and thumbnails

### Deployment Dependencies

- [Penpot self-hosting documentation](https://help.penpot.app/technical-guide/getting-started/)
- [Penpot configuration reference](https://help.penpot.app/technical-guide/configuration/)
- [Penpot on GitHub](https://github.com/penpot/penpot)
- [Template source on GitHub](https://github.com/nomideusz/penpot-railway)

### Implementation Details

**Sign up, then close registration.** Open the Penpot service's Railway domain and create your account; email verification is off (`disable-email-verification`) because Railway only allows outbound SMTP on the Pro plan. Registration is open to anyone who finds the URL, so once your team has signed up add `disable-registration` to `PENPOT_FLAGS` on the **Backend** service (the web app and exporter read the same value) and redeploy all three. Teammates can still join through invitations. Penpot has no instance-wide admin: every user owns their own teams and projects.

**One place to configure.** `PENPOT_FLAGS` and `PENPOT_PUBLIC_URI` live on the Backend service; the Penpot and Exporter services reference them. See the [list of flags](https://help.penpot.app/technical-guide/configuration/#penpot-configuration) for login with Google, GitHub, GitLab, OIDC or LDAP.

**Custom domain.** Add it to the Penpot service, then set `PENPOT_PUBLIC_URI` on Backend to `https://your-domain`.

**Email (optional).** For invitations by email, password resets and verification, add `PENPOT_SMTP_HOST`, `PENPOT_SMTP_PORT`, `PENPOT_SMTP_USERNAME`, `PENPOT_SMTP_PASSWORD`, `PENPOT_SMTP_TLS` (or `PENPOT_SMTP_SSL`) and `PENPOT_SMTP_DEFAULT_FROM` to Backend, and add `enable-smtp` to `PENPOT_FLAGS`. Without SMTP you can still invite people to a team and share the invitation link.

**Back up.** Postgres holds every design file; the bucket holds images, fonts and thumbnails. Losing `PENPOT_SECRET_KEY` only logs everyone out.

**Not included.** Upstream's optional admin console and MCP server (AI-assistant integration) are not part of this template, keeping it lean.

**Resources.** In testing the backend (a JVM, heap capped at 1 GB by `JAVA_OPTS` on Backend) idled at 0.9–1.1 GB, the exporter at 140–430 MB (its headless browser stays warm after an export), the web server around 60 MB, Postgres around 125 MB and Valkey under 10 MB: roughly 1.3–1.8 GB for the stack. That needs the Hobby plan; it does not fit the Trial plan's 512 MB per service.

## Why Deploy Penpot on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Penpot on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
