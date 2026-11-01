# VTherm Skills

Agent skills for diagnosing and controlling [Versatile Thermostat](https://github.com/jmcollin78/versatile_thermostat) through **Home Assistant’s official MCP server**.

This community package contains instructions and tool references. Each user connects their own Home Assistant instance separately. It does not include a custom MCP server, credentials, or thermostat data.

## Install

Install from `gael1980/vtherm-skills`:

```bash
npx skills add gael1980/vtherm-skills --skill vtherm-mcp
```

Choose your agent when prompted, or target Codex directly:

```bash
npx skills add gael1980/vtherm-skills --skill vtherm-mcp --agent codex
```

Installation is scoped to the current project by default. Add `--global` for installation across projects. The current CLI requires Node.js 22.20 or later. See the [Skills CLI documentation](https://github.com/vercel-labs/skills).

For local development and testing:

```bash
npx skills add ./vtherm-skills --list
npx skills add ./vtherm-skills --skill vtherm-mcp --agent codex
```

Run those local commands from the directory containing `vtherm-skills`, or use an absolute path. Agent-specific metadata in `agents/openai.yaml` is optional; the workflow is defined in `SKILL.md`. Installation compatibility does not establish live tool availability in every agent.

## Connect Home Assistant

1. Install a VTherm version containing the native LLM tools you want to use, and a compatible Home Assistant version. No minimum release number is asserted here; verify the tool catalog after setup.
2. Configure the [official Home Assistant MCP Server integration](https://www.home-assistant.io/integrations/mcp_server/) and connect your agent using your own instance URL and authentication.
3. Expose the intended VTherm entities to Assist. Tool availability and entity permissions are enforced by Home Assistant.
4. Check that tools ending in `versatile_thermostat__list_vtherms`, `versatile_thermostat__get_status`, and `versatile_thermostat__diagnose` are visible. Client prefixes vary.

Use the server’s **Read only** option for diagnostic access. Control requires write tools and permission to control the entity. Installing this skill does not create or authorize a connection.

## Feature availability

Availability checked on **1 November 2026**. Verify the installed version and actual connector catalog rather than assuming all tools below are available.

| Workflow | Required tools | Upstream status at this check |
| --- | --- | --- |
| List, inspect, diagnose | `list_vtherms`, `get_status`, `diagnose` | Native integration merged in [PR #2160](https://github.com/jmcollin78/versatile_thermostat/pull/2160); use a release containing it |
| Set target, preset, HVAC mode | `set_temperature`, `set_preset`, `set_hvac_mode` | Same native integration; explicit user request and control permission required |
| Recorder history, charts, CSV | `get_history` | [PR #2167](https://github.com/jmcollin78/versatile_thermostat/pull/2167) merged into official `main` on 1 November 2026 |
| Explore classic TPI coefficients | `check_tpi_optimization`, `analyze_tpi_coefficients` | Same merged PR #2167; estimates remain exploratory |
| Apply specified TPI coefficients | `set_tpi_coefficients` | Same merged PR #2167; explicit user request and control permission required |

A merged PR does not by itself establish that a published release contains it. Missing tools must be reported; the skill must not bypass a refusal using generic services or another server. A connector may need its tool catalog refreshed after an integration update.

PR #2167 was merged at 13:58 UTC in commit [`48271bd`](https://github.com/jmcollin78/versatile_thermostat/commit/48271bdbf094a469900ed40b534c5c41e9d9709b). At this check, the latest published release, [10.6.3](https://github.com/jmcollin78/versatile_thermostat/releases/tag/10.6.3), does not contain these additional tools. Install a VTherm version containing that change and verify the actual MCP catalog before using them.

TPI estimates remain exploratory and do not establish optimal settings. Analysis supports classic TPI over physical switches, needs at least seven usable days within the last fourteen days, and does not apply coefficients. Climate/valve analysis, SmartPI and Auto-TPI are unsupported by that analysis. Graphs require recorded climate history and client-side chart creation; HVAC action is not physical relay duty.

## Example prompts

| Français | English |
| --- | --- |
| Quels VTherm sont disponibles ? | List the available VTherms. |
| Pourquoi le thermostat du salon ne chauffe pas ? | Why is the living-room thermostat idle? |
| Mets le thermostat du salon à 19,5 °C. | Set the living-room thermostat to 19.5 °C. |
| Fais un graphe annoté du thermostat cuisine sur 48 h. | Make an annotated chart of the kitchen thermostat over 48 hours. |
| Les coefficients du thermostat du bureau sont-ils bien réglés ? | Are the office thermostat coefficients well tuned? |

The last two examples require the additional tools described above. Clients supporting explicit Codex skill invocation can prefix a request with `$vtherm-mcp`.

## Control and diagnostics

Diagnostic requests remain read-only. Writes require an explicit user request and a clear target. Large TPI changes require the tool’s additional confirmation workflow. The skill prioritizes VTherm’s own reasons, preserves entity IDs and enum values, and never invents missing history or uploads logs without authorization.

See [the skill](skills/vtherm-mcp/SKILL.md), [tool reference](skills/vtherm-mcp/references/tool-reference.md), and [diagnostic reference](skills/vtherm-mcp/references/diagnostic-reference.md).

## skills.sh directory

The CLI discovers this package through `skills/vtherm-mcp/SKILL.md`. According to the [skills.sh FAQ](https://skills.sh/docs/faq), directory ranking is populated automatically from anonymous installation telemetry when users install a GitHub repository with `npx skills add`. Publishing the source alone does not guarantee immediate listing; no indexing deadline is specified. Local validation here disables telemetry and does not establish public listing.

## Pour les utilisateurs francophones

Installez la skill avec la commande ci-dessus, puis connectez votre agent à votre propre serveur MCP officiel Home Assistant et exposez les thermostats à Assist. Les demandes en français sont prises en charge. Le diagnostic reste en lecture seule ; toute modification nécessite une demande explicite. Les fonctions de graphe, d’historique et d’analyse TPI ont été fusionnées dans le `main` officiel avec la PR #2167 le 1er novembre 2026. À cette vérification, la dernière release publiée, 10.6.3, ne les contient pas encore : utilisez une version intégrant cette évolution, puis vérifiez les outils visibles dans votre connexion MCP. Les estimations TPI sont exploratoires et ne prouvent pas que les coefficients proposés sont optimaux.

## License and attribution

[MIT](LICENSE). Maintained by `gael1980`. The instructions and references describe the native VTherm integration maintained in `jmcollin78/versatile_thermostat`; the upstream MIT copyright notice is retained. This is a community skill package, not an endorsement by Home Assistant, OpenAI, or Vercel.
