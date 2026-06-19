# OpenLMIS Eswatini Reporting Service

Eswatini-specific fork of the OpenLMIS [openlmis-report](https://github.com/OpenLMIS/openlmis-report) service. It manages and renders reports - JasperReports templates customized for the Eswatini OpenLMIS implementation.

Published as the Docker image **`openlmisci/eswatini-report`**, deployed in place of the stock `openlmis/report` service.

## Relationship to upstream

This repository is a fork of `OpenLMIS/openlmis-report`. Country-specific changes (custom reports, build pipeline) live here; upstream remains the source of truth for the core reporting engine.

- Upstream repo: https://github.com/OpenLMIS/openlmis-report
- `CHANGELOG.md` keeps the full upstream history below the `UPSTREAM HISTORY` divider;
  Eswatini versioning starts at the top.

Pull in upstream fixes periodically:

```bash
git remote add upstream https://github.com/OpenLMIS/openlmis-report.git   # one-time
git fetch upstream
git merge upstream/master
```

## Building & publishing the image

CI runs on **GitHub Actions** (`.github/workflows/build-and-push-image.yml`).
The workflow triggers on push to `master` and on manual dispatch, and it:

1. reads `serviceVersion` from `gradle.properties`,
2. builds the service jar - `docker compose -f docker-compose.builder.yml run --rm builder`,
3. builds the image - `docker compose -f docker-compose.builder.yml build image`,
4. logs in to Docker Hub and pushes `openlmisci/eswatini-report:<serviceVersion>`.

Required repository secrets: `DOCKER_HUB_USERNAME`, `DOCKER_HUB_PASSWORD`.

### Build locally

```bash
curl -o .env -L https://raw.githubusercontent.com/OpenLMIS/openlmis-ref-distro/master/settings-sample.env
docker compose -f docker-compose.builder.yml run --rm builder
docker compose -f docker-compose.builder.yml build image
```

## Developing custom reports

Reports are **data**, stored in the `report` schema (mainly the `jasper_templates` table). Add and maintain Eswatini reports as Flyway migrations under `src/main/resources/db/migration/`:

- **Jasper reports** - insert the compiled `.jrxml` (as `bytea`) into `jasper_templates`,
  or upload at runtime via `POST /api/reports/templates/common`.
- **Grouping** - categorise reports via `report_categories`.

Scaffold a migration file (runs the Gradle task in the build container):

```bash
docker compose run --rm --service-ports report
gradle generateMigration -PmigrationName=add_my_custom_report
```

New templates surface through the report API (`/api/reports/...`) - no UI change required.

## Versioning

Bump `serviceVersion` in `gradle.properties` (the image is tagged from it) and record the
change at the top of `CHANGELOG.md`, above the `UPSTREAM HISTORY` divider.

## Deployment

Deployed via `eswatini-deployment`: point the report service image at
`openlmisci/eswatini-report:<version>` and set `ESWATINI_REPORT_VERSION` in the environment.
The service registers with Consul under the name `report` (unchanged), so the gateway and
the rest of the stack resolve it exactly as before.

## More information

For core service architecture, API definitions, and configuration, see the upstream [openlmis-report README](https://github.com/OpenLMIS/openlmis-report/blob/master/README.md) and the [OpenLMIS documentation](https://docs.openlmis.org).
