---
name: vtherm-mcp
description: Help users install, configure, understand, diagnose and control Versatile Thermostat (VTherm). Use for choosing a thermostat type, sensors, presets, schedules, absence, heating/cooling, windows, safety, power shedding, central control, recorded charts and TPI questions. Provide documentation-based guidance without MCP; use Home Assistant's official MCP server, ha-mcp, or both for supported live operations. Keep diagnosis read-only and coefficient estimates exploratory.
---

# VTherm MCP

Support the official Home Assistant MCP server, ha-mcp alone, or both. Discover capabilities before selecting a workflow; do not require a particular server name. Keep diagnostic requests read-only and use write tools only for explicit control requests.

## When to Activate

Use for VTherm setup, everyday use, troubleshooting and supported control, in the user's language. For explanations or setup advice, start with the documentation workflow; a Home Assistant connection is optional. Discover live tools when the request needs the user's actual installation. Without usable MCP tools, provide documented guidance and interpret user-supplied evidence, clearly identifying its source and age.

Read [references/user-guide.md](references/user-guide.md) for first setup, thermostat-type choice, presets/schedules/absence, central configuration, comfort/energy review, missing tools and support preparation. Ask only for information that changes the answer; use accessible authorized evidence first. Keep ordinary explanations short and offer the next relevant step rather than a full technical questionnaire.

When a user needs an additional capability, interface or device integration, consult the current [official complements catalog](https://www.versatile-thermostat.org/fr/plugins/) and the add-on workflow in that reference. Select by the user's actual need; do not maintain a copied plugin list or recommend remembered plugins without checking their current purpose and compatibility.

Installing/configuring VTherm, editing preset temperatures, timed presets, central selectors, automations and protection settings are distinct from the supported climate writes below. Explain their documented UI workflow or prepare a requested draft; this skill does not provide execution workflows for those changes. Never turn a timed or scheduled request into an indefinite immediate write.

History export and classic TPI tools were merged into the official VTherm `main` in [PR #2167](https://github.com/jmcollin78/versatile_thermostat/pull/2167) on 1 November 2026. A merge does not establish availability in a published release, the user's installed version, or the current MCP catalog. Discover the actual tools before selecting these workflows; installing this skill does not install or update VTherm or refresh a connector's catalog.

## Select a workflow from available capabilities

Discover dedicated VTherm tools and ha-mcp tools, including deferred tools if the client supports discovery. Select per operation, not just by server label. Respect workspace/user restrictions on which connections may be used.

| Available capabilities | Workflow |
| --- | --- |
| No usable MCP connection | Documentation and user-supplied evidence via [references/user-guide.md](references/user-guide.md); no live inspection or control claims. |
| Native VTherm tools only | Use the dedicated tools described below. Report any missing capability. |
| ha-mcp tools only | Use the state, diagnosis, history and standard climate-control workflows in [references/ha-mcp.md](references/ha-mcp.md). Native TPI analysis/application remains unavailable without its dedicated tools. |
| Both | Prefer the dedicated VTherm tool for each covered operation. Use ha-mcp for supported operations the native catalog lacks, subject to the restrictions below. |

Read [references/ha-mcp.md](references/ha-mcp.md) before using ha-mcp, including in a mixed setup. Announce the chosen route and material limits briefly. Never issue the same write through both servers. Keep the target and Home Assistant instance consistent; equal entity IDs across servers do not prove equal instances.

Distinguish capability absence from refusal. A generic route is allowed only for the supported operations in that reference when the corresponding native tool is genuinely absent, with no known entity/exposure, permission, read-only or policy restriction. If a dedicated tool exists but rejects the target/value, denies access, requests confirmation, or has an uncertain outcome, stop that operation on that route. For a confirmation request, pause for the required user decision, then continue only on the same native route with its confirmation workflow. Never switch servers, use generic services or indirect APIs to obtain the refused result. Do not treat missing write tools or an entity omitted from the native exposed list as authorization to broaden access.

## Resolve the native tool namespace

Find tools by the suffix `versatile_thermostat__...`; do not assume a fixed MCP server name or prefix.

Typical names through the Assist API are:

- `assist__versatile_thermostat__list_vtherms`
- `assist__versatile_thermostat__get_status`
- `assist__versatile_thermostat__diagnose`
- `assist__versatile_thermostat__get_history`
- `assist__versatile_thermostat__check_tpi_optimization`
- `assist__versatile_thermostat__analyze_tpi_coefficients`
- `assist__versatile_thermostat__set_temperature`
- `assist__versatile_thermostat__set_preset`
- `assist__versatile_thermostat__set_hvac_mode`
- `assist__versatile_thermostat__set_tpi_coefficients`

If a dedicated tool is absent, apply the capability table above. Do not invent missing tools or claim ha-mcp implements the native API. The workflows below describe native tools unless a ha-mcp route is explicitly mentioned.

Read [references/tool-reference.md](references/tool-reference.md) when exact parameters, returned fields, or examples are needed.

## Match the user's language

Answer in the language of the request, including French, English and German; do not restrict help to these languages. Prefer documentation in that language when available.

Keep machine values exactly as returned by Home Assistant. Do not translate entity IDs, preset names, or HVAC mode values such as `comfort`, `boost`, `heat`, or `off` before passing them to a tool.

## Choose the right workflow

### List or identify a VTherm

Use native `list_vtherms`, or the verified ha-mcp entity-discovery workflow when absent, when the user asks what VTherms exist or when a room/name must be resolved to an entity ID.

For a write request, if one VTherm is the only clear match, use it. If multiple VTherms are plausible, do not guess which entity to modify.

### Inspect current state

Use native `get_status`, or ha-mcp state inspection when absent, when the user asks for the complete current state/attributes or when `diagnose` does not expose a needed raw attribute.

For a simple overview of several thermostats, prefer `list_vtherms` instead of calling `get_status` repeatedly.

### Debug or diagnose

For requests such as "pourquoi il ne chauffe pas ?", "why is it idle?", "est-il délesté ?", "debug", "diag", window/safety/presence/motion/auto-start-stop questions, call native `diagnose` first when available. Otherwise use the read-only ha-mcp attribute diagnosis from its reference; do not stop solely because the native tool is absent.

Interpret in this order:

1. `reasons.messages`
2. `reasons.hvac_off_reason`
3. `reasons.hvac_mode_reason`
4. `reasons.should_device_be_active`
5. `reasons.unavailable_underlying_entities`
6. Relevant diagnostic section such as `power_manager`, `window_manager`, `safety_manager`, `presence_manager`, `motion_manager`, `auto_start_stop_manager`, or the VTherm type section
7. Temperatures, target delta, HVAC action, preset, and other contextual values

Do not infer a root cause from temperature difference alone. Distinguish facts returned by VTherm from interpretation. If the data does not establish a cause, say the diagnosis is inconclusive.

For deeper troubleshooting, read [references/diagnostic-reference.md](references/diagnostic-reference.md).

### Change temperature

Use native `set_temperature`, or the ha-mcp standard climate-control route when absent, only when the user explicitly asks to change the target temperature.

Preserve the requested target and its units. The native tool validates the entity's min/max range and Home Assistant control permissions; the ha-mcp route requires the fresh-state checks and readback described in its reference.

### Change preset

Use native `set_preset`, or the ha-mcp standard climate-control route when absent, only when the user explicitly asks to select a preset.

Use a preset value present in the entity's `preset_modes`. If the user uses a natural-language label that does not exactly match a returned preset, resolve it from the available values rather than inventing one.

### Change HVAC mode

Use native `set_hvac_mode`, or the ha-mcp standard climate-control route when absent, only when the user explicitly asks to turn a VTherm on/off or select an HVAC mode.

Use a mode present in `hvac_modes`. Prefer explicit `heat`, `cool`, `off`, etc.; never emulate a requested state with a toggle operation.

## Write-safety rules

Treat `set_temperature`, `set_preset`, `set_hvac_mode`, and `set_tpi_coefficients` and generic climate-service calls as real Home Assistant changes.

- Never call a write tool during a read, debug, diagnose, explain, or "what would happen" request.
- Never change a different thermostat merely because the requested entity is unavailable.
- Never substitute a generic service call if the dedicated VTherm write tool rejects a value.
- Preserve the user's requested scope: one thermostat means one thermostat.
- After a successful write, summarize the requested change and `state_after_call` when returned.
- Explain that VTherm may subsequently alter the effective state because of central mode, window, power shedding, safety, auto-start/stop, presets, or regulation.
- Resolve explicit multi-room requests to a concrete list of intended entities before writing; clarify ambiguous membership. Apply only supported operations to those targets and report success, failure or uncertain outcome per entity and requested operation, including partial changes when mode and temperature need separate calls. Stop on a refusal or uncertain outcome and report remaining operations/targets as unattempted; inspect uncertain state before considering a retry. Do not claim a group change was atomic or automatically replay writes.

## Anti-Patterns

- Requiring MCP access to explain or help configure VTherm.
- Treating preset selection as editing its configured temperature, or `away` as an always-selectable preset.
- Applying an immediate temperature or preset for a request that also requires a duration, schedule or automatic restoration.
- Promising energy savings from temperature curves or HVAC action alone.
- Treating documentation, user-supplied snapshots or a merged PR as proof of the live installation's capabilities.

## Documentation and logs

Always include an official VTherm documentation link in each substantive VTherm answer, including setup advice, diagnosis, control results, charts, TPI analysis and capability limitations. Prefer the specific relevant page whose URL has been verified, in the user's language when available; otherwise link to [the official documentation](https://www.versatile-thermostat.org/). A link offered for further reading does not mean the page was consulted: say when browsing was unavailable, and never present an unread page as supporting evidence.

Use [the official VTherm website](https://www.versatile-thermostat.org/) as the primary source for current VTherm documentation. This applies to general questions about features, configuration and parameters as well as troubleshooting, on all three MCP configurations. When the answer is uncertain, a field is unfamiliar, or the skill/tool result does not explain the behavior, look up the relevant current page before answering if web access is available. Start from the site and follow its navigation/search; prefer the user's language (French `/fr/`, English `/en/`, German `/de/`). Follow current links rather than assuming the static paths in this skill remain valid.

Cite the specific page used and distinguish documented behavior from facts measured on the user's installation. Current documentation may describe a newer release: check the installed version when material and do not assert that a documented feature is installed or exposed by MCP. Documentation cannot establish a live root cause or supply missing historical data.

If browsing is unavailable, a page cannot be reached, or the documentation still does not establish an answer, state what remains unknown and direct the user to the most relevant verified page, or to https://www.versatile-thermostat.org/ when no exact page is verified. Do not invent explanations, settings, paths or quotations, and do not claim to have read a page that was not accessed.

If the instantaneous state is insufficient, recommend the VTherm log workflow rather than guessing:

1. Export relevant logs with `versatile_thermostat.download_logs` if the user wants to continue debugging.
2. Use the VTherm log analyzer at `https://www.versatile-thermostat.org/fr/debugger/`.

Do not trigger log download or upload logs to an external analyzer without the user's explicit request.

Read [references/diagnostic-reference.md](references/diagnostic-reference.md) for documentation URLs and the diagnostic sections returned by the tool.

## Classic TPI coefficient workflow (native tools required)

With ha-mcp alone and no native TPI tools, explain the capability limit in its reference; ordinary state, diagnosis, charts and climate control remain supported. Coefficient writes require native `set_tpi_coefficients`, including explicit user-supplied values. Do not substitute the generic TPI service or recreate the estimator.

A simple question such as “les coef du thermostat chambre parents sont-ils optimaux ?”, “mes coefficients sont-ils bien réglés ?” or “are my thermostat coefficients optimal?” requests the read-only check-and-analysis workflow. Resolve room names with `list_vtherms` when necessary; users need not name tools or entity IDs. A dashboard link does not make browser access a prerequisite. Current coefficients or a dashboard snapshot cannot establish optimality. If calculation tools are absent from the current connector, state that availability problem rather than falling back to the browser or generic tools. An older connection catalog can differ from another client’s catalog; do not claim a prompt can expose missing tools.

Use `check_tpi_optimization` (stage 1/3), then `analyze_tpi_coefficients` (stage 2/3). Announce each stage before calling. Both are read-only; neither can apply coefficients. Require classic TPI, at least 7 usable days and at most the last 14 days of Recorder history. No CSV, plot or report file is needed. Use the configured temperature sources and actual physical switch histories, including multiple switches under VTherm's equal power share convention. State explicitly that climate and valve history analysis is not yet supported when `unsupported_thermostat_type` is returned.

Present the current/proposed comparison, metrics, multi-resolution stability, bootstrap intervals and warnings. Always describe the estimates as exploratory and bootstrap intervals as approximate and uncalibrated; `recommendation_reliable: false` is not a certification. Keep the preferred 6-hour nominal time constant (about 18 hours to 95% correction under the local unsaturated model), not an identified optimum. Distinguish current and proposed nominal responses from observed room performance. The Kalman comparison is disabled in MCP pending validation: explain `experimental_model_not_validated_for_application`; it must not run or return coefficients. An older server may still return experimental values; do not recommend or apply them. Offline research does not establish reliability on the user’s home. Explain its reason; never clip, reconstruct or propose values from a refused fit, and never describe the remaining OLS estimate as reliable just because Kalman is unavailable. Free-run errors use recorded weather/heater inputs, not a simulation of new controller settings; report the number of usable runs. Include the returned `applicability_note` in the user’s language: estimates reflect the analyzed period; consider a new analysis after substantial heating/usage changes or degraded regulation. Outdoor temperature changes alone do not require recalculation.

Use `set_tpi_coefficients` (stage 3/3) only after an explicit user decision, or directly when the user explicitly supplies coefficient values. At least one of `coef_int` / `coef_ext` is required; partial changes need no history or prior analysis. This is a real write through the VTherm service, subject to authenticated user, Assist exposure and CONTROL permission.

For `large_change_confirmation_required`, show the comparison and ask for additional explicit user confirmation. Never self-confirm. Only after confirmation, repeat the same requested values with `confirm_large_change: true` and the returned `confirmation_token` (10-minute expiry, bound to user/entity/current/requested values). If the current settings changed, show the new comparison and request confirmation again. No generic service fallback on rejection. After calling, report `previous`, `requested`, `applied`; `state_changed: null` means application is uncertain and must be checked before retrying.


## Recorder history exports and charts

Resolve relative periods against the user's timezone and current time; state fixed start/end bounds and label the chart timezone. Distinguish an elapsed duration such as 48 hours from local calendar days across daylight-saving changes. Clarify dates only when ambiguity materially affects the request.

A request such as “tu me fais un graphe pro du thermostat de la cuisine annoté sur 48h” asks for an actual client-created chart using historical data. Resolve the room, call native `get_history` or the ha-mcp raw-history route when the native tool is absent, fetch all pages and create the chart if the client supports it. Do not substitute ApexCharts/Lovelace YAML unless the user explicitly requests a dashboard card. SmartPI supports history export; its refusal for TPI coefficient analysis must not block charts. Annotate recorded target/preset changes; report missing series. This tool does not expose historical SmartPI power percentages or physical relay states, and current values cannot reconstruct them. Do not invent those series or equate HVAC action with physical duty. If neither native history nor a usable ha-mcp history tool is available, explain the missing capability; no prompt can expose an absent tool. For ha-mcp, use its own history schema and completeness checks from the reference.

Use native `get_history` for recorded VTherm climate states/temperatures/targets/HVAC action/presets on any thermostat type. Pass `entity_id`, timezone-aware ISO `start_time` and `end_time` (at most 14 days per period), optional `offset` (0) and `limit` (1–500, default 500). Follow `next_offset` with identical bounds until null. Create the requested CSV/chart on the client when supported. This is read-only, requires authenticated READ permission and current Assist exposure. Associated sensor/relay histories are outside this tool's scope. Never invent missing values, replace null with zero, or present event changes as regular samples. Report retention/gap limits; finished pagination does not establish complete historical coverage. No generic fallback on refusal.
