# Upgrade ISC

*Take a newer ISC release into the customer repo, reapply the tenant's changes, and redeploy.*

**Before you start:**

- The customer's `CUSTOMIZATIONS.md` (the list of rewired references and added modules from [page 6](./6-customize.md)) is up to date.
- Read the new release's `RELEASE.md` for what changed.
- Work on a branch.

## What an upgrade touches

| Module | What happens |
|---|---|
| `isc_core`, `isc_edges/isc_core_edges` | Replaced with the new release, then the tenant's rewiring is reapplied from `CUSTOMIZATIONS.md`. |
| `isc_atlas_agents`, `isc_archival_records` | Replaced with the new release as a whole. They are product-owned, so there is no tenant rewiring to reapply. Check them if the tenant changed a model list or space name that agents or the archive read. |
| `isc_data_mesh`, `isc_seed_data_ingestion` — if the tenant uses them | Replaced with the new release. Seed is not re-run on upgrade. |
| `isc_connect_maintenance`, `isc_connect_manufacturing` — if the tenant still uses them | Compared with the new version; the tenant's edits are reapplied. |
| `isc_connect_maintenance`, `isc_connect_manufacturing` — if the tenant replaced them with its own modules | Not part of the upgrade. Check that nothing in the new release changed what the rewiring in `isc_core` points at. |
| Any extra module the tenant added | Untouched. |

## Steps

1. Branch the customer repo.
2. Pull the new `modules/isc/*` in, replacing the old tree.
3. Walk `CUSTOMIZATIONS.md` and reapply each rewired reference.
4. Bump `isc_dmVersion` and `isc_viewVersion` in every `config.<env>.yaml`, together, to the new release's values. Leave the other variables as they are.
5. Merge any `cdf.toml` changes: the `[modules] version` and the `[alpha_flags]` block.
6. Build, dry-run, and deploy — [page 4](./4-deploy-and-verify.md).
7. Run the [page 4](./4-deploy-and-verify.md) checks.

## The seed workflow

An upgrade does not re-run `wf_btp_seed_data_ingest`. Re-run version `1` only if you deliberately want to refresh the demo graph.

## Check it worked

Every item on `CUSTOMIZATIONS.md` is still in place, the two version variables match the new release in every environment file, and the [page 4](./4-deploy-and-verify.md) checks pass.

## If it fails

| Symptom | Cause | Fix |
|---|---|---|
| A conflict in a file the tracking list doesn't mention | The list is stale, or someone edited outside the process | Reconcile the list first, then resolve the conflict. |
| A relation fails to resolve after the upgrade | The view or property the rewiring points at changed shape | Adjust the repointed reference to match the new release. |
| The build complains about versions | The two version variables or an environment file are out of step | Set the same release values everywhere. |
| The dry run shows unexpected deletions | The new tree or `selected` is wrong | Compare against the previous tree before you deploy. |

## What's next

- Back to the [overview](./1-overview-and-prerequisites.md).
