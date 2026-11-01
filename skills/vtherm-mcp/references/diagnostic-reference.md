# VTherm diagnostic reference

## Contents

- Diagnostic evidence order
- Returned sections
- Documentation resources
- Escalation to logs
- Response style

## Diagnostic evidence order

Use VTherm's own reason fields before general inference:

1. `specific_states.messages` exposed as `reasons.messages`
2. `hvac_off_reason`
3. `hvac_mode_reason`
4. `should_device_be_active`
5. unavailable underlying entities
6. relevant feature manager section
7. temperatures and other state context

Do not treat `target_temperature > current_temperature` as proof that the underlying heater should currently be on. VTherm can intentionally remain idle/off because of scheduling, power, window, safety, central mode, auto-start/stop, regulation, or device timing.

When the returned evidence conflicts or is incomplete, state what is known and what is not established.

## Returned sections

`diagnose` can return these VTherm sections when present:

```text
configuration
current_state
requested_state
specific_states
preset_temperatures
presence_manager
motion_manager
power_manager
window_manager
safety_manager
auto_start_stop_manager
lock_manager
heating_failure_detection
vtherm_over_switch
vtherm_over_climate
vtherm_over_valve
```

Use only sections present in the response. Do not assume a feature is configured merely because a section name exists in this reference; check `feature_flags` and returned content.

## Documentation resources

French documentation:

- Main: https://www.versatile-thermostat.org/fr/
- Reference: https://www.versatile-thermostat.org/fr/docs/reference/
- Log collector: https://www.versatile-thermostat.org/fr/docs/feature-logs-collector/

English documentation:

- Main: https://www.versatile-thermostat.org/en/
- Reference: https://www.versatile-thermostat.org/en/docs/reference/
- Log collector: https://www.versatile-thermostat.org/en/docs/feature-logs-collector/

Official repository:

- https://github.com/jmcollin78/versatile_thermostat

Log analyzer:

- https://www.versatile-thermostat.org/fr/debugger/

The `diagnose` result also returns these URLs in `support_resources`; prefer the returned URLs if they differ from this static reference.

## Escalation to logs

If the current state cannot establish the cause:

1. Explain which evidence is missing or ambiguous.
2. Suggest collecting VTherm logs using the Home Assistant action `versatile_thermostat.download_logs`.
3. Suggest loading the resulting file into the VTherm log analyzer.
4. Do not execute the log export unless the user explicitly asks for it.
5. Do not upload the log to an external site without explicit user authorization.

Example Home Assistant action for the user to run when appropriate:

```yaml
action: versatile_thermostat.download_logs
data:
  entity_id: climate.thermostat_salon
  log_level: DEBUG
```

## Response style

For a diagnosis, keep the conclusion evidence-based:

```text
Conclusion: <what VTherm is doing>
Cause indicated by VTherm: <message/reason fields>
Relevant context: <feature section and important values>
Confidence: established / likely / inconclusive
Next step: <only if needed>
```

Answer in French for French questions and English for English questions. Keep Home Assistant/VTherm raw enum values in code formatting when useful.
