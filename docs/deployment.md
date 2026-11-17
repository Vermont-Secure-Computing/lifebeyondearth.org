# Deployment plan: Journal of Life Beyond Earth

## Server baseline (observed November 2026)

- VPSLabs Debian 12 VPS with Docker 29.9.0
- Approximately 15 GiB RAM (14 GiB available when checked)
- Approximately 99 GB system disk (89 GB available when checked)
- Correlia Compose project: `/srv/correlia/src/docker-compose.yml`
- Correlia Caddyfile: `/srv/correlia/src/Caddyfile` mounted read-only in `correlia-caddy`
- Existing containers: `correlia-api`, `correlia-db`, `correlia-caddy`
- Correlia Caddy serves HTTP/HTTPS on host ports 80/443
- Network `journal_proxy` already created manually on VPS

No changes to the VPS are performed by adding these files to GitHub.

## Deployment principles

1. Preserve Correlia's running stack and database unchanged.
2. Use a **separate** Compose project rooted at `/srv/lifebeyondearth`.
3. Connect Caddy and OJS only through the shared external `journal_proxy` network. Keep the OJS database on a private OJS-only network.
4. Do not publish OJS or database ports to the public internet.
5. Keep unpublished manuscripts and submissions outside the publicly served web directory. Use separately persistent private storage.
6. Pin verified, supported versions of upstream OJS and database software. Prefer PKP's published container workflow, and review its documented volume, configuration, upgrade and security requirements before deploying.
7. Keep credentials in an ignored, server-local `.env`, not in GitHub; restrict filesystem permissions.
8. Back up and test restore of **both** journal database and private uploaded files. Maintain an off-VPS backup and an OJS update/rollback plan.
9. Enable HTTPS and configure trustworthy email delivery before accepting accounts or submissions.

## Phase 1 — verify and prepare (no service disruption)

- Confirm `journal_proxy` exists with `docker network inspect journal_proxy`.
- Make and verify a restorable backup of Correlia and the existing Caddyfile.
- Check the PKP [Admin Guide](https://docs.pkp.sfu.ca/admin-guide/) and [container project](https://github.com/pkp/containers) for a supported release and its explicit Docker Compose and persistent-storage contract.
- Confirm the intended deployment images/versions and test OJS in a non-public environment.
- Check which DNS A/AAAA records are needed for `lifebeyondearth.org` and `www.lifebeyondearth.org`.

## Phase 2 — isolated OJS stack

- Create `/srv/lifebeyondearth` and install a reviewed, working Compose file.
- Create a dedicated MariaDB database with separate credentials, or another explicitly supported OJS database.
- Ensure OJS gets its own persistent application assets, uploads, and configuration.
- Connect its web service to `journal_proxy`; put its database on a separate private network.
- Verify application startup, file permissions, database migrations, and persistence through container recreation **before** enabling public access.

## Phase 3 — integrate with the existing Caddy

Back up `/srv/correlia/src/Caddyfile`. Add the journal site blocks without altering existing Correlia site definitions. Example (assumes the OJS web container is called `ojs-web` and Caddy is attached to `journal_proxy`):

```caddyfile
lifebeyondearth.org {
    encode gzip
    reverse_proxy ojs-web:80
}

www.lifebeyondearth.org {
    redir https://lifebeyondearth.org{uri} permanent
}
```

**Caddy networking:** The existing `correlia-caddy` container currently belongs to the Correlia Compose-managed network. The preferred durable integration is to add `journal_proxy` as an external network to the **Caddy service only** in Correlia's Compose configuration. A manual `docker network connect journal_proxy correlia-caddy` is useful for testing, but may be lost if Compose recreates the container. Review and back up the Compose file before making this change.

**Caddy configuration:** Test with the exact Caddy version before reloading:

```bash
docker exec correlia-caddy caddy validate --config /etc/caddy/Caddyfile --adapter caddyfile
docker exec correlia-caddy caddy reload --config /etc/caddy/Caddyfile --adapter caddyfile
```

The original Caddyfile was previously validated successfully. The user-provided terminal paste showed an `EOF` line after the site blocks; inspect and remove this only if it is actually in the on-disk file as part of a reviewed change.

## Phase 4 — acceptance tests

- `https://correlia.org` and existing Correlia API routes still work.
- Journal public domain serves the correct OJS site with valid HTTPS certificate.
- Journal database cannot be reached from the public internet.
- Anonymous visitors cannot fetch uploaded submissions or reviewer files directly.
- Recreate OJS containers and verify published content persists.
- Restore database and private files to a separate test environment.
- Check email deliverability, scheduled tasks, updates, security hardening, and incident recovery.
- Keep submissions disabled until formal editorial governance, consent and privacy procedures are ready.

## Caution: this repository is not a deployed journal

The current `compose.yaml` documents intended network isolation but deliberately contains no application services. It **must not** be treated as production-ready. The next engineering step is selecting and validating a specific upstream OJS release/image and completing the deployment manifests against its real configuration.
