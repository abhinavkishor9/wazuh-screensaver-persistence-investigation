# Lab 113 — Wazuh Screensaver Persistence Investigation

## Overview

This lab investigates Windows screensaver-related registry settings using Sysmon and Wazuh. The objective is to understand how registry value modifications can be monitored, how Sysmon Event ID 13 records these changes, and how endpoint telemetry can be validated before drawing conclusions from SIEM data.

The investigation also examines the distinction between observing a registry configuration and establishing that a persistence technique was successfully implemented.

## Objectives

- Inspect Windows screensaver-related registry settings.
- Review Sysmon Event ID 13 records for registry value modifications.
- Examine the existing Sysmon registry-event filtering configuration.
- Validate whether a targeted registry modification produces a local Sysmon event.
- Investigate the telemetry path from the Windows endpoint to Wazuh.
- Document configuration changes, troubleshooting steps, and evidence limitations.

## Environment

| Component | Details |
|---|---|
| Endpoint OS | Windows 11 |
| Endpoint monitoring | Sysmon 15.21 |
| SIEM / agent | Wazuh Agent 4.12.0 |
| Registry monitoring event | Sysmon Event ID 13 |
| Sysmon configuration | `C:\Tools\Sysmon\sysmonconfig.xml` |
| Screensaver registry path | `HKCU:\Control Panel\Desktop` |
| Test registry path | `HKCU:\Software\WazuhLabEvent13Test` |
| Wazuh dashboard index observed | `wazuh-alerts-4.x-2026.10.21` |

## Investigation Scenario

A Windows endpoint is being examined for screensaver-related registry configuration changes that could be relevant to persistence investigations.

The investigation begins by inspecting the screensaver settings and reviewing existing Sysmon registry telemetry. Existing Event ID 13 records include a McAfee-related registry update and an inventory-related registry update. These records demonstrate registry monitoring activity, but neither is sufficient by itself to establish malicious behavior.

A dedicated test registry value is then created under `HKCU:\Software\WazuhLabEvent13Test`. The Sysmon configuration is updated with a targeted registry filter so that the test modification can be validated independently of unrelated registry activity.

The resulting event is checked locally before investigating whether Wazuh has received and indexed the same telemetry.

## Investigation Workflow

1. Inspect screensaver-related registry values.
2. Review existing Sysmon Event ID 13 records.
3. Examine the active Sysmon registry-event include rules.
4. Preserve the original Sysmon configuration before making a targeted change.
5. Add a `TargetObject` include rule matching `WazuhLabEvent13Test`.
6. Validate and apply the Sysmon configuration.
7. Create or update the test registry value.
8. Confirm the resulting Event ID 13 in the local Sysmon Operational log.
9. Check Wazuh telemetry without assuming that local event generation guarantees SIEM ingestion.

## Key Findings

- `ScreenSaveActive` was observed with a value of `1`.
- `SCRNSAVE.EXE`, `ScreenSaveTimeOut`, and `ScreenSaverIsSecure` were blank in the inspected output.
- Existing Sysmon Event ID 13 records demonstrated that registry value changes were being logged.
- The active Sysmon configuration contained an existing screensaver-related filter named `Lab113-Screensaver`.
- A targeted `WazuhLabEvent13Test` registry filter was added and verified in the XML configuration.
- Sysmon accepted the updated configuration.
- The test registry modification generated a local Sysmon Event ID 13.
- Wazuh Agent service status was `Running`.
- End-to-end ingestion of the exact test event into Wazuh remained unconfirmed at the end of the documented checks.

## Evidence Summary

| Evidence | Observation | Interpretation |
|---|---|---|
| Screensaver registry inspection | `ScreenSaveActive = 1` | Screensaver activation setting was enabled |
| Existing Sysmon event | Event ID 13, Record ID `1494311` | A McAfee process changed a registry value |
| Existing Sysmon event | Event ID 13, Record ID `1494014` | An inventory-related registry value change was recorded |
| Sysmon configuration | Targeted test filter present | The intended filter was added to the include block |
| Sysmon configuration update | Validation successful | Sysmon accepted the updated configuration |
| Test registry event | Event ID 13, Record ID `1496536` | The targeted registry modification was captured locally |
| Wazuh Agent | Service running | The Windows agent service was active |
| Wazuh ingestion | Not conclusively verified | Further SIEM-side validation is required |

## Important Limitations

- A registry value or configuration state is not proof of malicious persistence.
- `ScreenSaveActive = 1` does not independently establish that a screensaver executable is configured or that persistence was achieved.
- Blank values in the inspected output do not establish the absence of all screensaver-related configuration.
- A Sysmon event generated locally does not prove that Wazuh received, indexed, or alerted on it.
- The presence of a document in `wazuh-alerts-*` does not mean every Sysmon event is represented there.
- The investigation does not establish that the observed McAfee activity was malicious.

## Conclusion

The investigation successfully validated local Sysmon collection of a targeted registry value modification after updating the registry-event filtering configuration.

The next validation requirement is to establish whether the same event reaches Wazuh and, if it does not, identify the point in the telemetry pipeline where collection or indexing needs further investigation.

The current evidence supports successful local event generation, but not a confirmed end-to-end Wazuh ingestion result.
