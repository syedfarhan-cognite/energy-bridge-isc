# Configure `config.<env>.yaml`

*Set the tenant's values in `config.<env>.yaml` and add the ISC modules to `selected`.*

**Before you start:** Pages [1](./1-overview-and-prerequisites.md) and [2](./2-customer-repo.md) are done.

## Where configuration lives

Two files, both in the customer repo root:

- **`.env`** — auth. Already covered in [prerequisites](./1-overview-and-prerequisites.md#before-you-start).
- **`config.<env>.yaml`** — the `variables:` block and `environment.selected`. `cdf build` reads this file.

## 1. Set the tenant values

| Variable | What it is |
|---|---|
| `cdf_project` | CDF project name, e.g. `${CDF_PROJECT}`. |
| `cdf_cluster` | CDF cluster, e.g. `${CDF_CLUSTER}`. |
| `idp_tenant_id` | Entra tenant GUID, e.g. `${IDP_TENANT_ID}`. |
| `hierarchy_path_trigger_clientId` | Client ID for the hierarchy-path workflow trigger (`isc_core`), e.g. `${HIERARCHY_PATH_TRIGGER_CLIENT_ID}`. |
| `records_archive_trigger_clientId` | Client ID for the 10 records-archive workflow triggers (`isc_archival_records`), e.g. `${RECORDS_ARCHIVE_TRIGGER_CLIENT_ID}`. |
| `isc_InstanceSpace` | The space this tenant's ISC data instances live in. |
| `isc_RawDatabase` | The RAW database this tenant lands data into. The shipped value is the test database `BTP_EDGE_TEST`. Unlike the `${…}` variables above, a forgotten value here doesn't error — it silently builds and deploys against ISC's own test data. |

These variables go under a module key named `isc`, nested inside `variables.modules` — not at the top level of `variables:`. If the customer repo already has other modules' variables there, add `isc:` as a sibling key under `modules:`, at the same indentation. One extra or missing space under `modules:` or `isc:` fails `cdf build`. This nesting is only for `variables:` — `environment.selected` (step 3 below) stays a flat list, unaffected.

```yaml
variables:
  modules:
    isc:
      cdf_project: ${CDF_PROJECT}
      cdf_cluster: ${CDF_CLUSTER}
      idp_tenant_id: ${IDP_TENANT_ID}
      hierarchy_path_trigger_clientId: ${HIERARCHY_PATH_TRIGGER_CLIENT_ID}
      records_archive_trigger_clientId: ${RECORDS_ARCHIVE_TRIGGER_CLIENT_ID}
      isc_InstanceSpace: acme_isc_instances
      isc_RawDatabase: acme_isc_landing
```

## 2. Leave the fixed variables alone

Copy these into `config.<env>.yaml` too, under the same `isc:` node as section 1, with the exact values below. "Leave alone" means don't change the values — not that you can skip adding them. `cdf build` fails on any that are missing.

| Variable | Purpose |
|---|---|
| `isc_dmVersion` | Version for every ISC data model. Bump together with `isc_viewVersion` on an upgrade — [page 7](./7-upgrade.md). |
| `isc_viewVersion` | Version for every ISC view. Bump together with `isc_dmVersion` on an upgrade — [page 7](./7-upgrade.md). |
| `isc_OrganizationalFoundationSchemaSpace` | Organizational Foundation schema. |
| `isc_MaterialManagementSchemaSpace` | Material Management schema. |
| `isc_AssetMaintenanceSchemaSpace` | Asset Maintenance schema. |
| `isc_CustomerDemandFulfilmentSchemaSpace` | Customer Demand Fulfilment schema. |
| `isc_ShipmentDeliverySchemaSpace` | Shipment & Delivery schema. |
| `isc_SourcingProcurementSchemaSpace` | Sourcing & Procurement schema. |
| `isc_PlanForecastSchemaSpace` | Planning & Forecasting schema. |
| `isc_TechnicalStructureSchemaSpace` | Bill of materials schema. |
| `isc_IsaSchemaSpace` | Manufacturing schema. |
| `isc_IscPlatformSchemaSpace` | Sync and ingest state schema. |
| `isc_TypedModelSchemaSpace` | Typed hierarchy views. |
| `isc_RecordsSpace` | Archived Records. Owned by `isc_archival_records`. |

```yaml
      isc_dmVersion: v1.6.0
      isc_viewVersion: v1.6.0
      isc_OrganizationalFoundationSchemaSpace: sp_organizational_foundation
      isc_MaterialManagementSchemaSpace: sp_material_management
      isc_AssetMaintenanceSchemaSpace: sp_asset_maintenance
      isc_CustomerDemandFulfilmentSchemaSpace: sp_customer_demand_fulfilment
      isc_ShipmentDeliverySchemaSpace: sp_shipment_delivery
      isc_SourcingProcurementSchemaSpace: sp_sourcing_procurement
      isc_PlanForecastSchemaSpace: sp_plan_forecast
      isc_TechnicalStructureSchemaSpace: sp_technical_structure
      isc_IsaSchemaSpace: sp_isa_manufacturing
      isc_IscPlatformSchemaSpace: sp_isc_platform
      isc_TypedModelSchemaSpace: sp_isc_solution
      isc_RecordsSpace: sp_supply_chain_records
```

These are product identifiers, not tenant settings. Renaming any of them — or changing their values — breaks the fixed schema, the Functions that read it, and every future upgrade.

## 3. Add the ISC modules to `selected`

`isc_core`, the two `isc_connect_*` modules, `isc_edges/*`, `isc_atlas_agents` and `isc_archival_records` are required. `isc_atlas_agents` and `isc_archival_records` depend on `isc_core` (its views and instance space), so keep them in the same `selected` list and deploy them together. `isc_data_mesh` is optional — only add it if this tenant is opting into the Data Product quality track. `isc_seed_data_ingestion` contains seed data only. Copy it into the customer repo and add it to `selected` only when the demo graph has to be deployed for testing; otherwise leave it out of both the repo and the config.

```yaml
environment:
  name: dev
  project: ${CDF_PROJECT}
  validation-type: dev
  selected:
    - modules/auth                             # customer's own
    - modules/ingest                           # customer's own
    - modules/isc/isc_core
    - modules/isc/isc_connect_maintenance
    - modules/isc/isc_connect_manufacturing
    - modules/isc/isc_edges/isc_core_edges
    - modules/isc/isc_edges/isc_connect_edges
    - modules/isc/isc_atlas_agents             # Atlas agents, skills, fn_btp_graph, fn_btp_insight
    - modules/isc/isc_archival_records      # records archive, streams, archive triggers
    - modules/isc/isc_data_mesh                # optional — only for the Data Product quality track
    - modules/isc/isc_seed_data_ingestion      # seed data only — add only to deploy demo/test data
```

> [!WARNING]
> Replacing the customer's existing `selected:` list instead of adding to it un-deploys their other modules on the next `cdf deploy`.

## Dev and prod

`${CDF_PROJECT}` only works for `validation-type: dev`. Prod and staging need an **explicit** project name — `cdf build` fails otherwise:

```text
Project name mismatch between project set in the environment section of 'config.prod.yaml'
and the environment variable 'CDF_PROJECT'
```

The shipped `config.prod.yaml` has the placeholder `example_prod` — replace it with the tenant's real project name. Keep the same `variables:` across environments; only `environment.name`, `environment.project`, and `environment.validation-type` differ.

## If it fails

| Symptom | Cause | Fix |
|---|---|---|
| A literal `${…}` shows up in a build or in a workflow trigger | The env var behind it is unset | Set it in `.env` ([page 1](./1-overview-and-prerequisites.md#before-you-start)). |
| Project-name mismatch on `cdf build -e prod` | `${CDF_PROJECT}` used in prod | Write the explicit project name. |
| A fixed variable is missing | The modules and the config come from different releases | Copy the modules again from the same release. |

## What's next

- [Build, deploy, and verify](./4-deploy-and-verify.md)
