# NewHireOnboarding

Workato "New Hire Onboarding" project. This git repo is the **source of truth**; Workato is the execution/runtime plane. Assets are managed as JSON via the `wk` CLI — not authored through the Workato UI, RCLM, or the AIRO MCP recipe builder.

This file is the index. Topic detail lives in `docs/`:

- [docs/setup.md](docs/setup.md) — layout, auth profile, connections, live endpoint
- [docs/gotchas.md](docs/gotchas.md) — sharp edges hit while integrating with `wk` / the API gateway
- [docs/deployment-pipeline.md](docs/deployment-pipeline.md) — CI/CD design discussion (status: undecided)
