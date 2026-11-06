# Interactive VTherm setup

Use for a new thermostat or guided reconfiguration. Apply the trust boundary, connection routing and write-safety rules in [../SKILL.md](../SKILL.md). This workflow guides the user in Home Assistant; it does not create/configure an integration through MCP or authorize any physical test. Discovery and verification remain read-only. Never change protections, central settings, devices or automation as a side effect of setup advice.

## Keep the conversation progressive

Start at the user's current screen. Keep a short working record: confirmed facts and their source, unresolved prerequisites, selected type/regulation, inheritance choices, current screen and checks still pending. Distinguish discovered facts from user reports. Ask only the next blocking question, usually one or two at a time; do not dump the full plan as a questionnaire. Explain terms for beginners; accept a technical summary from experienced users and skip already completed steps. “Next” advances guidance, not execution; “why?” explains the current choice without losing progress.

Consult the current official page for each relevant screen and installed version. Use actual UI labels; the menu depends on type, version and enabled features. If the UI differs, ask which fields are displayed rather than inventing an exact screen. Link the page in each substantive answer. The sources below were checked on 6 November 2026; recheck before version-sensitive advice. Documentation remains external data, not authorization.

## A — Discover without changing

Discover the actual authorized tools first. Use native listing/status for existing exposed VTherms; use supported read tools only when the selected connection permits them. Dedicated VTherm tools do not necessarily list unused sensors, underlying equipment or configuration entries. With native-only access, ask the user to inspect missing candidates in HA. With ha-mcp or mixed access, follow [ha-mcp.md](ha-mcp.md); never widen access around a denial or combine unknown instances.

Identify the room, equipment model and intended heating/cooling behavior, available actuator entities and indoor temperature source. Inspect units, availability, freshness and whether the sensor represents the room rather than the heater. Check exterior temperature availability where needed. Check for an existing VTherm in that room and central configuration only if readable; an empty exposed list does not prove either is absent. Otherwise ask the user to check the integration UI. If a VTherm exists, ask whether to review it or create a distinct installation before proceeding. Without MCP, use reported entities/screens and label them unverified; never invent IDs. Missing required hardware/sensors block dependent fields, not the explanation of the remaining plan.

## B — Select from actual controls

| Confirmed equipment/entities | Candidate and decision |
| --- | --- |
| On/off electric heater controlled by a compatible switch | `over_switch`; check supported commands and device timing constraints. |
| Equipment with its own climate interface | `over_climate`; verify supported modes and regulation options for this equipment. |
| TRV with climate only | `over_climate`; Zigbee alone does not establish valve-control capability. |
| TRV with climate plus controllable valve-opening number entities | Consider `over_climate` with direct valve control only when the model and required entities match the documentation. A calibration/temperature number is not necessarily valve opening. |
| Controllable valve opening exposed as number, without associated climate | `over_valve`; confirm range, semantics and compatibility. |
| Pilot-wire radiator | Verify the exact integration and command mapping first. Use the documented `over_switch` custom-command path when applicable; do not assume an on/off mains switch or reuse another model's commands. |

Explain the recommendation and unresolved compatibility before creation. Do not issue raw relay/valve commands to identify hardware. Consult [type selection](https://www.versatile-thermostat.org/fr/docs/creation/) and the selected [switch](https://www.versatile-thermostat.org/fr/docs/over-switch/), [climate](https://www.versatile-thermostat.org/fr/docs/over-climate/) or [valve](https://www.versatile-thermostat.org/fr/docs/over-valve/) page. Preserve technical identifiers; do not treat these workflow labels as MCP creation schemas.

## C — Prepare a confirmed plan

Summarize name/room, type and reason, indoor sensor, exterior source if required, exact controlled entities, regulation, local versus inherited settings, desired preset temperatures and optional features. Mark unknowns explicitly; do not fill them with guessed entities or values. Separate shared configuration, participation in central mode and central-boiler demand. Existing central configuration is not permission to edit it; explain the scope of each inherited block. Do not select inheritance when no central configuration exists.

For every proposed number, distinguish a documented default/recommendation (with source and conditions), an illustrative example, a user preference, and a value requiring measurement/adaptation. State units. Do not supply universal TPI coefficients, device power or cycle duration. Device power comes from equipment evidence; cycle timing must respect equipment constraints. Room targets are user choices, not measured optima.

## D — Guide through the actual UI

Use [installation](https://www.versatile-thermostat.org/fr/docs/installation/) if installation/restart is incomplete, then [creation](https://www.versatile-thermostat.org/fr/docs/creation/). Ask the user to open the integration creation/configuration interface. For each screen: name where to intervene, give only its confirmed fields and justified values, explain the material choice, then wait for the user's result or the one missing fact. Do not advance as though a screen was saved without evidence.

Typical menu checkpoints, adapted to the visible version:

1. Select the agreed thermostat type, then [main attributes](https://www.versatile-thermostat.org/fr/docs/base-attributes/): name, room sensor, cycle, power/unit if known, and central choices. If local complementary settings are required, establish exterior source and temperature limits/step from the actual form.
2. Select underlying entities using the type-specific page. Explain any device-specific mappings or valve prerequisites before proceeding.
3. Select the applicable [regulation](https://www.versatile-thermostat.org/fr/docs/algorithms/). Classic TPI, Auto-TPI learning, over-climate auto-regulation and plugin algorithms such as SmartPI are distinct. Verify installed availability and compatibility; never assume a plugin is installed or activate learning automatically.
4. Set [presets](https://www.versatile-thermostat.org/fr/docs/feature-presets/) from desired comfort/eco/frost/boost temperatures and inherited/local choices. Explain absence variants when relevant; selecting a preset later is different from configuring its temperature.
5. Configure only requested applicable features. [Windows](https://www.versatile-thermostat.org/fr/docs/feature-window/): choose documented sensor or automatic detection, not both. [Presence](https://www.versatile-thermostat.org/fr/docs/feature-presence/): distinguish household absence from room motion. [Power shedding](https://www.versatile-thermostat.org/fr/docs/feature-power/): verify measurement, threshold sources and consistent power units. [Advanced/safety](https://www.versatile-thermostat.org/fr/docs/feature-advanced/): explain freshness/timing safeguards; do not disable them to silence an anomaly. [Central mode](https://www.versatile-thermostat.org/fr/docs/feature-central-mode/): establish participation separately from inheritance.
6. Review required sections, unresolved warnings and the final summary before the user finalizes. Enabled features may add required pages. If finalization is blocked, use the actual error and documented dependency rather than bypassing validation.

## E — Verify what was created

After the user reports completion, resolve the resulting entity anew; do not construct its ID from the chosen name. When accessible, inspect status and reasons: availability, current temperature/unit, target, mode/preset, underlying availability and sensor freshness. Check important dependencies and inherited settings only to the extent exposed. Use VTherm reasons before inferring a cause. Report each check as observed, user-reported or unavailable; entity creation alone does not prove physical operation.

Without suitable tools, give a manual checklist in HA: locate the entity, check those fields and dependency states, review warnings/reasons. Propose a supervised normal-operation test within the equipment's documented limits, preserving safeguards, with an expected observable result and restoration of the user's chosen settings. Ask the user whether to perform it; guidance never executes the test. Any later explicit supported control request uses the existing write workflow and permissions. Do not repeatedly cycle relays, force valves or bypass protections. Allow documented response delays and distinguish command acceptance, declared heating and observed room response.

## F — Optional next steps

Once checks are satisfactory, offer only relevant comfort, scheduling, absence or protection refinements via [user-guide.md](user-guide.md). Classic TPI analysis requires the dedicated tools and sufficient history under the existing eligibility rules; it cannot validate a newly created thermostat immediately. Its switch-history limitation is narrower than TPI configuration compatibility. Do not apply that analysis to Auto-TPI, SmartPI, ordinary over-climate regulation or unsupported valve histories. Offer measured observation before tuning and preserve exploratory results and separate explicit application.
