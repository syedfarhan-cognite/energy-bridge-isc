# Build, deploy, and verify

*Build and deploy the ISC modules, then check Fusion.*

**Before you start:** Pages [1](./1-overview-and-prerequisites.md), [2](./2-customer-repo.md), and [3](./3-configure.md) are done.

Run every command from the customer's Toolkit root — not from a folder whose parent has another `.env`. Toolkit searches parent folders for one, and can pick up the wrong project.

## 1. Build

```text
cdf build
```

## 2. Dry run

```text
cdf deploy --dry-run
```

Check the output:

- On a first install, it must plan changes **to** `isc_core`
- Test the output of the dry run and check that it contains all the modules we intend to deploy.

If either is wrong, stop and recheck `selected` and the config — don't proceed to a real deploy.

## 3. Deploy

```text
cdf deploy
```

For prod:

```text
cdf build -e prod
cdf deploy --dry-run -e prod
cdf deploy -e prod
```

## 4. Check Fusion

Counts below are for this release — if they don't match, re-check `selected` and the config before assuming something is broken.

| Resource | Expected |
|---|---|
| Spaces, containers, views | 13, 72, and 75 |
| Data models | 11 (10 domain models and `isc_typed_model`) |
| Transformations | 54 |
| Functions | 5, all **Ready** (this can take a few minutes after the deploy ends) |
| Workflows | 4 (see [Functions and workflows](../isc-architecture/functions-and-workflows.md) for which) |
| Workflow triggers | 14 created, none skipped (4 hierarchy path, 10 records archive) |
| Atlas | 3 agents and 4 skills |
| Archive streams | 2 |
| Location filter | `isc_location_filter` |

In Fusion: **Functions** for the 5 Ready; **Workflows** for trigger status; **Atlas** for the agent list.

### Check by module

| Module | What to confirm |
|---|---|
| `isc_core` | Schema spaces and the 11 data models exist. `fn_btp_hierarchy` is Ready. 4 hierarchy path triggers are created. |
| `isc_atlas_agents` | 3 agents and 4 skills are listed in Atlas. `fn_btp_graph` and `fn_btp_insight` are Ready. |
| `isc_archival_records` | Records space `sp_supply_chain_records` exists. `fn_btp_records_archive` is Ready. 10 archive triggers are created. The 2 archive streams exist. |
| `isc_seed_data_ingestion` (demo only) | `wf_btp_seed_data_ingest` is listed in Workflows, with the seed RAW tables and transformations. |

## You're done when

- `cdf deploy` finished without errors.
- All 5 Functions are **Ready**.
- Workflow triggers are **created**, not skipped.
- **Integrated Supply Chain (Main)** is listed in Atlas.
- The seed workflow has completed ([page 5](./5-load-seed-data.md)) — or was deliberately skipped.

## If it fails

| Symptom | Cause | Fix |
|---|---|---|
| "Unknown resource kind" on build | `[alpha_flags]` missing from `cdf.toml` | Add the block ([page 2](./2-customer-repo.md)). |
| A trigger is skipped, or fails with `AADSTS700016` | A trigger client ID is unset | Set it in `.env` ([page 1](./1-overview-and-prerequisites.md)). |
| A Function is not Ready after about 5 minutes | Functions not activated, or a build error | Check activation, then open the Function's logs. |
| Dry run shows deletions outside `modules/isc/*` | `selected` replaced the customer's list | Restore their modules and rerun. |

## What's next

Wait until all five Functions show **Ready** before you run the seed workflow — starting it earlier is the most common mistake at this step.

- [Load the seed data](./5-load-seed-data.md)
