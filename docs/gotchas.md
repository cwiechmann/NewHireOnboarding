# Gotchas

## API gateway auth header

Auth header is **`api-token: <token>`** — not `Authorization: Bearer`. Sending the wrong header/scheme returns a generic `{"error":"access to this API has been disallowed"}` 401 that looks identical to an IP-allowlist rejection. Confirmed by probing the same gateway with no auth, wrong method, and a nonexistent path — all returned the identical error, which is what proved it wasn't IP-based. Don't waste time chasing the IP-allowlist theory before checking the header name.

Reference: https://docs.workato.com/en/api-mgmt/auth-token.html

## `wk pull` / `wk push` token privilege

Need an API token with the "Projects" (package export) privilege. A token that works fine for `/api/recipes` can still 401 on `/api/export_manifests` — this is a separate scope, not a general auth failure.

## Salesforce lookup uses the `User` sobject

The recipe searches Salesforce `User` (internal login users/employees), not `Contact` or `Lead`. Testing with an arbitrary email that only exists as a Contact/Lead will correctly 404 — that's expected behavior, not a bug.

## `wk push` silently no-ops without `--force` on a first deploy to a new environment

`wk push` and `wk push --dry-run` both reported all files as `"unchanged"` (empty `import_results`, exit `0`) against a target workspace that had never received the project — confirmed nothing was actually uploaded (checked via `wk recipes list` on the target folder). The local sync-state cache tracks content hashes from the last push, not live per-workspace state, so it isn't aware the target environment is empty. `wk push --force` uploads regardless of cached status and is required for any first-time or cross-environment deploy. A CI job must always use `--force` for deploys — checking `exit code == 0` alone is not sufficient evidence that anything was pushed.

## `-p <profile>` doesn't fully re-scope folder resolution

`wk diff -p <other-profile>` (and likely other path-resolving commands) authenticates against the overridden profile but can still resolve folder paths against the *project-linked* profile/workspace recorded in `.wk/wk.toml` — it errored `resolving folder "X" in workspace "<linked-workspace>"` even with `-p` pointing elsewhere. Use `wk link --profile <name>` followed by `wk auth switch <name>` to fully retarget a project at a different environment; don't rely on `-p` alone for anything beyond a one-off auth override on commands that don't touch folder paths.

## Connections with no name match import as `null`, not a stub

If a recipe references a connection by name (`config[].account_id.name`) that doesn't exist in the target workspace, `wk push` imports the recipe with `account_id: null` for that provider. It does **not** auto-create an empty placeholder/stub connection. The recipe deploys inactive/disconnected; the named connection must be created in the target workspace (matching folder + name) separately, then re-pushed with `--force` to bind.

## API endpoints require manual activation after push

A freshly pushed API/recipe is not automatically started. Calling its live endpoint before activation returns the same generic `{"error":"access to this API has been disallowed"}` 401 described above (auth-header gotcha) — indistinguishable from an auth failure until you check whether the API has been manually activated in the target workspace.

## Slack bot must already be a member of a pre-existing target channel

The recipe's "find or create channel" logic (`conversations.create`, falling back to `conversations.list` if the channel already exists) doesn't verify or re-establish channel membership on the fallback path. If the channel already exists and the bot user isn't a member, the subsequent `conversations.invite` / `chat.postMessage` calls fail with a raw Slack `not_in_channel` error, surfaced as a 500 from the recipe's global catch handler. Fix by manually adding the bot to the existing channel (Slack bots auto-join channels they create themselves, so this only bites when the channel pre-existed).

## `wk recipes delete` mutates the locally tracked project regardless of your current directory

Deleting a recipe from a workspace (`wk recipes delete <id>`) also deletes the matching local `.recipe.json`/`.meta.json` in whichever local wk project last synced that recipe — confirmed this happened even when the command was run from a completely unrelated directory (a scratch folder outside the repo), not the project tree. wk appears to track the recipe↔local-file association globally (by workspace + recipe name), not by cwd. There is no flag to do a server-only delete while skipping local cleanup. If the local file is git-tracked and clean, `git restore <file>` recovers it immediately; if it has uncommitted local edits, they'd be lost. Confirmed connection-rebinding still works correctly after a full delete+recreate: re-pushing after a delete creates a brand-new recipe ID, and it re-binds to existing same-named connections automatically (see deployment-pipeline.md).
