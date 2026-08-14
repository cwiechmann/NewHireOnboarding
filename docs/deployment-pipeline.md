# Deployment pipeline

**Status: partially decided. CI tool and core push mechanics are confirmed by a live test (2026-08-14). Branch/tag model and full CI workflow are not yet implemented.**

## Goal

Git branches/tags map to environments (e.g. dev/staging/prod). Deploys push the exact JSON already in the repo — no rebuild step. Confirmed: `wk push`, scripted per environment, replaces RCLM for this — RCLM is bypassed entirely.

## Decided

- **CI tool: GitHub Actions.**
- **Environments are separate Workato workspaces**, not one workspace with environment tags. Confirmed via distinct numeric workspace IDs (dev `754943` "Chris Wiechmann" vs prod `756625` "Environment Production") and distinct owner emails. Each environment needs its own `wk` auth profile and its own GitHub encrypted secret for CI.
- **Mechanism: scripted `wk push --force` per environment**, each with its own auth profile, over RCLM's UI-driven promote flow — confirmed working end-to-end against a live prod-equivalent workspace on 2026-08-14 (push → connection binding → API activation → live invocation all succeeded).

## Confirmed via live test (2026-08-14)

- **`--force` is required for any first deploy to a new environment.** `wk push` (and `wk push --dry-run`) both report files as `"unchanged"` / empty `import_results` and exit `0` even when the target environment has never received the project — the local change-tracking cache is keyed off content hashes from the last push, not live per-workspace state. A CI job that checks exit code alone would report "deployed" while uploading nothing. **Always pass `--force` for cross-environment/CI deploys.**
- **Connections that don't match by name import as `account_id: null` on the recipe — not as an auto-created stub connection.** This corrects the earlier assumption below. A recipe pushed to a workspace with no matching connection name arrives disconnected (and inactive), not wired to a placeholder. Per-environment connections (same `name` + folder, e.g. `🔌 Connections`, differing only by which workspace/credentials they point to) must already exist in the target workspace *before* the push for the recipe to bind and be runnable.
- **API endpoints are not auto-activated on push, but activation is fully scriptable — no UI step required.** After `wk push`, both the recipe and its API endpoint start disabled; the live endpoint 401s (`access to this API has been disallowed` — same symptom as an auth/IP issue, see gotchas.md) until both are activated: `wk recipes start <recipe_id>` and `wk api endpoints enable <endpoint_id>` (find the endpoint id via `wk api endpoints list`, matched by `recipe_id`). Confirmed both steps are required — starting the recipe alone was not sufficient, the endpoint also had to be enabled separately.
- **`-p <profile>` override does not fully re-scope folder/path resolution for commands like `wk diff`** — it authenticates against the overridden profile but folder-path resolution can still anchor to the project's *linked* profile/workspace (recorded in `.wk/wk.toml`), causing misleading errors that name the wrong workspace. Use `wk link --profile <name>` + `wk auth switch <name>` to fully retarget a project to a different environment, rather than relying on `-p` alone for anything beyond simple auth overrides.
- **Connection remapping by name survives a full recipe delete + redeploy.** Deleted the recipe from prod (leaving its already-created, same-named Salesforce/Slack connections in place), then re-ran `wk push --force`: the recreated recipe (a brand-new recipe ID) automatically bound to the existing connections by name, with no manual rewiring. Confirms the promotion model is viable: provision per-environment connections once (by name), and every subsequent push/redeploy — including a full teardown — re-binds correctly.

## Still not yet decided / implemented

- Branch/tag model (e.g. `main` → dev, `release/*` or `v*` tags → prod?)
- The actual GitHub Actions workflow YAML (secrets per environment, `wk link`/`wk auth switch` + `wk push --force --no-create`, `wk recipes start` + `wk api endpoints enable`, prod gated by a required-reviewer GitHub Environment)
- How environment-specific connections get created/authorized in each target workspace before first deploy (currently manual, via UI — pushing doesn't create or authorize them)
- Environment properties (workspace-level config vars) — no plan yet
