# Shared Keycloak integration

Outline uses the Keycloak service in the sibling project:

```text
../common-keycloak-instance
```

It does not start or own a Keycloak container itself.

## Required service

Start the shared service before Outline:

```sh
(cd ../common-keycloak-instance && docker compose up -d)
docker compose up -d
```

For local mode, Outline uses this issuer:

```text
http://keycloak.local:5001/realms/outline
```

For Cloudflare mode, it uses:

```text
https://auth.pi-coding.com/realms/outline
```

The active values are set in the untracked `docker.env` file. See [MODES.md](MODES.md) to switch safely.

## Sign-in behavior

Outline uses the Keycloak client ID `outline`. When a user selects sign-in from Outline, the OIDC request includes this callback URL:

```text
https://outline.pi-coding.com/auth/oidc.callback
```

After authentication, Keycloak validates the callback against the `outline` client and returns the user to Outline. Other applications use their own clients and callbacks, so signing in from those applications returns users to those applications instead.

Do not reuse Outline's client secret in another application.

## Migration and rollback

The original Outline Keycloak realm, client configuration, and users were imported into the shared Keycloak PostgreSQL database. The previous stopped Keycloak Docker volume, `keycloak-outline_keycloak_data`, remains available as a rollback backup. Do not start the old Keycloak container while the shared instance is bound to port `5001`.
