# VTherm through ha-mcp

Use this reference when the native VTherm tool for a supported task is absent and ha-mcp is available. It also applies to mixed installations. Discover actual tools and schemas; names and options vary by version. Client discovery/search tools may reveal deferred tools: an initial short catalog is not proof of absence. Do not assume ha-mcp proxies Home Assistant's native LLM tools.

## Select the connection and target

Keep discovery, reads, service calls and verification on the same Home Assistant instance. Two connections can point to different homes, and identical entity IDs do not establish instance identity. If the relationship is unknown, keep the workflow on one connection; clarify before combining evidence or changing an ambiguous target. Do not require both servers or ask users to install the official one for ordinary supported tasks.

Typical capabilities are `ha_search` (older catalogs may offer `ha_search_entities`), `ha_get_entity`, `ha_get_state`, `ha_get_history`, `ha_list_services` and `ha_call_service`. Use the live schema rather than copying parameters onto a differently named tool. Respect any server-provided usage guide relevant to the selected tools.

Resolve a room among `climate` entities using available search tools; follow search pagination before claiming an exhaustive list. Verify VTherm membership from registry platform `versatile_thermostat` when available, or unambiguous integration metadata. A climate domain, room name, or friendly name alone does not establish membership. If identity is uncertain, report that and avoid a guessed write. Fetch current state/attributes for confirmed targets; retain returned units and enums.

## Diagnose from current attributes

When native `diagnose` is absent, use a read-only state inspection. Read full attributes, including nested `specific_states` and feature-manager sections when present. Interpret `specific_states.messages`, `hvac_off_reason`, `hvac_mode_reason`, `should_device_be_active` and `unavailable_underlying_entities` in that order, then the relevant manager and temperatures. Read [diagnostic-reference.md](diagnostic-reference.md) for interpretation and documentation.

Explain that this diagnosis uses exposed state attributes. Do not invent the native `reasons`, `feature_flags` or `support_resources` response, or assume missing sections mean disabled protections. If attributes do not establish a cause, explain what is missing and offer the documentation/log workflow. Native collector logs and linked-plugin diagnostics require `versatile_thermostat__get_logs` and `versatile_thermostat__get_plugins`. Report when these tools are absent. Exposed attributes can provide context, and Recorder history can show recorded state changes. Neither establishes collector contents or native plugin coverage. Do not call a generic write tool or execute code to collect diagnostic data.

For central configuration and dependencies, apply the [central-context procedure](diagnostic-reference.md#central-configuration-and-per-thermostat-context) to the fields actually exposed. Generic state access does not guarantee access to configuration entries or inheritance flags. Include confirmed central usage in the diagnosis and mark unavailable central settings or provenance unknown; do not execute templates/code or use configuration writes to obtain them.

## Temperature, preset and HVAC mode

Only for an explicit change request and a verified target:

1. Read fresh state, units, supported modes/presets and temperature bounds. Resolve a natural-language preset against returned values. If a necessary constraint or unit cannot be established, clarify or report the limitation before writing.
2. Discover the service schema when needed with a read-only service-list tool. Use only the matching standard climate service: `climate.set_temperature` with `temperature`, `climate.set_preset_mode` with `preset_mode`, or `climate.set_hvac_mode` with `hvac_mode`. Convert units only when needed to preserve the requested target and state the conversion. Reject nonfinite or out-of-range targets; never silently clamp. Never use toggle, direct relay control, configuration edits, or broad area/device targets to implement these requests.
3. Call once for the exact entity. For a catalog exposing the documented `ha_call_service` schema, a temperature example is:

   ```json
   {"domain":"climate","service":"set_temperature","entity_id":"climate.thermostat_salon","data":{"temperature":19.5}}
   ```

4. Re-read with `ha_get_state` and compare requested and observed values. Service acceptance alone does not prove the target was applied. On timeout or uncertain outcome, inspect state before any retry; do not automatically replay a write or switch servers. Explain any mismatch and VTherm's possible subsequent adjustments without inventing their cause.

These calls use ha-mcp's credentials and policies. They do not inherit the native tools' Assist-exposure checks, per-user context or confirmation tokens. Never claim otherwise. Obey read-only mode, denied/hidden entities, approval gates and authorization errors; do not change policy/settings or reroute to defeat them. An absent write tool can reflect a restriction, not missing functionality. Generic control is allowed only when there is no known restriction or refusal for the requested operation/target.

## Recorded charts and CSV

Use `ha_get_history` when native `get_history` is absent, with explicit fixed timezone-aware start/end bounds. For versions supporting these options, select raw history (`source: history`), `minimal_response: false`, `significant_changes_only: false`, `summary: false`, and chronological order (`order: asc`) to preserve recorded climate attributes and attribute-only changes. Inspect the live schema first; omit unsupported options and disclose any resulting loss of detail.

Page one entity at a time using the returned continuation information; keep bounds and query options fixed. Use the row update/event timestamp for attribute changes and retain returned timezone metadata; an unchanged earlier `last_changed` must not move attribute-only temperature/setpoint updates into the past. Check warnings, truncation, unknown/unavailable states, missing attributes, units and coverage before plotting. If continuation cannot be completed, label the chart partial. Do not apply the native tool's 500-row/14-day schema to ha-mcp; use its actual limits. Narrow requests to stay within workload limits without defeating permission denials. Do not claim that statistics are raw event history, or that changing a climate attribute necessarily changed its HVAC mode.

Create the requested chart/CSV on the client. Annotate only recorded events; preserve missing data. Associate sensor/relay series only when their identity and access are established; label those as separate measured series. Never reconstruct physical duty or historical SmartPI output from HVAC action/current percentages. Missing history cannot be supplied by current attributes. Generic history is not the native TPI identification pipeline.

## TPI capability boundary

Without `check_tpi_optimization` and `analyze_tpi_coefficients`, explain that this skill cannot run the native coefficient analysis through the current connection. Current coefficients and observed tracking may be described, but cannot establish optimality or justify invented replacement estimates. Do not rebuild the estimator in a code-execution tool, template, SQL query or generic service call.

Without native `set_tpi_coefficients`, report that coefficient application is unsupported by this skill on that connection, including when the user supplies values. Do not substitute `versatile_thermostat.set_tpi_parameters`: a generic call would omit the native comparison/token workflow. Ordinary climate control and charts remain usable. Connecting the official MCP is an optional way to obtain native functions, not a prerequisite for the other workflows.

## Sources and validation scope

Documentation reviewed on 3 November 2026: [ha-mcp repository](https://github.com/homeassistant-ai/ha-mcp), [tool explorer](https://homeassistant-ai.github.io/ha-mcp/tools/). Catalog discovery on the user's installed version is authoritative; a roadmap/wiki proposal is not evidence of implemented tool passthrough. This reference does not establish a live test against any installation.
