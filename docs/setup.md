# Setup

## Layout

- `.wk/wk.toml` — wk project config (local, not committed — see `.wk/.gitignore`). Active auth profile: `eu-chris-wiechmann-dev` (workspace "Chris Wiechmann", region eu).
- `New Hire Onboarding/` — synced wk project folder (server folder id 1589797, project id 1206254):
  - `onboard_new_hire.recipe.json` — the recipe (server flow id 2276523). Flow: API trigger (`email`) → Salesforce `search_sobjects` on `User` by email → create-or-find `#team-welcome` Slack channel → invite user → post welcome message → return JSON (200 success / 404 no user found / 500 on error).
  - `onboard_new_hire.api_endpoint.json` — API endpoint, `POST /onboard_new_hire`.
  - `new_hire_onboarding.api_group.json` — API collection/group config (handle `new-hire-onboarding-v1`).
  - `project.json` — folder metadata.
- Connections (not stored here, referenced by name/folder): "Demo Salesforce account" (salesforce), "Slack My Demo-Bot" (slack_bot), both in the `🔌 Connections` folder.

## Live endpoint

`POST https://apim.eu.workato.com/chrisw/new-hire-onboarding-v1/onboard_new_hire`

See [gotchas.md](gotchas.md) for the auth header format.
