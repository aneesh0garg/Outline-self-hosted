# Run guide

## Start the stack

Start the identity provider first, then Outline:

```sh
(cd keycloak-outline && docker compose up -d)
docker compose up -d
```

Open Outline at `https://outline.localhost:9443`.

## Check status

```sh
docker compose ps
(cd keycloak-outline && docker compose ps)
```

Expected services:

- `outline`, `outline-postgres`, and `outline-redis` become healthy.
- `outline-caddy` serves the local HTTPS endpoint.
- `keycloak-outline` listens on port `5001`.

## View logs

```sh
docker compose logs -f outline
docker compose logs -f caddy
(cd keycloak-outline && docker compose logs -f keycloak)
```

To inspect all application services without following them:

```sh
docker compose logs --tail=100
```

## Stop and restart

Stop containers while retaining databases, uploads, and Keycloak data:

```sh
docker compose down
(cd keycloak-outline && docker compose down)
```

Start them again with the commands in [Start the stack](#start-the-stack).

## Update container images

Pull and recreate one Compose project at a time:

```sh
(cd keycloak-outline && docker compose pull && docker compose up -d)
docker compose pull
docker compose up -d
```

Check the logs after updating. Major Outline, PostgreSQL, Redis, and Keycloak upgrades can require version-specific migration work; review the upstream release notes before updating a non-disposable instance.

## Troubleshooting

### Outline restarts while fetching OIDC configuration

Confirm Keycloak is running and that the host entry exists:

```sh
(cd keycloak-outline && docker compose ps)
curl http://keycloak.local:5001/realms/outline/.well-known/openid-configuration
```

If the request fails, add `127.0.0.1 keycloak.local` to the host machine's hosts file, then restart Outline:

```sh
docker compose restart outline
```

### Browser rejects the HTTPS certificate

Trust the Caddy local root certificate as described in the [setup guide](SETUP.md#4-trust-the-local-https-certificate). For connection diagnostics only:

```sh
curl -k https://outline.localhost:9443
```

### Sign-in returns to Outline with an error

In Keycloak, verify that the `outline` client has this exact redirect URI:

```text
https://outline.localhost:9443/auth/oidc.callback
```

Also confirm the client secret in `docker.env` matches the current Keycloak client secret. Restart Outline after changing `docker.env`:

```sh
docker compose up -d --force-recreate outline
```

### Cloudflare Tunnel is connected but sign-in fails

When publishing through Cloudflare Tunnel, both Outline and Keycloak need public hostnames. Follow [CLOUDFLARE_TUNNEL.md](CLOUDFLARE_TUNNEL.md) and confirm that the Keycloak issuer, Keycloak hostname, and client redirect URI all use `https://auth.pi-coding.com` and `https://outline.pi-coding.com`.

### Reset local data

To reset all persistent local data, stop the projects and remove their named volumes. This permanently deletes Outline content, PostgreSQL data, Redis data, and Keycloak configuration.

```sh
docker compose down -v
(cd keycloak-outline && docker compose down -v)
```

Then repeat the [setup guide](SETUP.md).
