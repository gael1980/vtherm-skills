# Maintaining VTherm Skills

## Scope and source

This repository publishes reusable agent instructions for the native Versatile Thermostat integration through Home Assistant’s official MCP server. Keep that architecture. Do not introduce a custom VTherm MCP server. The publication source is `skills/vtherm-mcp/`; read its entire `SKILL.md` and relevant references before editing.

In the maintainer’s VTH workspace, `../vtherm-mcp/` is a local companion copy. When present, compare both copies before editing and keep their four skill files identical after an authorized update. Investigate differences; do not overwrite independent local changes. Do not require that sibling directory for users or standalone clones.

## Git synchronization

- Before a new editing session, inspect the repository root, status, branch, remotes and upstream. The publication repository is `gael1980/vtherm-skills`; `origin` points there and `main` normally tracks `origin/main`.
- If clean and tracking the expected branch, run `git pull --ff-only`, then recheck status and HEAD before editing. Also fetch `origin` and compare against `origin/main`; do not merge main automatically into another branch.
- If local changes exist, preserve them. Fetch remote references, report any synchronization issue, and do not pull, stash, reset or commit merely to clean the tree. Explain failures, missing upstreams and divergence without forcing integration. For a new repository, the first authorized push establishes tracking.
- Before staging, committing or pushing, verify the repository root, stage explicit paths and inspect the staged diff. Present the precise change summary, commit message and validation before commit/push. Publication requires user authorization; maintenance instructions alone do not grant it. No force push or destructive Git operations.
- Keep credentials, personal Home Assistant URLs, histories, logs, workspace memory, temporary validation output and `.DS_Store` out of this repository.

## Keep the skill current

For each relevant native VTherm LLM/MCP change, inspect the actual implementation and upstream status. Update `SKILL.md`, tool/diagnostic references, optional UI metadata and README only where affected. If working in VTH, also check the VTherm FR/EN/DE documentation and README files and update workspace memory outside this repository.

Preserve tool identifiers, parameter names, permissions, confirmation flows, refusal semantics, pagination and scientific limitations. Diagnostics remain read-only; real changes require explicit user requests. Never bypass a dedicated tool’s refusal using generic services, another server or indirect APIs. Never present exploratory TPI estimates as validated optimal settings.

Keep README feature availability dated and evidence-based. Distinguish open PRs, merged changes, published releases, installed versions and the client’s actual tool catalog. Do not invent minimum release numbers or assume a merged PR is already released.

## Validation and publication

- Run the available official skill-creator validator when installed. Validate frontmatter and `agents/openai.yaml`, check relative reference links, and compare the companion copy when present.
- When packaging or discovery changes, test `npx skills add <local-repository-path> --list` and a named Codex installation in a temporary project, without changing the maintainer’s installed skills. Use `DISABLE_TELEMETRY=1` for local validation.
- After an authorized push, verify the remote commit and test installation from the GitHub source in a temporary project. Inspect installed files against the committed package.
- Report checks actually run. Format/discovery/install checks do not establish live model routing, Home Assistant behavior or skills.sh listing. Do not call Home Assistant or change thermostat settings as part of packaging validation.
- skills.sh listing is driven by CLI installation telemetry; publication alone does not guarantee immediate indexing. Report observed listing status separately.

These are maintenance instructions for authorized work, not a background synchronization or automatic publishing job.
