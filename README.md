# Journal of Life Beyond Earth

Website and deployment configuration for the **Journal of Life Beyond Earth** — a scholarly journal in development, exploring life's origins, environments, survival, and future.

**Website:** https://lifebeyondearth.org

## Status

This is an infrastructure project. The journal is **not yet accepting submissions** and does not yet operate a formal peer-review process. Editorial roles, publishing responsibilities, and policies remain to be established.

## Architecture

- **Hosting:** VPSLabs VPS, Debian 12 with Docker
- **Existing reverse proxy:** Caddy, shared with `correlia.org`
- **Journal application:** [Open Journal Systems (OJS)](https://pkp.sfu.ca/software/ojs/)
- **Deployment:** Independent Docker Compose project in `/srv/lifebeyondearth`
- **Networking:** Existing external Docker network `journal_proxy` for Caddy-to-OJS traffic
- **Data:** Private journal database and unpublished manuscript storage (never committed to Git)

The existing Correlia application and PostGIS database must remain unchanged.

## Getting started

Read **[Deployment plan](docs/deployment.md)** for a safe, staged installation and Caddy integration. The server stack is intentionally not automatically deployed from GitHub. Review and pin official OJS and database image versions before enabling it.

## Repository conventions

- Never commit passwords, environment files, manuscripts, reviewer reports, database dumps, keys or other sensitive material.
- Keep editorial independence distinct from hosting and founding sponsorship.
- Back up and verify restoration of the existing Correlia environment before touching its proxy configuration.
- Apply changes in staging first; do not open public submissions until privacy, authentication, backups, email and editorial governance are ready.

## Sources

- [OJS official site](https://pkp.sfu.ca/software/ojs/)
- [OJS administration documentation](https://docs.pkp.sfu.ca/admin-guide/)
- [PKP's official container project](https://github.com/pkp/containers)
