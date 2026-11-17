# Load the seed data

*`cdf deploy` creates the models and the empty tables, but no data. Run the seed workflow by hand to load the demo graph.*

**Before you start:** All 5 Functions show **Ready** in Fusion ([page 4](./4-deploy-and-verify.md)). Starting earlier fails: `fn_btp_edge` (shipped with this module) runs as part of the workflow itself, and `fn_btp_hierarchy` (from `isc_core`) is invoked right after by the hierarchy dataModeling triggers to sync `root`/`path` once the hierarchy data lands.

## Where the seed workflow comes from

The seed workflow, its RAW tables and its transformations ship in the `isc_seed_data_ingestion` module, which is **for seed data only** and **demo or test only**. It is not part of the ISC solution itself. Copy the module into the customer repo's `modules/isc/` only when you need to deploy the demo data, and skip it for a tenant that loads its own data. Add `modules/isc/isc_seed_data_ingestion` to `environment.selected` ([page 3](./3-configure.md#3-add-the-isc-modules-to-selected)) and deploy it **together with `isc_core`**, in the same `cdf build`. The transformations read spaces and views that `isc_core` defines, so a build of the seed module alone fails with `Unknown reference to spaces` or `Unknown reference to views`.

## Decide first

| Do you want the shipped demo graph? | Then |
|---|---|
| Yes — demo, training, or trying the agents on sample data | Run this page. |
| No — the tenant loads only its own data | Skip it. The Atlas sample questions that rely on the demo graph will find nothing until some data is ingested. |

> [!WARNING]
> Don't load the demo graph into a project that holds real data — the sample IDs are fictional and will mix with the tenant's own records.

## Steps

1. In Fusion, open **Workflows**, then `wf_btp_seed_data_ingest`.
2. Start version `1` manually.
3. Wait for the status **COMPLETED**.

## What it does

It loads the `DeclaredTypes` catalog first, then the business, product, geography, and warehouse hierarchies, then the rest of the domain data — materials, orders, shipments, work orders, and their relationships. Path sync runs after the graph lands, and an edge check runs at the end.

## What the demo graph contains

42 seed tables covering all the domains — small enough to explore by hand. The Atlas guide's sample questions rely on this data.

## Check it worked

The run status is **COMPLETED**, and **Data explorer** shows instances on views such as `MaterialMaster`, `PurchaseOrderLineItem`, and `Shipment`.

## If it fails

| Symptom | Cause | Fix |
|---|---|---|
| The run fails at the start | A Function is not Ready yet | Wait for all 5, then start again. |
| A task fails mid-run | A transformation error | Open that transformation's logs. Don't treat a failed run as a loaded graph. |
| `cdf build` reports `Unknown reference to spaces` or `views` for seed transformations | `isc_seed_data_ingestion` was built without `isc_core` | Build with the full `selected` list, so `isc_core` is in the same build. |
| The run completes but Data explorer is empty | Looking in the wrong space | Check the instance space the config sets ([page 3](./3-configure.md)). |

## Re-running

A normal upgrade doesn't re-run the seed workflow. Re-run version `1` only to refresh or reload the demo graph.

## What's next

- [Customize for the tenant](./6-customize.md)
