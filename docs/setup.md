# Setup

## Layout

- `.wk/wk.toml` — wk project config (local, not committed — see `.wk/.gitignore`). Active auth profile: `eu-chris-wiechmann-dev` (workspace "Chris Wiechmann", region eu).
- `New Hire Onboarding/` — synced wk project folder (server folder id 1589797, project id 1206254):
  - `onboard_new_hire.recipe.json` — the recipe (server flow id 2276523). Flow: API trigger (`email`) → Salesforce `search_sobjects` on `User` by email → create-or-find `#team-welcome` Slack channel → invite user → post welcome message → return JSON (200 success / 404 no user found / 500 on error).
  - `onboard_new_hire.api_endpoint.json` — API endpoint, `POST /onboard_new_hire`.
  - `offboard_employee.recipe.json` / `offboard_employee.api_endpoint.json` — offboarding: posts goodbye in `#team-welcome`, kicks the user, deactivates the Salesforce User, best-effort Slack `admin.users.remove` (needs an admin token; bot tokens fail non-fatally). `POST /offboard_employee`.
  - `employee_lifecycle.api_group.json` — API collection "Employee Lifecycle" (handle kept as `new-hire-onboarding-v1` so the live URL is unchanged).
  - `project.json` — folder metadata.
- Connections (not stored here, referenced by name/folder): "Demo Salesforce account" (salesforce), "Slack My Demo-Bot" (slack_bot), both in the `🔌 Connections` folder.

## Live endpoint

`POST https://apim.eu.workato.com/chrisw/new-hire-onboarding-v1/onboard_new_hire`

`POST https://apim.eu.workato.com/chrisw/new-hire-onboarding-v1/offboard_employee` (not yet pushed)

See [gotchas.md](gotchas.md) for the auth header format.
