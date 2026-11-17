# Data models

*How ISC's spaces, data models, containers, and views fit together — and the two patterns that sit on top: typed hierarchy views and edge views.*

**Who this is for:** data engineers and solution architects.

## Concepts

| Term | What it is |
|---|---|
| Space | The top-level, access-controlled container for a set of data models, containers, views, and instances. |
| Container | Defines the properties and storage for a node or edge type — the schema. |
| View | A queryable projection of one or more containers' properties, optionally filtered (e.g. by `declaredType`). |
| Direct relation | A property on a node or edge pointing at one other node. |
| Reverse direct relation | A view property that lists every node pointing back at this one through a named direct relation elsewhere (computed, not stored — e.g. a `Zone`'s `StorageBin` children, reversed off `StorageLocations.parent`). |
| Edge | A typed, queryable relationship instance between a start node and an end node, with its own properties — see [Direct relations and edges](#direct-relations-and-edges). |

## Spaces

ISC's reference architecture spans all nine domains from the [functional overview](../isc-functional-overview/overview.md#the-nine-domains-7-core-2-connect-layers) — but this training demo data and fully-built containers/views cover only the **upstream** side (sourcing/procurement, asset maintenance, inbound logistics, material management, planning/forecast, organizational foundation). The **downstream** domain spaces (customer demand fulfilment, technical structure, manufacturing) are kept in the module tree so the full domain picture stays visible, scaffolded for a tenant to build out when those domains come into scope.

12 spaces, split by what they hold:

| Space | Holds |
|---|---|
| `sp_isc_solution` | The typed hierarchy views (see [Typed hierarchy views](#typed-hierarchy-views) below) |
| `sp_isc_platform` | Hierarchy path-sync and RAW-ingest state (`FunctionSyncState`, `RawIngestState` containers/views) |
| The instance space (default `sp_isc_supply_chain_instances`) | All business hierarchy and domain data instances |
| 5 upstream domain schema spaces | Organizational foundation, material management, sourcing/procurement, logistics (shipment/delivery), planning/forecast — containers, views, and demo data all built |
| `sp_isc_customer_demand_fulfilment`, `sp_isc_technical_structure` | Downstream domain spaces, part of the full ISC reference architecture. Reserved for a tenant to build out `DataModel.yaml`, containers, and views when customer demand fulfilment or technical structure are in scope — not populated in this training repo |
| `sp_isc_asset_maintenance` | `isc_connect_maintenance` schema (replaceable reference module) — upstream, fully built |
| `sp_isa_manufacturing` | `isc_connect_manufacturing` schema (replaceable reference module) — downstream manufacturing-execution domain. Containers are scaffolded; no `DataModel.yaml`/views built since this training repo's demo data is upstream-only |

An edge view (see [Direct relations and edges](#direct-relations-and-edges)) lives in the schema space of one of the domains it connects (for example `MaterialReservation` is in the material management space). There is no separate edges space.

## Data models

8 `DataModel.yaml` files — all on the upstream side (see [Spaces](#spaces) above). The downstream domains (customer demand fulfilment, technical structure, manufacturing) have no `DataModel.yaml` yet, just their reserved space:

| Data model | Space | Contains |
|---|---|---|
| `isc_typed_model` | `sp_isc_solution` | Concrete hierarchy views — `Company`, `BusinessUnit`, `CostCenter`, `GeoRegion`, `Country`, `City`, `Warehouse`, `Zone`, `StorageBin` |
| `IscPlatform` | `sp_isc_platform` | `FunctionSyncState`, `RawIngestState` |
| `OrganizationalFoundation`, `MaterialManagement`, `SourcingProcurement`, `ShipmentDelivery`, `PlanForecast` | 5 upstream domain spaces | The 5 built domains covered in [supply-chain](../isc-functional-overview/overview.md) |
| `AssetMaintenance` | `sp_isc_asset_maintenance` | `isc_connect_maintenance` reference module (upstream, fully built) |

Not yet built: `CustomerDemandFulfilment`, `TechnicalStructure`, and `Manufacturing` — their spaces exist (`sp_isc_customer_demand_fulfilment`, `sp_isc_technical_structure`, `sp_isa_manufacturing`) but have no `DataModel.yaml`, so they won't show up in Fusion's **Data models** list yet. `isc_connect_manufacturing` does ship containers for a future `Manufacturing` model (`Equipment`, `Personnel`, `ProductionSchedule`, etc.) — see [Containers and views](#containers-and-views).

## Containers and views

Standard pattern: each entity (e.g. `Supplier`, `MaterialLot`) has one container holding its properties and one view projecting those properties, both in the entity's domain schema space. Every container/view carries the same base source-tracking properties — `sourceId`, `sourceContext`, `sourceCreatedTime`, `sourceUpdatedTime`, `sourceCreatedUser`, `sourceUpdatedUser` — on top of its entity-specific fields. Total container and view counts are owned by [page 4](../isc-deployment-and-customization/4-deploy-and-verify.md#4-check-fusion) — not repeated here.

Every data model and view also carries a version, driven by the `dmVersion` config variable (`v1.0.0` today) — see [page 3](../isc-deployment-and-customization/3-configure.md). A separate `viewVersion` variable is also defined in config but isn't referenced anywhere in the shipped YAML; `dmVersion` alone drives both data model and view versions.

Core views don't implement CDM views, but a couple relate to one: `MaterialMaster` and `Plan` each carry a typed direct relation to `cdf_cdm`'s `CogniteUnit` (`v1`) for their unit of measure. `isc_connect_maintenance`'s `FunctionalAsset` goes further and implements `cdf_cdm`'s `CogniteAsset` (`v1`) outright. `cdf_cdm` is the Cognite Core Data Model — a system space every CDF project provides by default, so this needs no extra setup.

The unbuilt downstream domains carry scaffolded containers too: `technical_structure` ships `BillOfMaterialsItem` and `BillOfMaterialsHeader` containers (no views yet), and `isc_connect_manufacturing` ships an `ISAAsset` container that also models a `CogniteAsset` relation — neither is queryable until their data model is built.

## Typed hierarchy views

Hierarchy entities use a different pattern: one container per hierarchy tree in the domain space (e.g. `BusinessUnits` in `OrganizationalFoundation`) holds every level of that tree and a `declaredType` property. A concrete view per level — `Company`, `BusinessUnit`, `CostCenter` for the business hierarchy (similarly `GeoRegion`/`Country`/`City` for geography, `Warehouse`/`Zone`/`StorageBin` for storage) — lives in `sp_isc_solution`'s `isc_typed_model` data model, filtering the shared container on `declaredType` to project only the matching nodes. This is why hierarchy views live in a different space (`sp_isc_solution`) than their container (the domain schema space).

See [Build a new hierarchy](../isc-deployment-and-customization/advanced/custom-hierarchies.md#build-a-new-hierarchy) for how to add a new typed view and trigger when extending a hierarchy.

## Direct relations and edges

8 typed edge views, split across two modules under `isc_edges/`:

| Edge view | Connects | Module |
|---|---|---|
| `SupplierSuppliesMaterial` | Supplier → Material | `isc_core_edges` |
| `LaneTransportsMaterial` | Transportation lane → Material | `isc_core_edges` |
| `LotStoredInBin` | Material lot → Storage bin | `isc_core_edges` |
| `ShipmentExecutesReceipt` | Shipment → Scheduled receipt | `isc_core_edges` |
| `ShipmentExecutesTransfer` | Shipment → Scheduled transfer | `isc_core_edges` |
| `MaterialReservation` | Material inventory → work order component | `isc_connect_edges` |
| `NotificationLinksEntity` | Master notification → the entity it impacts | `isc_connect_edges` |
| `UsesTransportationLane` | Shipment → Transportation lane | `isc_connect_edges` |

**`ShipmentExecutesTransfer` has no seed data.** The container, view, and model are deployed, but `isc_seed_data_ingestion` ships no transformation for it — `scheduled_transfer.Table.csv` and `stock_transfer_order.Table.csv` are header-only. It'll show up empty in Fusion until a tenant adds its own transfer data.

Three of the edges — reservation (`MaterialReservation`), notification links (`NotificationLinksEntity`), and transportation-lane usage (`UsesTransportationLane`) — are in `isc_connect_edges`, the replaceable reference module, not `isc_core_edges`. A tenant that already tracks these relationships elsewhere replaces this module rather than editing it; see [page 6](../isc-deployment-and-customization/6-customize.md).

Every edge container is `usedFor: edge` and carries the same relationship properties on top of the standard source-tracking set: `relationType`, `relationStatus`, `validFrom`, `validTo` — plus entity-specific fields:

| Edge view | Extra properties beyond the common pack |
|---|---|
| `SupplierSuppliesMaterial` | `unitCost`, `currency`, `priorityRank`, `contractRef`, `minQuantity`, `maxQuantity`, `otifScore` (+ `otifScorePeriodStart`/`End`) |
| `LaneTransportsMaterial` | `priorityRank`, `minLotSize`/`maxLotSize`/`lotSizeUnit`, `costPerTonne`/`costCurrency`, `hazardousGoodsApproved`, `customsClearanceRequired` |
| `LotStoredInBin` | `qtyOnHand`, `qtyBlocked`, `lastUpdated`, `sourceSystem`, `linkLastUpdatedTime` |
| `ShipmentExecutesReceipt`, `ShipmentExecutesTransfer` | `quantity`/`quantityUnit`/`quantityType`/`uomConversionFactor`, ASN fields (`asnNumber`/`asnQty`/`asnDate`), BoL fields (`bolNumber`/`bolQty`/`bolDate`), `actualQty`, `receivedDate`, `actualDepartureDate`/`actualPickupDate`/`actualArrivalDate`, `goodsReceiptDocument`, `discrepancyQty`/`discrepancyReason`, `damageFlag`, `qualityHoldFlag`, `executionStatus`, direct relation to `material` — identical on both |
| `MaterialReservation` | `reservationNumber`, `quantityReserved`, `quantityUnit`, `reservationDate`, `requirementDate`, `sourceSystem`, `linkLastUpdatedTime` |
| `NotificationLinksEntity` | `linkRole`, `severityAtLinkTime`, `correlationId`, `impactCategory` |
| `UsesTransportationLane` | `plannedTransitDays`/`actualTransitDays`, `delayReason`, `actualCost`/`costCurrency`, `customsClearanceRequired`, `legSequence`/`legRole` |

Which module owns which relation source, and how to rewire an `isc_connect_edges` relationship to point at an existing system of record, is covered in [page 6](../isc-deployment-and-customization/6-customize.md) — not repeated here.

## Who reads these models

These identifiers are bound in two places, both outside this module tree:

- The 3 Atlas agents each query via a `Query` tool scoped to location filter `loc_energy_bridge_supply_chain` (`providedAtRuntime` data models/instance spaces, not an inline baked-in list) — see [Atlas agents](atlas-agents-and-skills.md).
- `fn_btp_hierarchy` — the one Function that exists — takes space and view identifiers as runtime input on its workflow trigger (`view_space`, `platform_space`, `instance_spaces`, etc.), not generated constants baked in at build time — see [Functions and workflows](functions-and-workflows.md).

Renaming a space, data model, or view here can still break the location filter, trigger inputs, or transformations that reference the old identifier, so check those before renaming.

## Where to look

In Fusion: **Data models** for the 8 data models and their views, **Spaces** for the 12 spaces, **Containers** (under a space) for the underlying properties and indexes.
