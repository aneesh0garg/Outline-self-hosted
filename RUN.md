# Run guide

For switching between localhost and Cloudflare Tunnel, use [MODES.md](MODES.md). Do not mix the hostname and OIDC settings from the two modes.

## Start the stack

Start the shared identity provider first, then Outline:

```sh
(cd ../common-keycloak-instance && docker compose up -d)
docker compose up -d
```

Open Outline at `https://outline.localhost:9443`.

## Start Cloudflare Tunnel

`RUN.md` does not create a Cloudflare Tunnel. Tunnel creation is a one-time action in the Cloudflare dashboard; see [CLOUDFLARE_TUNNEL.md](CLOUDFLARE_TUNNEL.md) for that setup.

After the tunnel has been created and its token has been saved in the untracked `docker-compose.cloudflare.yml` file, start its local connector with:

```sh
docker compose -f docker-compose.yml -f docker-compose.cloudflare.yml up -d cloudflared
```

Confirm that it connected:

```sh
docker compose -f docker-compose.yml -f docker-compose.cloudflare.yml logs --tail=40 cloudflared
```

Look for `Registered tunnel connection`. The Cloudflare dashboard should then report the tunnel as **Healthy**.

## Check status

```sh
docker compose ps
(cd ../common-keycloak-instance && docker compose ps)
```

Expected services:

- `outline`, `outline-postgres`, and `outline-redis` become healthy.
- `outline-caddy` serves the local HTTPS endpoint.
- `shared-keycloak` listens on port `5001`.

## View logs

```sh
docker compose logs -f outline
docker compose logs -f caddy
(cd ../common-keycloak-instance && docker compose logs -f keycloak)
```

To inspect all application services without following them:

```sh
docker compose logs --tail=100
```

## Stop and restart

Stop the Outline containers while retaining databases and uploads:

```sh
docker compose down
```

This command is safe: it stops and removes containers but keeps the named Docker volumes, including the databases and uploads. **Do not add `-v`** during an ordinary shutdown.

Stop the shared Keycloak stack separately only when no other project needs sign-in:

```sh
(cd ../common-keycloak-instance && docker compose down)
```

This is also safe for the shared Keycloak database. Never run `docker compose down -v` from `../common-keycloak-instance` unless you intentionally want to erase every realm, client, and user for every project that uses that Keycloak instance.

Start them again with the commands in [Start the stack](#start-the-stack).

## Update container images

Pull and recreate Outline:

```sh
docker compose pull
docker compose up -d
```

Update the shared Keycloak project separately and only after considering every connected application:

```sh
(cd ../common-keycloak-instance && docker compose pull && docker compose up -d)
```

Check the logs after updating. Major Outline, PostgreSQL, Redis, and Keycloak upgrades can require version-specific migration work; review the upstream release notes before updating a non-disposable instance.

## Troubleshooting

### Outline restarts while fetching OIDC configuration

Confirm Keycloak is running and that the host entry exists:

```sh
(cd ../common-keycloak-instance && docker compose ps)
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

### Reset local Outline data

To reset all persistent Outline data, stop the project and remove its named volumes. This is a destructive recovery command, not a normal shutdown. It permanently deletes Outline content, PostgreSQL data, and Redis data.

```sh
docker compose down -v
```

Then repeat the [setup guide](SETUP.md). Do not remove the shared Keycloak volumes unless you intend to reset authentication for every project using it.
