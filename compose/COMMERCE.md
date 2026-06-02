# Commerce Setup

## Prerequisites

- Docker Desktop
- Composer keys with access to Adobe Commerce (EE) on `repo.magento.com`
- **GitHub SSH access** to the [magento-commerce](https://github.com/magento-commerce) organization — required to clone the Data Solutions extension repos during setup

## Setup

This repo is for **Adobe Commerce Enterprise** with **Adobe Data Solutions extensions** cloned
locally. Edition and version are read from `env/commerce.env`.

`bin/setup-commerce` handles the full install: teardown, extension repo cloning, EE download,
Magento install, 2FA disabled for development, Composer extras, `module:enable --all`, and compile.

Extension repos must be cloned **before** the first Docker start because `compose.commerce.yaml`
bind-mounts them into the container. `bin/setup-commerce` does this automatically.

```bash
bin/setup-commerce
```

Custom domain, edition, or version (all optional — defaults come from `env/commerce.env`):

```bash
bin/setup-commerce my-store.test enterprise 2.4.8-p3
```

## Sample Data and Dev Modules

After setup completes, run `bin/init-commerce` to install Magento sample data, enable a long
admin session lifetime, and disable password expiration for local development:

```bash
bin/init-commerce
```

## Overriding Magento Environment Variables

You can override any Magento environment variable by adding entries to `env/commerce.env` using this pattern:

```
CONFIG__DEFAULT__<namespace>__<key>=<value>
```

Be careful as these variables are always applied. They override whatever Magento has stored in the database or config files.

**Example** — set the SaaS environment to sandbox:

```bash
# example
CONFIG__DEFAULT__MAGENTO_SAAS__ENVIRONMENT=sandbox
```

## Volume Mount Configuration

Extension volume mounts are defined in `compose.commerce.yaml` and loaded automatically by
`bin/docker-compose` whenever that file is present (i.e. in any project scaffolded with
`lib/template-commerce`).

### Modules already in vendor (Adobe Commerce 2.4.8+)

The following modules now ship with Adobe Commerce and are **commented out** by default
in `compose.commerce.yaml` to avoid autoload conflicts:

| Extension repo                      | Vendor package(s)                                                                               |
| ----------------------------------- | ----------------------------------------------------------------------------------------------- |
| `services-connector`                | `magento/services-connector`                                                                    |
| `services-id`                       | `magento/module-services-id`, `module-services-id-graph-ql-server`, `module-services-id-layout` |
| `commerce-data-export/DataExporter` | `magento/module-data-exporter`                                                                  |
| `commerce-data-export/QueryXml`     | `magento/module-query-xml`                                                                      |
| `saas-export/SaaSCommon`            | `magento/module-saas-common`                                                                    |
| `data-solutions-magento-bff`        | `magento/module-graph-ql-server`, `magento/module-admin-graph-ql-server`                        |

### Developing on an absorbed module

To work on a module that's already in vendor:

1. Uncomment its volume mount in `compose.commerce.yaml`
2. Make sure the extension repo is checked out to a compatible branch
3. Run `bin/restart`

The volume mount overrides the vendor version at the container level. When done,
comment it back out and restart to return to the vendor version.
