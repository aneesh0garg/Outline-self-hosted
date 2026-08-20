# Setup guide

This guide configures a fresh local Outline installation using the shared Keycloak instance for sign-in.

## Prerequisites

- Docker Desktop with Docker Compose v2
- macOS, Linux, or Windows with administrator access to add a local host entry
- A browser

Verify Docker is ready:

```sh
docker compose version
docker info
```

## 1. Configure the environment

Create the untracked runtime configuration from the template:

```sh
cp docker.env.example docker.env
cp postgres.env.example postgres.env
```

Generate two high-entropy values and set them as `SECRET_KEY` and `UTILS_SECRET`. On macOS or Linux, one option is:

```sh
openssl rand -hex 32
```

Set a strong `POSTGRES_PASSWORD` in `postgres.env`, then use the same value in `DATABASE_URL` in `docker.env`. Set `OIDC_CLIENT_SECRET` after creating the Outline client in the shared Keycloak instance in step 3.

The default local values are:

```dotenv
URL=https://outline.localhost:9443
OIDC_ISSUER_URL=http://keycloak.local:5001/realms/outline
OIDC_CLIENT_ID=outline
```

## 2. Make Keycloak resolvable

Outline reaches Keycloak through Docker's host gateway. Your browser also needs to resolve the issuer hostname during sign-in. Add this line to the host machine's hosts file:

```text
127.0.0.1 keycloak.local
```

On macOS or Linux:

```sh
sudo sh -c 'echo "127.0.0.1 keycloak.local" >> /etc/hosts'
```

On Windows, add the same entry to `C:\\Windows\\System32\\drivers\\etc\\hosts` from an elevated editor.

## 3. Start and configure the shared Keycloak instance

Start the shared service first:

```sh
(cd ../common-keycloak-instance && docker compose up -d)
```

Open `http://localhost:5001/admin` and sign in with the administrator from `../common-keycloak-instance/.env`.

Create the OIDC resources:

1. Select the shared realm (the migrated installation uses `outline`; future applications can use the same realm).
2. Create or update the client with Client ID `outline` and client authentication enabled.
3. Set the client protocol to OpenID Connect and enable the standard authorization-code flow.
4. Add `https://outline.localhost:9443/auth/oidc.callback` as a valid redirect URI.
5. Add `https://outline.localhost:9443` as a valid web origin.
6. Copy the generated client secret into `OIDC_CLIENT_SECRET` in `docker.env`.
7. Create at least one user, give it a password, and ensure its email and username are set.

The discovery endpoint should then be available from the shared service at:

```text
http://keycloak.local:5001/realms/outline/.well-known/openid-configuration
```

## 4. Trust the local HTTPS certificate

Caddy uses a local certificate for `outline.localhost`. If the browser warns that the certificate is not trusted, trust the generated Caddy root certificate.

On macOS, import `caddy-local-root.crt` into Keychain Access and set it to **Always Trust**. If the file is not present or does not match the currently running Caddy instance, export the root certificate from Caddy's data volume and trust that certificate instead.

For a quick local verification only, you can continue through the browser warning; do not use this workaround in a production environment.

## 5. Start Outline

From the repository root:

```sh
docker compose up -d
docker compose ps
```

Wait until `outline`, `postgres`, and `redis` report healthy. Then open:

```text
https://outline.localhost:9443
```

Use the Keycloak sign-in option. The first successful user becomes the initial Outline administrator.

Continue with the [run guide](RUN.md) for routine operation and troubleshooting.
