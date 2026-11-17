# Bring ISC into the customer repo

*ISC does not get its own repo. It lives inside the customer's complete Cognite Toolkit repository, next to every other module that tenant runs.*

**Before you start:** complete the [prerequisites](./1-overview-and-prerequisites.md), including confirming `cdf auth verify` succeeds in the customer's repo.

## Copy the module tree

Copy the `isc/` folder into the customer repo's `modules/` directory, so it lands at `modules/isc/` — next to the customer's existing modules:

```text
customer-toolkit/
  cdf.toml
  config.dev.yaml
  modules/
    auth/                       # customer's own
    ingest/                     # customer's own
    isc/                        # <- copied in from this offering
      isc_core/                 # product-owned schema and hierarchy Functions — see 6-customize.md for connecting to existing maintenance/manufacturing
      isc_connect_maintenance/  # reference implementation — replace or wire per tenant
      isc_connect_manufacturing/
      isc_edges/
        isc_core_edges/         # fixed
        isc_connect_edges/      # wiring layer
      isc_atlas_agents/         # product-owned: Atlas agents, skills, fn_btp_graph, fn_btp_insight
      isc_archival_records/  # product-owned: records archive, streams, archive triggers
      isc_data_mesh/            # optional: data quality track
      isc_seed_data_ingestion/  # seed data only: copy only to deploy demo/test data, omit in production
```

> [!NOTE]
> **`isc_seed_data_ingestion` is optional to copy.** Copy it into the customer repo only when the demo seed data has to be deployed, for example in a test, demo or sandbox project. It contains seed data only (RAW tables, transformations and `wf_btp_seed_data_ingest`) and no schema, functions or agents. A production tenant that loads its own data should leave the folder out of `modules/isc/` and out of `environment.selected`.

See [the rule that decides where changes go](./1-overview-and-prerequisites.md#the-rule-that-decides-where-changes-go) for what's fixed and what's yours to edit in this tree.

> [!WARNING]
> Stop here if the customer repo already has a `modules/isc/` directory that doesn't match this offering — a prior manual copy, or an older release. Diff it first. Overwriting it wholesale destroys any customization already made to `isc_connect_*`.

## Merge `cdf.toml`

Merge this offering's `cdf.toml` into the customer's — don't replace their file, it has their own settings. Keep `[modules] version` aligned with the version this offering ships: see the shipped [`cdf.toml`](../../cdf.toml), don't hardcode the number here.

As part of that merge, copy in the `[alpha_flags]` block (`agents = true`, `agent-skills = true`). ISC ships Agent and Skill YAML under `modules/isc/isc_atlas_agents/agents/`, and Toolkit only recognizes those resource kinds when this block is present. Omitting it shows up as **unknown resource kind** on `cdf build`.

This is a Toolkit CLI setting in `cdf.toml` — not a feature you toggle in Fusion.

## Check it worked

`modules/isc/` now holds the modules below, each with its own `module.toml`: `isc_core`, `isc_connect_maintenance`, `isc_connect_manufacturing`, `isc_edges/isc_core_edges`, `isc_edges/isc_connect_edges`, `isc_atlas_agents`, `isc_archival_records`, `isc_data_mesh`, and `isc_seed_data_ingestion`. The last one is present only if you copied it for seed or test data, so a production repo has eight.

## What's next

- [Configure `config.<env>.yaml`](./3-configure.md)
