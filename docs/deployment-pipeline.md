# Deployment pipeline

**Status: undecided, still under discussion. Nothing here has been implemented.**

## Goal

Git branches/tags map to environments (e.g. dev/staging/prod). Deploys push the exact JSON already in the repo — no rebuild step. Open question whether Workato RCLM is used at all, or whether `wk push` alone (scripted in CI) replaces it.

## Discussion so far

- Leaning discussed: prefer scripted `wk push` per environment (each with its own auth profile) over relying on RCLM's UI-driven promote flow, since the requirement is "deploy JSON as-is" rather than "promote via diff/rebuild."
- Observed (not yet verified against official docs): connection references inside recipe JSON (`config[].account_id`) are encoded as `{folder, name, zip_name}` — matching an existing connection by name/folder, or creating an empty stub connection if no match is found, appears to be a property of Workato's package export/import format itself. This format is shared by RCLM, "copy to folder," and `wk push` alike — it does not appear to be something RCLM uniquely provides. This needs confirming before the pipeline design leans on it, since it directly affects how per-environment connections would be handled.
- Per-environment connection remapping and environment properties still need a concrete plan regardless of which mechanism is used.

## Not yet decided

- Branch/tag model (e.g. `main` → dev, `release/*` or `v*` tags → prod?)
- Whether RCLM plays any role, or is bypassed entirely in favor of `wk push`
- How environment-specific connections and properties get set correctly on each push
- CI tool (GitHub Actions vs Jenkins vs other)
