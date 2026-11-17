# Deploy Integrated Supply Chain: overview and prerequisites

*What deploying ISC involves, and what to have ready before you start.*

**Who this is for:** data engineers and solution architects deploying ISC into a customer's CDF project. New to ISC? Read [what it is](../isc-functional-overview/overview.md) first — this page is about deploying it, not what it models.

## What deploying involves

ISC ships as a Cognite Toolkit module tree. You bring it into the customer's existing Toolkit repository and deploy it alongside every other module that tenant already runs — there is no separate ISC installer. If you only want to deploy ISC itself, with no other tenant modules, you can deploy this offering's repo directly instead — see [Deploying ISC standalone](#deploying-isc-standalone) below.

2. [Bring ISC into the customer repo](./2-customer-repo.md)
3. [Configure `config.<env>.yaml`](./3-configure.md)
4. [Build, deploy, and verify](./4-deploy-and-verify.md)
5. [Load the seed data](./5-load-seed-data.md)

## The rule that decides where changes go

| Module | Role |
|---|---|
| `isc_core` | The product-owned supply-chain schema (organizational foundation, material management, demand fulfilment, sourcing/procurement, shipment/delivery, planning, technical structure), the platform spaces, the location filter, and the hierarchy Functions, workflows and triggers. Types, views and external IDs ship as the product baseline; a handful of its relations get **repointed** at the customer's own maintenance/manufacturing models if the reference connect modules are replaced (see [customize](./6-customize.md)). |
| `isc_edges/isc_core_edges` | Product-owned edge (relationship) types for the schema above. Ships with `isc_core` as part of the product baseline. New tenant-specific relationships go in `isc_edges/isc_connect_edges` instead (see [customize](./6-customize.md)). |
| `isc_connect_maintenance`, `isc_connect_manufacturing`, `isc_edges/isc_connect_edges` | Reference implementations wiring `isc_core` to a customer's **existing** maintenance (CMMS) or manufacturing (ISA-95/ISA-88) systems — replaced by the customer's own modules, or kept and left as-is if the tenant doesn't have its own. |
| `isc_atlas_agents` | The Atlas layer: the 3 ISC agents, the 4 Atlas skills, and the `fn_btp_graph` and `fn_btp_insight` Functions the agents call. Product-owned — agent, skill and Function external IDs are a contract. Deploy after `isc_core`. |
| `isc_archival_records` | The records archive: the records space and containers, the archive streams, `fn_btp_records_archive`, `wf_btp_records_archive`, and the 10 per-view archive triggers. Product-owned. Deploy after `isc_core` (the domain views and instance space must exist). |
| `isc_data_mesh` (optional) | The data-product catalog and quality rules over the model above. Rarely edited — only to add a new data product. |
| `isc_seed_data_ingestion` (demo only) | The Nordic Packaging Corp demo graph: RAW tables, transformations and `wf_btp_seed_data_ingest`. It contains seed data only. Copy it into the customer repo only when the demo data has to be deployed for testing, and leave it out of customer production repos and configs. |

`isc_atlas_agents` and `isc_archival_records` are part of the ISC solution, not add-ons: include both in `environment.selected` for every tenant. Only `isc_data_mesh` and `isc_seed_data_ingestion` are optional.

Where a specific change goes: [customize](./6-customize.md).

> **Never:**
> - Commit `.env` to any repository — it holds secrets.

## What `cdf deploy` doesn't do

`cdf deploy` creates schema, Functions, and workflows. It does **not** load data into the domain models. For the shipped demo graph, manually run `wf_btp_seed_data_ingest` in Fusion afterward — [step 5](./5-load-seed-data.md).

## Before you start

Work in the customer's repo — unless you only want to deploy ISC itself, with no other tenant modules, in which case this offering's repo works directly (see [Deploying ISC standalone](#deploying-isc-standalone)). `cdf auth verify` should already succeed there.

- A CDF project with **Cognite Functions** activated — request it; allow up to ~2 hours.
- A service principal with: data models, RAW, transformations, files, functions, sessions, workflows, plus:
  - `agentsAcl` — for the Atlas agents and skills in `isc_atlas_agents`.
  - `streamRecordsAcl:WRITE` — for the archive streams in `isc_archival_records`.
- The customer repo's `.env` already set up for [`client_credentials`](https://docs.cognite.com/cdf/deploy/cdf_toolkit/guides/auth#client_credentials) — the same as any Toolkit project — plus two ISC-specific variables with no fallback:

  ```text
  HIERARCHY_PATH_TRIGGER_CLIENT_ID=<entra-app-id>
  RECORDS_ARCHIVE_TRIGGER_CLIENT_ID=<entra-app-id>
  ```

  Use the same value as `IDP_CLIENT_ID` if the tenant uses one Entra app. If either is unset, the triggers it covers are skipped or fail at Entra with `AADSTS700016`.
- The Cognite Toolkit CLI (`cdf`), matching the `[modules] version` this offering's `cdf.toml` ships.

Collect these values for the tenant before you start — reusing another tenant's identifiers here overwrites that tenant's data:

| Value | Example (not a real value) |
|---|---|
| CDF project | `example_prod` |
| Cluster | `bluefield` |
| Entra tenant ID | GUID |
| Instance space | `acme_isc_instances` |
| RAW database | `acme_isc_landing` |

## Deploying ISC standalone

If you only want ISC itself, with no other tenant modules, deploy this offering's own repo instead of bringing it into a customer repo.

1. **Env** — set in `.env` (not committed): `CDF_PROJECT`, `CDF_CLUSTER`, `IDP_TENANT_ID`, the [`client_credentials`](https://docs.cognite.com/cdf/deploy/cdf_toolkit/guides/auth#client_credentials) variables, plus `HIERARCHY_PATH_TRIGGER_CLIENT_ID` and `RECORDS_ARCHIVE_TRIGGER_CLIENT_ID` — same requirements as above.
2. **Build** — `cdf build --env <env>`, using this repo's own `config.dev.yaml` / `config.prod.yaml`.
3. **Dry-run** — `cdf deploy --dry-run --env <env>`. Review the plan.
4. **Deploy** — `cdf deploy --env <env>`.
5. **Seed** (optional) — run `wf_btp_seed_data_ingest` in Fusion — see [step 5](./5-load-seed-data.md).

## Check it worked

`cdf auth verify` succeeds and reports the customer's project and cluster, not an internal test project.

## What's next

- [Bring ISC into the customer repo](./2-customer-repo.md)
