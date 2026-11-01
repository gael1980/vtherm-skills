---
name: vtherm-mcp
description: Use Versatile Thermostat (VTherm) through Home Assistant's official MCP/Assist tools. Use when a user asks in French or English for a thermostat graph/annotated chart/CSV ("fais-moi un graphe pro du thermostat cuisine annoté sur 48h"), to list VTherms, inspect a VTherm state, debug or diagnose why it is heating, idle, off, shed, in safety, affected by a window/presence/motion/auto-start-stop rule, or explicitly change a VTherm target temperature, preset, or HVAC mode, ask whether thermostat coefficients are optimal or well tuned ("mes coef sont-ils optimaux ?"), check/analyze classic TPI coefficients from Recorder history, or explicitly set TPI coefficients. Also use for VTherm troubleshooting that may require the official VTherm documentation, log collector, or log analyzer. Prefer the dedicated versatile_thermostat MCP tools over generic Home Assistant tools for these tasks.
---

# VTherm MCP

Use the dedicated Versatile Thermostat tools exposed by Home Assistant's official MCP server. Keep diagnostic requests read-only and use write tools only for explicit control requests.

History export and classic TPI tools were merged into the official VTherm `main` in [PR #2167](https://github.com/jmcollin78/versatile_thermostat/pull/2167) on 1 November 2026. A merge does not establish availability in a published release, the user's installed version, or the current MCP catalog. Discover the actual tools before selecting these workflows; installing this skill does not install or update VTherm or refresh a connector's catalog.

## Resolve the tool namespace

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

If the dedicated VTherm tools are unavailable, report that clearly. Do not silently replace a VTherm diagnostic or control request with unrelated generic Home Assistant tools.

Read [references/tool-reference.md](references/tool-reference.md) when exact parameters, returned fields, or examples are needed.

## Match the user's language

Answer in the language of the request. Support both French and English naturally.

Keep machine values exactly as returned by Home Assistant. Do not translate entity IDs, preset names, or HVAC mode values such as `comfort`, `boost`, `heat`, or `off` before passing them to a tool.

## Choose the right workflow

### List or identify a VTherm

Use `list_vtherms` when the user asks what VTherms exist or when a room/name must be resolved to an entity ID.

For a write request, if one VTherm is the only clear match, use it. If multiple VTherms are plausible, do not guess which entity to modify.

### Inspect current state

Use `get_status` when the user asks for the complete current state/attributes or when `diagnose` does not expose a needed raw attribute.

For a simple overview of several thermostats, prefer `list_vtherms` instead of calling `get_status` repeatedly.

### Debug or diagnose

For requests such as "pourquoi il ne chauffe pas ?", "why is it idle?", "est-il délesté ?", "debug", "diag", window/safety/presence/motion/auto-start-stop questions, call `diagnose` first.

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

Use `set_temperature` only when the user explicitly asks to change the target temperature.

Pass the numeric target requested by the user. The tool validates the entity's min/max range and Home Assistant control permissions.

### Change preset

Use `set_preset` only when the user explicitly asks to select a preset.

Use a preset value present in the entity's `preset_modes`. If the user uses a natural-language label that does not exactly match a returned preset, resolve it from the available values rather than inventing one.

### Change HVAC mode

Use `set_hvac_mode` only when the user explicitly asks to turn a VTherm on/off or select an HVAC mode.

Use a mode present in `hvac_modes`. Prefer explicit `heat`, `cool`, `off`, etc.; never emulate a requested state with a toggle operation.

## Write-safety rules

Treat `set_temperature`, `set_preset`, `set_hvac_mode`, and `set_tpi_coefficients` as real Home Assistant changes.

- Never call a write tool during a read, debug, diagnose, explain, or "what would happen" request.
- Never change a different thermostat merely because the requested entity is unavailable.
- Never substitute a generic service call if the dedicated VTherm write tool rejects a value.
- Preserve the user's requested scope: one thermostat means one thermostat.
- After a successful write, summarize the requested change and `state_after_call` when returned.
- Explain that VTherm may subsequently alter the effective state because of central mode, window, power shedding, safety, auto-start/stop, presets, or regulation.

## Documentation and logs

When a diagnosis is complex or a VTherm-specific field needs interpretation, consult the matching VTherm documentation if web access is available. Prefer the language of the user's request.

If the instantaneous state is insufficient, recommend the VTherm log workflow rather than guessing:

1. Export relevant logs with `versatile_thermostat.download_logs` if the user wants to continue debugging.
2. Use the VTherm log analyzer at `https://www.versatile-thermostat.org/fr/debugger/`.

Do not trigger log download or upload logs to an external analyzer without the user's explicit request.

Read [references/diagnostic-reference.md](references/diagnostic-reference.md) for documentation URLs and the diagnostic sections returned by the tool.

## Classic TPI coefficient workflow

A simple question such as “les coef du thermostat chambre parents sont-ils optimaux ?”, “mes coefficients sont-ils bien réglés ?” or “are my thermostat coefficients optimal?” requests the read-only check-and-analysis workflow. Resolve room names with `list_vtherms` when necessary; users need not name tools or entity IDs. A dashboard link does not make browser access a prerequisite. Current coefficients or a dashboard snapshot cannot establish optimality. If calculation tools are absent from the current connector, state that availability problem rather than falling back to the browser or generic tools. An older connection catalog can differ from another client’s catalog; do not claim a prompt can expose missing tools.

Use `check_tpi_optimization` (stage 1/3), then `analyze_tpi_coefficients` (stage 2/3). Announce each stage before calling. Both are read-only; neither can apply coefficients. Require classic TPI, at least 7 usable days and at most the last 14 days of Recorder history. No CSV, plot or report file is needed. Use the configured temperature sources and actual physical switch histories, including multiple switches under VTherm's equal power share convention. State explicitly that climate and valve history analysis is not yet supported when `unsupported_thermostat_type` is returned.

Present the current/proposed comparison, metrics, multi-resolution stability, bootstrap intervals and warnings. Always describe the estimates as exploratory and bootstrap intervals as approximate and uncalibrated; `recommendation_reliable: false` is not a certification. Keep the preferred 6-hour nominal time constant (about 18 hours to 95% correction under the local unsaturated model), not an identified optimum. Distinguish current and proposed nominal responses from observed room performance. The Kalman comparison is disabled in MCP pending validation: explain `experimental_model_not_validated_for_application`; it must not run or return coefficients. An older server may still return experimental values; do not recommend or apply them. Offline research does not establish reliability on the user’s home. Explain its reason; never clip, reconstruct or propose values from a refused fit, and never describe the remaining OLS estimate as reliable just because Kalman is unavailable. Free-run errors use recorded weather/heater inputs, not a simulation of new controller settings; report the number of usable runs. Include the returned `applicability_note` in the user’s language: estimates reflect the analyzed period; consider a new analysis after substantial heating/usage changes or degraded regulation. Outdoor temperature changes alone do not require recalculation.

Use `set_tpi_coefficients` (stage 3/3) only after an explicit user decision, or directly when the user explicitly supplies coefficient values. At least one of `coef_int` / `coef_ext` is required; partial changes need no history or prior analysis. This is a real write through the VTherm service, subject to authenticated user, Assist exposure and CONTROL permission.

For `large_change_confirmation_required`, show the comparison and ask for additional explicit user confirmation. Never self-confirm. Only after confirmation, repeat the same requested values with `confirm_large_change: true` and the returned `confirmation_token` (10-minute expiry, bound to user/entity/current/requested values). If the current settings changed, show the new comparison and request confirmation again. No generic service fallback on rejection. After calling, report `previous`, `requested`, `applied`; `state_changed: null` means application is uncertain and must be checked before retrying.


## Recorder history exports and charts

A request such as “tu me fais un graphe pro du thermostat de la cuisine annoté sur 48h” asks for an actual client-created chart using historical data. Resolve the room, call `get_history`, fetch all pages and create the chart if the client supports it. Do not substitute ApexCharts/Lovelace YAML unless the user explicitly requests a dashboard card. SmartPI supports history export; its refusal for TPI coefficient analysis must not block charts. Annotate recorded target/preset changes; report missing series. This tool does not expose historical SmartPI power percentages or physical relay states, and current values cannot reconstruct them. Do not invent those series or equate HVAC action with physical duty. If the connection lacks `get_history`, explain the missing-tool problem; no prompt can expose an absent tool.

Use `get_history` for recorded VTherm climate states/temperatures/targets/HVAC action/presets on any thermostat type. Pass `entity_id`, timezone-aware ISO `start_time` and `end_time` (at most 14 days per period), optional `offset` (0) and `limit` (1–500, default 500). Follow `next_offset` with identical bounds until null. Create the requested CSV/chart on the client when supported. This is read-only, requires authenticated READ permission and current Assist exposure. Associated sensor/relay histories are outside this tool's scope. Never invent missing values, replace null with zero, or present event changes as regular samples. Report retention/gap limits; finished pagination does not establish complete historical coverage. No generic fallback on refusal.
