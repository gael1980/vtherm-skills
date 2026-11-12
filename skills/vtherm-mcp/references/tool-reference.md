# Native VTherm MCP tool reference

This reference documents the dedicated native tools. For ha-mcp-only or mixed setups, select the route in [../SKILL.md](../SKILL.md) and read [ha-mcp.md](ha-mcp.md). Do not apply these native schemas, exposure guarantees or confirmation tokens to generic tools.

## Contents

- Tool naming
- Read tools
- Write tools
- Common examples
- Errors and fallbacks

## Tool naming

The Home Assistant Assist API commonly namespaces the integration tools as:

```text
assist__versatile_thermostat__list_vtherms
assist__versatile_thermostat__get_status
assist__versatile_thermostat__diagnose
assist__versatile_thermostat__get_history
assist__versatile_thermostat__get_logs
assist__versatile_thermostat__get_plugins
assist__versatile_thermostat__check_tpi_optimization
assist__versatile_thermostat__analyze_tpi_coefficients
assist__versatile_thermostat__set_temperature
assist__versatile_thermostat__set_preset
assist__versatile_thermostat__set_hvac_mode
assist__versatile_thermostat__set_tpi_coefficients
```

Match tools by the `versatile_thermostat__...` suffix if the client applies a different namespace.

## Read tools

### `versatile_thermostat__list_vtherms`

Parameters: none.

Use to list VTherm entities exposed to Assist and obtain a compact state.

Returned data includes `count` and `thermostats`. Newer implementations can also return `central_configuration` and per-thermostat usage, as described below. A thermostat summary can contain:

```text
entity_id
name
hvac_mode
hvac_action
preset_mode
current_temperature
target_temperature
target_minus_current
min_temperature
max_temperature
target_temperature_step
hvac_modes
preset_modes
thermostat_type
messages
```

### `versatile_thermostat__get_status`

Parameters:

```json
{"entity_id": "climate.thermostat_salon"}
```

Returns the complete current Home Assistant state:

```text
entity_id
state
last_changed
last_updated
attributes
```

Use for raw attributes not already surfaced by `diagnose`.

In implementations exposing `central_configuration`, the result includes central-entry presence (`configured`), eligibility (`active`), reviewed `settings`, and `thermostat` usage with `inheritance`, `uses_central_configuration`, central-mode/boiler participation and `effective_settings`. Inherited presets use `effective_preset_temperatures`. List results expose the central summary and compact usage per thermostat; `diagnose` includes the same detailed object as status. Access requires authenticated READ permission and a loaded, currently exposed target. Admins can read reviewed central groups, while nonadmins see only inherited groups for their authorized targets. Raw configuration, lock codes, boiler service strings and templates are excluded. `unavailable` and `ambiguous` preserve uncertainty. The object is proposed in [PR #2232](https://github.com/jmcollin78/versatile_thermostat/pull/2232), open and unmerged as checked on 12 November 2026; inspect actual results before using it.

Older implementations return only entity state. Central-mode and boiler participation may appear in `attributes.configuration`, with the last reported central mode in `attributes.specific_states`. Per-group inheritance and full central settings are not guaranteed. Follow [central-context interpretation](diagnostic-reference.md#central-configuration-and-per-thermostat-context) and report missing provenance explicitly.

### `versatile_thermostat__diagnose`

Parameters:

```json
{"entity_id": "climate.thermostat_salon"}
```

Use first for behavior analysis and troubleshooting. It returns a structured snapshot with:

```text
entity_id
name
availability
current
reasons
feature_flags
sections
support_resources
last_changed
last_updated
```

Primary reason fields:

```text
reasons.messages
reasons.hvac_off_reason
reasons.hvac_mode_reason
reasons.should_device_be_active
reasons.device_actives
reasons.unavailable_underlying_entities
reasons.temperature_slope
reasons.is_sleeping
reasons.is_recalculate_scheduled
```

### `versatile_thermostat__get_logs`

Read the VTherm collector in memory. Both this tool and `get_plugins` require a loaded VTherm climate entity exposed to the assistant. These reads do not run services or change configuration.

- Required: `entity_id`. Optional: `start_time`, `end_time` and `min_level` (`DEBUG`, `INFO`, `WARNING` or `ERROR`, default `DEBUG`).
- Timestamps require ISO 8601 with seconds and an explicit offset, such as `2026-11-08T18:30:00+01:00` or `2026-11-08T17:30:00Z`. Bounds are inclusive and normalized to UTC. End defaults to query time. Start defaults to one hour before the effective end. Reject an inverted interval.
- Returns the last 5,000 matches in collector insertion order, with `filters`, `selected_count`, `returned_count`, `omitted_count`, `truncated` and `entries`. Entries contain `timestamp`, `level`, `level_name`, `logger_name` and `message`. There is no pagination or maximum query duration.
- Use effective bounds and counts when summarizing. For a truncated or oversized result, use narrower fixed time windows where needed and account for overlapping inclusive boundaries. Retention, eviction and restarts limit completeness even when `truncated` is false. Filtering uses the thermostat name and includes untagged messages, so individual entries may have uncertain attribution.

### `versatile_thermostat__get_plugins`

Pass only `entity_id`. The tool reads configured links and diagnostics already published for the thermostat, without refreshing, calculating or issuing commands. It checks SmartPI, Hysteresis, Pellet Regulation, Auto-fan, Heating Failure Detection, Climate Replication and Inhibition. Discovery does not cover all third-party or future plugins.

Read `plugins`, `unresolved_links`, `coverage`, `diagnostics_status` and per-plugin facts and diagnostic sources. Keep configuration, algorithm selection, loading, availability and activity distinct. Preserve unknown producers and identities, source publication timestamps, and partial or unavailable evidence. A null `configured_plugin_count` is unknown. Use `identified_configured_plugin_count` for the identified subset when present. A logical Hysteresis activity flag does not prove physical heating.

For either tool, report structured `errors` when returned, including target or collector failures, and respect native refusals. See [diagnostic-reference.md](diagnostic-reference.md) for interpretation.

## Write tools

All write tools are non-read-only, non-destructive, and idempotent. The three climate write tools below use standard Home Assistant `climate` services and require an authenticated user context with permission to control the entity.

### `versatile_thermostat__set_temperature`

Parameters:

```json
{
  "entity_id": "climate.thermostat_salon",
  "temperature": 19.5
}
```

Uses `climate.set_temperature`. The tool rejects targets outside the VTherm min/max range when those limits are available.

### `versatile_thermostat__set_preset`

Parameters:

```json
{
  "entity_id": "climate.thermostat_salon",
  "preset": "comfort"
}
```

Uses `climate.set_preset_mode`. The requested preset must be present in `preset_modes`.

### `versatile_thermostat__set_hvac_mode`

Parameters:

```json
{
  "entity_id": "climate.thermostat_salon",
  "hvac_mode": "heat"
}
```

Uses `climate.set_hvac_mode`. The requested mode must be present in `hvac_modes`.

A successful write result contains:

```text
success
action
requested
state_after_call
note
```

`state_after_call` is immediate state after the service call. VTherm can later recalculate or override effective behavior according to its configured rules.

## Common examples

French:

```text
Quels VTherm sont disponibles ?
Pourquoi le thermostat du salon ne chauffe pas ?
Le salon est-il délesté ?
Donne-moi l'état complet de climate.thermostat_salon.
Mets climate.thermostat_salon à 19,5 °C.
Passe le salon en preset comfort.
Éteins le VTherm du salon.
```

English:

```text
List the available VTherms.
Why is the living-room VTherm idle?
Is this thermostat being power-shed?
Show the complete state of climate.thermostat_salon.
Set climate.thermostat_salon to 19.5 C.
Set the living-room preset to comfort.
Set the living-room VTherm HVAC mode to off.
```

## Errors and fallbacks

If a requested entity is not exposed to Assist, the dedicated tools will not accept it. Report the limitation. Never bypass a dedicated VTherm tool's refusal with a generic Home Assistant service, another server, or an indirect API call to obtain the same result.

If a preset or HVAC mode is rejected, show the allowed values returned by the tool or obtain them from `list_vtherms`/`get_status`.

If a temperature is rejected, report the VTherm min/max returned by the error.

If a write lacks an authenticated Home Assistant user context or permission, report the permission error rather than attempting another route.

## TPI tools

Implementation maintenance: `TPIHistory` owns each thermostat’s HA context/sources and `TPIIdentification` owns its grid and analysis resolution. Time-grid sampling belongs to `Grid.sample_previous`. The tool names, input schemas, numerical methods and write-confirmation workflow are preserved. Current development uses the `llm/` package (`__init__.py`, `diagnostics.py`, `diagnostic_time.py`, plus `central_configuration.py` with PR #2232), replacing the older `llm.py`. Manual updates must keep that package, `recorder_export.py`, `tpi_history.py` and `tpi_identification.py` on a compatible revision; follow the documentation for the installed version.

Natural questions about “optimal”, “well tuned”, “optimaux”, “adaptés” or “bien réglés” coefficients should select `check_tpi_optimization`, then `analyze_tpi_coefficients` if eligible, without a write. Use `list_vtherms` to resolve the room. Browser access and a technical user prompt are not prerequisites. A connector missing these tools cannot perform the calculation even if another client exposes them.

- `versatile_thermostat__check_tpi_optimization`: `{ "entity_id": "climate.thermostat_salon" }`. Read-only, stage 1/3. Resolves sources and Recorder coverage without OLS/bootstrap. Returns eligibility, current coefficients, `sources` and `history` (`history_requested_days`, `history_available_days`, `history_usable_days`).
- `versatile_thermostat__analyze_tpi_coefficients`: same input, read-only stage 2/3. Returns `model`, validation MAE/RMSE/ΔT R², `proposed`, current comparison, `bootstrap`, `multi_resolution`, `median_coef_ext`, `span_coef_ext`, `relative_span`, convergence table and warnings. Also returns `estimate_quality`, `proposal_estimator`, `experimental_estimator`, `diagnostics`, `current_nominal_response`, `proposed_nominal_response` and `applicability_note` (FR/EN). At least 7 usable days, last 14 days maximum. Block bootstrap: 1000 repetitions, 24 h, seed 20261030. No automatic application.
- `versatile_thermostat__set_tpi_coefficients`: `{ "entity_id": "climate.thermostat_salon", "coef_int": 0.35, "coef_ext": 0.025 }`. Explicit write, stage 3/3. One or both finite values; coef_int 0–10, coef_ext 0–1. No history required for an explicit direct change. Calls `versatile_thermostat.set_tpi_parameters`, never edits ConfigEntry directly. Returns `previous`, `requested`, `applied`.

Analysis supports Celsius classic TPI over physical switches (single/multiple, inverse switch supported); all relay histories are required. Duty is reconstructed from actual ON durations with equal `P/N` shares as in VTherm, not guessed from HVAC action or current on_percent. Virtual commands, SmartPI and Auto-TPI are refused. [PR #2233](https://github.com/jmcollin78/versatile_thermostat/pull/2233), open and unmerged as checked on 12 November 2026, proposes experimental `over_climate_valve` analysis using the recorded common `valve_open_percent` command divided by 100 (`heating_source_kind: valve_open_percent_command_proxy`). Commands are integrated over each minute; missing/nonfinite/out-of-range values invalidate intervals and are never zero-filled or clipped. This is not physical valve feedback, measured flow or thermal power. Explain `heating_proxy_note`: valve nonlinearity, thresholds, water temperature, boiler availability, balancing and delays can bias the pair. Plain `over_climate` and `over_valve` remain unsupported. Older servers may still return `unsupported_thermostat_type` for valve-regulated climates; respect the refusal. No release or live catalog availability is implied. A direct coefficient write remains possible for classic TPI types supported by the existing service.

For both switches and valve-regulated climates, describe every estimate as experimental and requiring verification before application. New check/analysis responses include `experimental: true`, `verification_required_before_application: true`, bilingual `experimental_notice` and warning `experimental_coefficients_verify_before_application`; valve responses also include `valve_command_is_not_measured_heat`. Reads require current exposure and authenticated `READ` before resolving historical sources. Successful estimates have `estimate_quality: exploratory`, `proposal_estimator: ols_reference` and `recommendation_reliable: false`. Bootstrap percentile intervals are approximate and uncalibrated, with `measurement_noise_bias_corrected: false`; they do not guarantee parameter accuracy. `experimental_estimator` reports `status: unavailable`, `reason: experimental_model_not_validated_for_application` and `promoted_to_proposal: false`. The Kalman estimator is disabled in the MCP analysis, with no calculation or coefficient output, because simulations still show parameter error under model mismatch. Its implementation remains for offline research only; do not recommend values returned by older servers as validated settings.

For offline research only, Kalman screening requires at least 40 fresh training observations and 10 fresh validation observations, training heater standard deviation at least 0.05 and OLS design condition number below 1000. Both initializations must converge with finite results and yield coefficients in the service ranges (internal 0–10, external 0–1). Thermal g/B parameters within 5% of a search bound or relative coefficient span above 25% across starts cause refusal. These are conservative policies, not calibrated reliability tests. Reasons include `latent_insufficient_excitation`, `latent_optimizer_not_converged`, `latent_coefficients_out_of_range`, `latent_thermal_parameter_near_bound`, `latent_initialization_sensitive` and insufficient fresh observations. An unavailable estimate returns a reason without Kalman coefficients; never clip or reconstruct them. The OLS result remains exploratory even if Kalman is unavailable. Statistical bootstrap intervals are not clipped to service limits.

The preferred 6 h is a nominal time constant chosen by design, not an identified optimum; 95% correction takes about 18 h in the local unsaturated model. `current_nominal_response` and `proposed_nominal_response` describe that model, not guaranteed room response. `diagnostics` reports training excitation, observed temperature steps, held-out residual correlations and 1/3/6-hour free-run RMSE with run counts. These free runs retain recorded weather and heater inputs; they do not simulate applying the proposed coefficients. Observed steps do not establish sensor resolution.

Include `applicability_note` in the user's language: estimates come from the analyzed history and suitability under other conditions has not been established. Consider rerunning on representative data after substantial heating/household-use changes or degraded regulation, before considering adjustments. Outdoor temperature changes alone do not require recalculation.

Safety/window/power exclusion sensors must be recorded. Missing required diagnostic histories cause refusal, not an assumption that protection was off. No future temperature interpolation; causal hold limited to 60 min. Binary states persist until the next event, including unknown/unavailable invalidation.

Large change: absolute delta ≥0.15 (coef_int) or ≥0.01 (coef_ext), or relative delta ≥50%; any nonzero change from zero is large. The tool first returns `confirmation_required: true` and a token without writing. After additional user confirmation, pass `confirm_large_change: true`, the exact returned `confirmation_token`, and the same values. Tokens expire in 10 minutes and are bound to user/entity/previous/requested values. Never confirm without the user's decision.

Each result contains `stage`, `stage_number`, `stage_total`, `state_changed`. A refusal before writing states no coefficient was changed. After service/readback failure, `state_changed: null` means unknown application; re-read before retrying. MCP Read only retains both read tools and hides the write tool.


## `versatile_thermostat__get_history`

Natural requests for a professional/annotated thermostat graph over 48 h use this historical data tool when available and client-side chart creation, not unsolicited YAML for a dashboard card. All types, including SmartPI, are supported. Only annotate recorded events; neither historical SmartPI power nor physical relay history is included. Do not reconstruct those missing series from current state. When this native tool is absent, the supported ha-mcp history route may be used under the routing rules; if no history tool is usable, report the limitation. Never invent data or replace the requested chart with configuration instructions.

Read-only climate entity history for client-side CSV/charts. Input: `entity_id`, `start_time`, `end_time` (timezone-aware ISO 8601; maximum 14 days per period), `offset` (default 0), `limit` (default/max 500). Fetch all pages using `next_offset` and fixed bounds until null. Returns `rows` (`timestamp`, `state`, recorded `attributes`), `total_rows`, `next_offset`, `pagination_complete`, UTC bounds and semantics. Attributes: current_temperature, temperature, target_temp_low/high, hvac_action, preset_mode, unit_of_measurement. Missing attributes are null. Current Assist exposure and authenticated READ permission are checked at every call. All VTherm types; no TPI eligibility requirement. Associated sensor/relay histories are not included. Pagination is not a frozen snapshot. Retention/exclusions/outages can limit history. No HA writes or generated files on HA; the client creates the requested artifact when supported.
