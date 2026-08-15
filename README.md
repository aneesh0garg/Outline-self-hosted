# Outline — self-hosted local stack

A Docker Compose setup for running [Outline](https://www.getoutline.com/) locally with PostgreSQL, Redis, Caddy HTTPS, and Keycloak for OpenID Connect (OIDC) sign-in.

This repository is intended for local development and evaluation. It is not a production-hardened deployment.

## What runs

| Service | Purpose | Local address |
| --- | --- | --- |
| Outline | Knowledge base application | `https://outline.localhost:9443` |
| Caddy | HTTPS reverse proxy | `http://localhost:9081`, `https://localhost:9443` |
| Keycloak | OIDC identity provider | `http://localhost:5001` |
| PostgreSQL | Outline database | Internal Docker network |
| Redis | Jobs, cache, and collaboration support | Internal Docker network |

## Quick start

1. Install Docker Desktop and ensure it is running.
2. Follow the [setup guide](SETUP.md) to configure `docker.env`, Keycloak, local hostname resolution, and the local certificate.
3. Start Keycloak, then the main stack:

   ```sh
   cd keycloak-outline
   docker compose up -d
   cd ..
   docker compose up -d
   ```

4. Open `https://outline.localhost:9443` and sign in through Keycloak.

For day-to-day commands, troubleshooting, and shutdown steps, see the [run guide](RUN.md).

## Repository layout

```text
.
├── docker-compose.yml              # Outline, PostgreSQL, Redis, and Caddy
├── Caddyfile                       # Local HTTPS reverse proxy configuration
├── docker.env.example              # Safe configuration template
├── postgres.env.example            # Safe PostgreSQL configuration template
└── keycloak-outline/
    └── docker-compose.yml          # Local Keycloak service
```

## Security notes

- `docker.env` contains credentials and application secrets. It is deliberately ignored by Git; start from `docker.env.example`.
- The Keycloak Compose file defaults to `admin` / `admin` for its bootstrap administrator. Change this before using the stack beyond a disposable local environment.
- Do not expose the default ports directly to the internet. Use a production-grade reverse proxy, TLS certificates, backups, SMTP, and managed secrets for a real deployment.

## License

Outline itself is distributed under its own license. See the upstream [Outline repository](https://github.com/outline/outline) for product source, licensing, and documentation.
