# Publish Outline with Cloudflare Tunnel

This guide makes the already-running local Docker stack reachable at two public hostnames:

- `outline.pi-coding.com` — Outline
- `auth.pi-coding.com` — Keycloak sign-in

Cloudflare Tunnel is free, but this computer must remain powered on, connected to the internet, and running Docker. It is suitable for a personal or small-team setup, not a highly available production deployment.

## 1. Move `pi-coding.com` DNS to Cloudflare

1. Create or sign in to a Cloudflare account.
2. In Cloudflare, choose **Add a site** and enter `pi-coding.com`.
3. At your domain registrar, replace the current nameservers with the two Cloudflare nameservers shown during onboarding.
4. Wait until Cloudflare shows the zone as **Active**.

Cloudflare Tunnel needs the domain zone in your Cloudflare account to create the published hostnames.

## 2. Create a remotely-managed tunnel

1. In the Cloudflare dashboard, open **Networking → Tunnels**.
2. Select **Create a tunnel** and name it `outline-home`.
3. Choose **Docker** as the connector type.
4. Copy the token from the displayed Docker command. It is the value after `--token`.

Do not commit or share this token.

## 3. Configure the local tunnel connector

From the repository root, create the untracked configuration file:

```sh
cp docker-compose.cloudflare.example.yml docker-compose.cloudflare.yml
```

Edit `docker-compose.cloudflare.yml` and replace:

```text
REPLACE_WITH_YOUR_CLOUDFLARE_TUNNEL_TOKEN
```

with the token copied from Cloudflare.

## 4. Add the two public routes in Cloudflare

In **Networking → Tunnels → outline-home → Routes**, add these published applications:

| Public hostname | Service URL |
| --- | --- |
| `outline.pi-coding.com` | `http://outline:3000` |
| `auth.pi-coding.com` | `http://host.docker.internal:5001` |

The first service is the Outline container on the Docker network. The second reaches Keycloak through its local port `5001`.

## 5. Switch local settings to the public hostnames

Update these lines in the untracked `docker.env` file:

```dotenv
URL=https://outline.pi-coding.com
OIDC_ISSUER_URL=https://auth.pi-coding.com/realms/outline
```

Create the Keycloak environment file and set its public hostname:

```sh
cp keycloak-outline/.env.example keycloak-outline/.env
```

Then edit `keycloak-outline/.env` so it contains:

```dotenv
KC_HOSTNAME=https://auth.pi-coding.com
```

## 6. Update the Keycloak client

Open Keycloak at `http://localhost:5001/admin`, select the `outline` realm and `outline` client, then set:

```text
Valid redirect URI: https://outline.pi-coding.com/auth/oidc.callback
Valid web origin: https://outline.pi-coding.com
```

Save the client configuration.

## 7. Start the public stack

Restart Keycloak to apply its public hostname, then start Outline and the tunnel:

```sh
(cd keycloak-outline && docker compose up -d --force-recreate)
docker compose -f docker-compose.yml -f docker-compose.cloudflare.yml up -d --force-recreate outline cloudflared
```

Wait for the Cloudflare dashboard to show the tunnel as **Healthy**, then open:

```text
https://outline.pi-coding.com
```

Cloudflare provides the public HTTPS certificate. The existing Caddy service is not used by the public tunnel; after changing `URL`, Outline redirects browser traffic to `outline.pi-coding.com`.

## Check and stop

Check connector logs:

```sh
docker compose -f docker-compose.yml -f docker-compose.cloudflare.yml logs -f cloudflared
```

Stop only the public connector:

```sh
docker compose -f docker-compose.yml -f docker-compose.cloudflare.yml stop cloudflared
```

Stopping the connector leaves the local Outline and Keycloak services running, but the public hostnames will no longer work.
