# How to run EvoPIE

## Simple local development

Run:

```bash
just local-up
```

The `local-up` target runs:

```bash
docker compose --profile local up --build -d
```

This starts the `local` Docker Compose profile. It builds the app from your
current checkout, publishes the app on port 5000 without nginx or TLS, and runs
in detached mode. Open `http://127.0.0.1:5000` after startup.

The local profile stores EvoPIE data in `./data` by default. Set
`EVOPIE_DATA_DIR` before starting if you want to store the database and uploads
somewhere else.

## Watched local development

Run:

```bash
just local-watch
```

The `local-watch` target runs:

```bash
just check-watch-ignore
docker compose --profile local up --build --watch
```

The first command checks that ignored paths match in Compose Watch and the
project's Docker ignore files. That check runs:

```bash
./scripts/check-compose-watch-ignore.py
```

The second command starts the `local` Docker Compose profile with Compose Watch
enabled. Source changes are synced into the running containers and the affected
service restarts. Dependency or Dockerfile changes still rebuild images.

Compose Watch stays attached to your terminal. Docker Compose does not allow
combining `--watch` with detached mode.

## Deployment to production

Run:

```bash
just prod-up example.edu /etc/letsencrypt /srv/evopie/data master
```

The `prod-up` target runs:

```bash
EVOPIE_SERVER_NAME="example.edu" \
EVOPIE_CERT_DOMAIN="example.edu" \
EVOPIE_CERTS_DIR="/etc/letsencrypt" \
EVOPIE_DATA_DIR="/srv/evopie/data" \
EVOPIE_GIT_REF="master" \
docker compose --profile production up --build -d
```

Replace `example.edu`, `/etc/letsencrypt`, and `/srv/evopie/data` with your
production domain, certificate directory, and data directory. The final
argument is optional and defaults to `master`; use it to deploy another
branch, tag, or commit.

The production profile runs the Flask app behind nginx with HTTPS.
Production builds fetch the upstream repository in the application Dockerfiles.
The nginx service
expects certificate files under:

```text
${EVOPIE_CERTS_DIR}/live/${EVOPIE_CERT_DOMAIN}/fullchain.pem
${EVOPIE_CERTS_DIR}/live/${EVOPIE_CERT_DOMAIN}/privkey.pem
```

Production requires `EVOPIE_DATA_DIR`, `EVOPIE_SERVER_NAME`, and
`EVOPIE_CERTS_DIR`. To use a database URI other than the default
`sqlite:////app/data/db.sqlite`, set `EVOPIE_DATABASE_URI` before starting.

## Local HTTPS mode

First create a local self-signed certificate:

```bash
just local-certs localhost ./certs
```

The `local-certs` target runs:

```bash
EVOPIE_CERT_DOMAIN="localhost" \
EVOPIE_CERTS_DIR="./certs" \
./scripts/create-local-certs.sh
```

The helper writes certificate files to:

```text
./certs/live/localhost/fullchain.pem
./certs/live/localhost/privkey.pem
```

Then start the production profile with local HTTPS settings:

```bash
just prod-up localhost ./certs ./data master
```

That runs the same `prod-up` command shown above, but with local values. Local
HTTPS uses nginx, so it uses the `production` Compose profile even when the
certificate is self-signed. Open `https://127.0.0.1:5000` after startup.

A browser warning is expected because the certificate is self-signed. Do not
commit generated certificate files or private keys.
