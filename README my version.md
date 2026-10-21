# wazuh-screensaver-persistence-investigation
## Overview
### What Is Screensaver Persistence?

Windows stores user-specific screensaver configuration in the Registry. An attacker with sufficient access may modify these settings to reference an unexpected executable. If Windows subsequently launches that executable during screensaver activity, the configuration could provide a mechanism for recurring code execution.

The relevant registry location is:

`HKEY_CURRENT_USER\Control Panel\Desktop`

This location applies to the current user's profile. Other user accounts may have different screensaver configurations.

### How Potential Persistence Works

1. **Registry configuration change:** A screensaver-related value is modified to reference an unexpected executable path.
2. **Screensaver activation:** Windows may launch the configured screensaver when the applicable activation conditions are met.
3. **SOC investigation:** The analyst examines registry modification events, process execution, executable paths, timestamps, and user context to determine whether the activity is suspicious.

A registry value alone does not prove that persistence is active. The configured path may be legitimate, the executable may not exist, or the setting may never be used.

### Registry Values to Understand

| Registry Value | Meaning |
|---|---|
| `SCRNSAVE.EXE` | Path configured for the screensaver executable. |
| `ScreenSaveActive` | Indicates whether screensaver activity is enabled by the setting. |
| `ScreenSaveTimeOut` | Inactivity timeout before screensaver activation, typically represented in seconds. |
| `ScreenSaverIsSecure` | Indicates whether authentication is required when resuming, subject to Windows policy and configuration. |


This lab investigates Windows screensaver-related registry settings using Sysmon and Wazuh. The objective is to understand how registry value modifications can be monitored, how Sysmon Event ID 13 records these changes, and how endpoint telemetry can be validated before drawing conclusions from SIEM data.

The investigation also examines the distinction between observing a registry configuration and establishing that a persistence technique was successfully implemented.

## Lab Objectives

- Examine Windows screensaver-related registry settings and identify the configuration values relevant to a persistence investigation.
- Analyze Sysmon Event ID 13 records to understand how registry value modifications are captured and represented in endpoint telemetry.
- Review existing Sysmon registry-event filtering rules and determine why a targeted registry modification may not appear in the expected event search.
- Safely modify the active Sysmon configuration by introducing a narrowly scoped registry monitoring rule while preserving the original configuration.
- Validate the updated Sysmon configuration and confirm that the intended registry modification generates a corresponding local event.
- Correlate the test registry value, process information, event timestamp, record ID, and target object to establish a reliable evidence trail.
- Investigate the distinction between local event generation, Wazuh Agent operation, manager-side receipt, event indexing, and alert generation.
- Troubleshoot telemetry visibility using a staged, evidence-driven approach without unnecessary service restarts or unrelated configuration changes.
- Document confirmed findings, unresolved questions, troubleshooting decisions, and limitations without treating registry configuration alone as proof of malicious persistence.
  
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

## Lab Scenario

A Windows endpoint is undergoing a security investigation to determine how screensaver-related registry modifications can be monitored and how effectively endpoint telemetry is delivered to a centralized SIEM. Screensaver configuration can be relevant when investigating persistence techniques, but registry values must be interpreted in context rather than automatically classified as malicious.

During the initial review, the investigator examines the registry values under `HKCU:\Control Panel\Desktop` and reviews existing Sysmon Event ID 13 records. The available events demonstrate that registry modifications are being recorded, but the evidence does not establish that the observed activity is malicious or that a screensaver-based persistence mechanism has been configured.

The investigation then encounters a telemetry visibility issue: a controlled registry modification is not initially found in the expected local event search. The investigator reviews the active Sysmon configuration, identifies the existing registry-event include rules, and introduces a narrowly scoped filter for a dedicated test registry path. The original configuration is preserved before the change, and the updated configuration is validated before generating another test event.

The subsequent registry modification produces a matching Sysmon Event ID 13, confirming that the targeted activity is captured locally. The investigation then moves to Wazuh, where the agent service is confirmed to be running. However, the available evidence does not conclusively establish whether the exact test event has reached the manager and been indexed.

The task is to document the investigation from registry inspection through local event validation, assess the available evidence, and identify the remaining checks required to verify end-to-end telemetry delivery.

### Investigation Focus

- Inspect screensaver-related registry values and evaluate their significance without assuming malicious activity.
- Review Sysmon registry telemetry and troubleshoot missing events through targeted configuration analysis.
- Generate a controlled registry modification and verify the corresponding local Sysmon event.
- Investigate Wazuh telemetry visibility while distinguishing agent status from confirmed event ingestion.
- Record confirmed findings, unresolved questions, and evidence limitations in a structured investigation report.
  
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

