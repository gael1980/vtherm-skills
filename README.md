# VTherm Skills

Agent skills for setting up, understanding, diagnosing and controlling [Versatile Thermostat](https://github.com/jmcollin78/versatile_thermostat). Get documentation-based guidance without a connection, or use **Home Assistant’s official MCP server, ha-mcp, or both** for supported live operations.

This community package contains instructions and tool references. Each user connects their own Home Assistant instance separately. It does not include a custom MCP server, credentials, or thermostat data.

## What it helps with

- Guided setup: identify equipment, choose the thermostat type, build a confirmed plan, walk through Home Assistant screens and verify the result. Beginners proceed one step at a time; experienced users can go straight to technical settings. See [guided setup](skills/vtherm-mcp/references/guided-setup.md).
- Everyday use: understand presets, manual targets, schedules, absence, cooling and central-boiler behavior.
- Troubleshooting: interpret VTherm reasons, distinguish missing tools from denied access and prepare a redacted support report.
- Comfort and energy: review recorded behavior while distinguishing measured consumption from estimates.
- Complements: consult the current [official catalog](https://www.versatile-thermostat.org/fr/plugins/) and relevant project documentation to explain suitable extensions for the user's needs, without maintaining a copied plugin list or assuming compatibility.
- Live operations: inspect, diagnose, read collector logs and linked-plugin diagnostics, chart history and execute supported explicit climate/TPI changes when the required tools are available.

Answers follow the user's language, including French, English and German. See the [user workflows](skills/vtherm-mcp/references/user-guide.md). Installation/configuration advice and automation drafts do not execute changes. Timed presets, schedules, preset-temperature edits and central selectors are documented guidance; they are outside this skill's execution workflows. A timed request is never replaced by an indefinite immediate change.

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

Connect your agent to your own Home Assistant using one of these configurations. Installing the skill does not create a connection or grant permission.

| Configuration | Skill behavior |
| --- | --- |
| No MCP connection | Explains features, guides setup and interprets user-supplied evidence using official documentation; cannot inspect or change the live installation. |
| Official MCP only | Uses dedicated native VTherm tools available in the catalog. |
| [ha-mcp](https://github.com/homeassistant-ai/ha-mcp) only | Discovers VTherms, reads state/reasons, creates recorded charts/CSV and uses standard climate services for explicit control. Native TPI analysis/application is unavailable without dedicated tools. |
| Both | Prefers dedicated VTherm tools per operation; uses supported ha-mcp routes for absent capabilities. Never duplicates writes or bypasses a refusal. |

For the official route, install a VTherm/HA version supporting the desired tools, configure the [official MCP Server integration](https://www.home-assistant.io/integrations/mcp_server/), expose the intended thermostats to Assist, and check for tools ending in `versatile_thermostat__list_vtherms`, `versatile_thermostat__get_status` and `versatile_thermostat__diagnose`. Prefixes vary.

For ha-mcp, follow its own connection documentation and discover the installed catalog. The skill uses generic state/history tools and standard climate services; it does not assume native Assist tools are relayed. See [the ha-mcp workflow](skills/vtherm-mcp/references/ha-mcp.md) for version/schema checks and limits. ha-mcp policies and credentials differ from native Assist exposure and user-context checks.

Use the chosen server's read-only settings for diagnostic access. A hidden/denied operation must not be rerouted to another server. In mixed setups, keep instance identity and target consistent; identical entity IDs can exist in different homes. Workspace restrictions still take precedence.

## Native feature availability

PR #2232/#2233 status checked on **12 November 2026**; release 10.6.4 contents below were verified on **8 November 2026**. Verify the installed version and actual connector catalog rather than assuming all tools below are available.

| Workflow | Required tools | Upstream status at this check |
| --- | --- | --- |
| List, inspect, diagnose | `list_vtherms`, `get_status`, `diagnose` | Native integration merged in [PR #2160](https://github.com/jmcollin78/versatile_thermostat/pull/2160); use a release containing it |
| Set target, preset, HVAC mode | `set_temperature`, `set_preset`, `set_hvac_mode` | Same native integration; explicit user request and control permission required |
| Recorder history, charts, CSV | `get_history` | [PR #2167](https://github.com/jmcollin78/versatile_thermostat/pull/2167) merged into official `main` on 1 November 2026 |
| Explore classic TPI coefficients | `check_tpi_optimization`, `analyze_tpi_coefficients` | Same merged PR #2167; estimates remain exploratory |
| Read collector logs and linked-plugin diagnostics | `get_logs`, `get_plugins` | [PR #2209](https://github.com/jmcollin78/versatile_thermostat/pull/2209) merged on 8 November 2026 and verified in VTherm 10.6.4 |
| Central configuration and per-thermostat inheritance | Existing `list_vtherms`, `get_status`, `diagnose`; extended response fields | [PR #2232](https://github.com/jmcollin78/versatile_thermostat/pull/2232) open, unmerged; release/installed availability not established |
| Experimental valve TPI analysis and verification notices for switches/valves | Existing `check_tpi_optimization`, `analyze_tpi_coefficients`; extended eligibility/results | [PR #2233](https://github.com/jmcollin78/versatile_thermostat/pull/2233) open, unmerged; release/installed availability not established |
| Apply specified TPI coefficients | `set_tpi_coefficients` | Same merged PR #2167; explicit user request and control permission required |

A merged PR does not by itself establish that a published release contains it. Missing tools must be reported; the skill must not bypass a refusal using generic services or another server. A connector may need its tool catalog refreshed after an integration update.

The published release [10.6.4](https://github.com/jmcollin78/versatile_thermostat/releases/tag/10.6.4) contains both sets of tools. The two diagnostics tools target loaded VTherms exposed to Assist. Collector logs may be truncated or incomplete, and plugin discovery and state depend on the available evidence. Verify both tools in the actual MCP catalog.

TPI estimates remain exploratory and do not establish optimal settings. Analysis supports classic TPI over physical switches, needs at least seven usable days within the last fourteen days, and does not apply coefficients. SmartPI and Auto-TPI are unsupported. PR #2233 proposes experimental `over_climate_valve` analysis from recorded opening commands (a proxy, not measured heat); plain `over_climate` and `over_valve` remain unsupported. This does not establish release or live installation availability. For both switches and valve-regulated climates, explicitly verify experimental estimates before application. Graphs require recorded climate history and client-side chart creation; HVAC action is not physical relay duty.

PR #2232 proposes adding `central_configuration` to native list/status/diagnosis results, including per-thermostat inheritance and effective settings. It requires authenticated READ permission for loaded, exposed targets and exports reviewed fields only. The PR is open and unmerged; its presence in a release or installation is not established. Older servers retain the partial visibility described above. The two PRs extend existing tools: their names alone do not prove the new response fields or valve support are available.

## Example prompts

| Français | English |
| --- | --- |
| J'ai des vannes avec des entités climate et number : quel type choisir ? | My TRVs expose climate and number entities. Which type should I choose? |
| Comment configurer des horaires et gérer mes absences ? | How do I set up schedules and absence behavior? |
| Aide-moi à comprendre ma consommation sans changer les réglages. | Help me understand consumption without changing settings. |
| Quels VTherm sont disponibles ? | List the available VTherms. |
| Pourquoi le thermostat du salon ne chauffe pas ? | Why is the living-room thermostat idle? |
| Consulte les journaux du salon depuis une heure et identifie les plugins qui lui sont liés. | Check the living-room logs from the past hour and identify its linked plugins. |
| Mets le thermostat du salon à 19,5 °C. | Set the living-room thermostat to 19.5 °C. |
| Fais un graphe annoté du thermostat cuisine sur 48 h. | Make an annotated chart of the kitchen thermostat over 48 hours. |
| Les coefficients du thermostat du bureau sont-ils bien réglés ? | Are the office thermostat coefficients well tuned? |

Charts use native history or the supported ha-mcp history route. TPI analysis requires the dedicated tools described above. Clients supporting explicit Codex skill invocation can prefix a request with `$vtherm-mcp`.

## Control and diagnostics

Diagnostic requests remain read-only. Writes require an explicit user request and a clear target. Coefficient application requires the native tool; large TPI changes require its additional confirmation workflow. The skill does not replace it with a generic TPI service call. The skill prioritizes VTherm’s own reasons, preserves entity IDs and enum values, and never invents missing history or uploads logs without authorization.

Configuration reviews and diagnoses explicitly report central-configuration visibility and each inspected thermostat's confirmed inheritance, central-mode participation and central-boiler participation. Relevant inherited values are included when accessible. Older entity-state tools may expose effective settings without their central/local origin or the full central entry; the skill reports these gaps rather than assuming no central configuration exists.

Every substantive VTherm answer links to the relevant verified page of the [official documentation](https://www.versatile-thermostat.org/), or its homepage when the exact page is unverified. For uncertain feature/configuration questions, the skill consults the current documentation when browsing is available. If it cannot consult the documentation or establish the answer, it states that limit. A reading suggestion is distinguished from a consulted source; documented features are distinguished from the installed version and live evidence.

See [the skill](skills/vtherm-mcp/SKILL.md), [tool reference](skills/vtherm-mcp/references/tool-reference.md), [diagnostic reference](skills/vtherm-mcp/references/diagnostic-reference.md), and [ha-mcp workflow](skills/vtherm-mcp/references/ha-mcp.md).

## skills.sh directory

The CLI discovers this package through `skills/vtherm-mcp/SKILL.md`. According to the [skills.sh FAQ](https://skills.sh/docs/faq), directory ranking is populated automatically from anonymous installation telemetry when users install a GitHub repository with `npx skills add`. Publishing the source alone does not guarantee immediate listing; no indexing deadline is specified. Local validation here disables telemetry and does not establish public listing.

## Pour les utilisateurs francophones

Demandez « Je viens d’installer VTherm, aide-moi à configurer le chauffage de ma cuisine ». Le [parcours guidé](skills/vtherm-mcp/references/guided-setup.md) avance écran par écran : matériel, capteurs, type, héritage central, régulation, presets et options, puis vérification. Vous pouvez demander pourquoi un paramètre est utile, passer à la suite ou aller directement aux réglages techniques. Vous effectuez la création dans Home Assistant ; la découverte MCP reste en lecture seule.

Pour un besoin complémentaire (interface, programmation, matériel ou régulation), la skill consulte le [catalogue officiel des compléments](https://www.versatile-thermostat.org/fr/plugins/), puis la documentation des projets pertinents. Elle explique leur utilité et les prérequis vérifiés selon votre besoin, sans conserver une liste figée ni installer automatiquement un complément. Si elle ne peut pas vérifier les sources, elle le précise et vous renvoie vers le catalogue.

Sans connexion MCP, la skill peut déjà vous aider à choisir le type de VTherm, préparer la configuration, comprendre les presets, horaires, absences et fonctions centrales, ou préparer un dossier de dépannage à partir des informations fournies. Elle répond dans votre langue et s'appuie sur la documentation officielle. Les conseils et brouillons d'automatisation ne modifient pas votre installation. Les commandes temporaires, horaires et modifications de configuration restent des parcours guidés ; une demande limitée dans le temps ne devient jamais une commande immédiate sans retour automatique.

Installez la skill avec la commande ci-dessus, puis connectez votre agent au MCP officiel Home Assistant, à **ha-mcp seul**, ou aux **deux**. Avec les deux, les outils VTherm dédiés sont prioritaires pour chaque opération disponible. Avec ha-mcp seul, la skill utilise les états et raisons exposés, l'historique pour les graphiques/CSV et les services climate pour les commandes explicites. Le diagnostic reste en lecture seule ; les refus et restrictions ne sont jamais contournés par l'autre serveur.

L'analyse et l'application des coefficients TPI nécessitent les outils natifs dédiés ; un appel de service générique ne reproduit pas leur procédure de confirmation. Ces outils ont été fusionnés dans le `main` officiel avec la PR #2167 le 1er novembre 2026. Vérifiez la version installée et le catalogue réel : la fusion ne prouve pas leur disponibilité. Les estimations restent exploratoires et ne prouvent pas un réglage optimal.

Avec VTherm 10.6.4 et les outils natifs correspondants dans le catalogue MCP, la skill peut lire les journaux du collecteur et les diagnostics publiés par les plugins liés au thermostat. Elle signale la troncature et les états de plugins inconnus.

Au 12 novembre 2026, les PR [#2232](https://github.com/jmcollin78/versatile_thermostat/pull/2232) (contexte central) et [#2233](https://github.com/jmcollin78/versatile_thermostat/pull/2233) (analyse TPI expérimentale `over_climate_valve` et avertissements pour switches/vannes) sont ouvertes, non fusionnées. Elles enrichissent les outils existants ; leurs noms ne suffisent pas à prouver ces nouvelles capacités. Pour les vannes, la commande enregistrée est un proxy, pas une mesure de chaleur. Toute estimation reste expérimentale, à vérifier avant application, y compris pour `over_switch`. La disponibilité en release ou dans votre installation reste à vérifier.

Les revues de configuration et les diagnostics indiquent la visibilité de la configuration centrale et, pour chaque VTherm examiné, les réglages hérités vérifiables, la participation au mode central et à la chaudière centrale. Les valeurs centrales pertinentes sont incluses lorsqu'elles sont accessibles. Si les outils exposent uniquement les réglages effectifs sans leur origine ou sans la configuration centrale complète, cette limite est signalée.

Chaque réponse VTherm substantielle renvoie vers la page pertinente vérifiée de la [documentation officielle](https://www.versatile-thermostat.org/), ou vers son accueil si la page exacte n'est pas vérifiée. En cas d'incertitude, la skill consulte cette documentation si elle peut naviguer et précise sinon cette limite. Un lien proposé à lire ne vaut pas consultation ; la documentation reste distincte de l'état réel de votre installation.

La compatibilité décrite repose sur les contrats d'outils et la validation du paquet ; elle ne constitue pas un essai réel des trois configurations sur Home Assistant.

## License and attribution

[MIT](LICENSE). Maintained by `gael1980`. The instructions and references describe the native VTherm integration maintained in `jmcollin78/versatile_thermostat`; the upstream MIT copyright notice is retained. This is a community skill package, not an endorsement by Home Assistant, OpenAI, or Vercel.
