# Run modes: localhost and Cloudflare Tunnel

Use one mode at a time. The active mode is determined by the untracked `docker.env` file, the shared `../common-keycloak-instance/.env` file, and the Keycloak client's redirect URI.

| Setting | Localhost mode | Cloudflare mode |
| --- | --- | --- |
| Outline URL | `https://outline.localhost:9443` | `https://outline.pi-coding.com` |
| Keycloak issuer | `http://keycloak.local:5001/realms/outline` | `https://auth.pi-coding.com/realms/outline` |
| Shared Keycloak hostname | `http://keycloak.local:5001` | `https://auth.pi-coding.com` |
| Outline client redirect URI | `https://outline.localhost:9443/auth/oidc.callback` | `https://outline.pi-coding.com/auth/oidc.callback` |
| Outline client web origin | `https://outline.localhost:9443` | `https://outline.pi-coding.com` |
| Public connector | Not running | `cloudflared` running |

## Switch to localhost mode

1. Stop the public connector:

   ```sh
   docker compose -f docker-compose.yml -f docker-compose.cloudflare.yml stop cloudflared
   ```

2. Edit the untracked `docker.env` file:

   ```dotenv
   URL=https://outline.localhost:9443
   OIDC_ISSUER_URL=http://keycloak.local:5001/realms/outline
   ```

3. Edit `../common-keycloak-instance/.env`:

   ```dotenv
   KC_HOSTNAME=http://keycloak.local:5001
   ```

4. In Keycloak, update the `outline` client's **Valid redirect URIs** and **Web origins** to the localhost values in the table above.

5. Recreate the shared Keycloak service and Outline:

   ```sh
(cd ../common-keycloak-instance && docker compose up -d --force-recreate)
   docker compose up -d --force-recreate outline
   ```

6. Open `https://outline.localhost:9443`.

## Switch to Cloudflare Tunnel mode

Before switching, ensure the Cloudflare Tunnel is configured as described in [CLOUDFLARE_TUNNEL.md](CLOUDFLARE_TUNNEL.md), including both published routes.

1. Edit the untracked `docker.env` file:

   ```dotenv
   URL=https://outline.pi-coding.com
   OIDC_ISSUER_URL=https://auth.pi-coding.com/realms/outline
   ```

2. Edit `../common-keycloak-instance/.env`:

   ```dotenv
   KC_HOSTNAME=https://auth.pi-coding.com
   ```

3. In Keycloak, update the `outline` client's **Valid redirect URIs** and **Web origins** to the Cloudflare values in the table above.

4. Recreate the shared Keycloak service and start Outline with the public connector:

   ```sh
(cd ../common-keycloak-instance && docker compose up -d --force-recreate)
   docker compose -f docker-compose.yml -f docker-compose.cloudflare.yml up -d --force-recreate outline cloudflared
   ```

5. Wait until Cloudflare reports the tunnel as **Healthy**, then open `https://outline.pi-coding.com`.

## Important notes

- Do not commit `docker.env`, `../common-keycloak-instance/.env`, or `docker-compose.cloudflare.yml`; they contain deployment-specific configuration and credentials.
- The Keycloak `outline` client must match the mode currently in use. A mismatch causes sign-in failures or redirects to the wrong hostname.
- When returning to localhost mode, keep the Cloudflare DNS routes in place if desired; with `cloudflared` stopped, the public hostnames simply become unavailable.
