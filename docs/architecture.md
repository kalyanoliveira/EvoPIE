# EvoPIE architecture notes

These notes preserve useful architectural context from older local run
notes. The current Docker and TLS workflow is documented in `README.md`
and `docs/docker-tls.md`.

## Layer overview

EvoPIE is organized as a Flask application with a separate updater process and
shared data/model layers.

```text
+--------------------------------------------------+
| evopie web app                 | updater process |
|                                |                 |
|   +------------------------+   |                 |
|   | Dash data dashboard    |   |                 |
|   |                        |   |                 |
|   |   +----------------+   |   |                 |
|   |   | analysislayer  |   |   |                 |
|   |   +----------------+   |   |                 |
+--------------------------------------------------+
| datalayer                                        |
+--------------------------------------------------+
```

## Components

- `evopie/` contains the main Flask application, page routes, API routes, quiz
  models, templates, and static assets.
- `evopie/datadashboard/` contains the Dash dashboard integration.
- `analysislayer/` computes dashboard data and generated visualizations.
- `updater.py` runs the analysis updater process outside the web request path.
- `datalayer/` owns shared SQLAlchemy models, database setup, and data access.
- `nginx/` contains the production reverse proxy image and generated TLS
  config.

## Runtime model

The web service and updater service share the same configured database URI.
The updater initializes or refreshes analysis/dashboard artifacts, while the
web service handles interactive application requests.

In Docker, the modern Compose setup uses profiles:

- `local` runs the web and updater services directly for HTTP development.
- `production` runs nginx in front of the web service for HTTPS deployment.

See `README.md` and `docs/docker-tls.md` for current commands and required
environment variables.
