# Gotchas

## API gateway auth header

Auth header is **`api-token: <token>`** — not `Authorization: Bearer`. Sending the wrong header/scheme returns a generic `{"error":"access to this API has been disallowed"}` 401 that looks identical to an IP-allowlist rejection. Confirmed by probing the same gateway with no auth, wrong method, and a nonexistent path — all returned the identical error, which is what proved it wasn't IP-based. Don't waste time chasing the IP-allowlist theory before checking the header name.

Reference: https://docs.workato.com/en/api-mgmt/auth-token.html

## `wk pull` / `wk push` token privilege

Need an API token with the "Projects" (package export) privilege. A token that works fine for `/api/recipes` can still 401 on `/api/export_manifests` — this is a separate scope, not a general auth failure.

## Salesforce lookup uses the `User` sobject

The recipe searches Salesforce `User` (internal login users/employees), not `Contact` or `Lead`. Testing with an arbitrary email that only exists as a Contact/Lead will correctly 404 — that's expected behavior, not a bug.
