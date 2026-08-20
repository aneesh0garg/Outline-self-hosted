# Outline — self-hosted local stack

A Docker Compose setup for running [Outline](https://www.getoutline.com/) locally with PostgreSQL, Redis, and Caddy HTTPS. Authentication is provided by the shared [common-keycloak-instance](../common-keycloak-instance) service.

This repository is intended for local development and evaluation. It is not a production-hardened deployment.

## What runs

| Service | Purpose | Local address |
| --- | --- | --- |
| Outline | Knowledge base application | `https://outline.localhost:9443` |
| Caddy | HTTPS reverse proxy | `http://localhost:9081`, `https://localhost:9443` |
| PostgreSQL | Outline database | Internal Docker network |
| Redis | Jobs, cache, and collaboration support | Internal Docker network |

## Quick start

1. Install Docker Desktop and ensure it is running.
2. Start the shared Keycloak instance, then follow the [setup guide](SETUP.md) to configure `docker.env`, the Keycloak client, local hostname resolution, and the local certificate.
3. Start Outline:

   ```sh
   (cd ../common-keycloak-instance && docker compose up -d)
   docker compose up -d
   ```

4. Open `https://outline.localhost:9443` and sign in through Keycloak.

For day-to-day commands, troubleshooting, and shutdown steps, see the [run guide](RUN.md).

To expose this local stack through `pi-coding.com` without a hosting bill, follow the [Cloudflare Tunnel guide](CLOUDFLARE_TUNNEL.md).

For the exact changes required when switching between localhost and Cloudflare, see [run modes](MODES.md).

## Repository layout

```text
.
├── docker-compose.yml              # Outline, PostgreSQL, Redis, and Caddy
├── Caddyfile                       # Local HTTPS reverse proxy configuration
├── docker.env.example              # Safe configuration template
├── postgres.env.example            # Safe PostgreSQL configuration template
└── docker-compose.cloudflare.example.yml # Optional public tunnel connector
```

## Security notes

- `docker.env` contains credentials and application secrets. It is deliberately ignored by Git; start from `docker.env.example`.
- The shared Keycloak administrator and database credentials live in `../common-keycloak-instance/.env`. Change its defaults before public use.
- Do not expose the default ports directly to the internet. Use a production-grade reverse proxy, TLS certificates, backups, SMTP, and managed secrets for a real deployment.

## License

Outline itself is distributed under its own license. See the upstream [Outline repository](https://github.com/outline/outline) for product source, licensing, and documentation.
