# Troubleshooting Notes — Lab 113: Wazuh Screensaver Persistence

## 1. Objective

Document the troubleshooting performed while validating Sysmon Event ID 13 collection and investigating whether the resulting event reached Wazuh.

The approach prioritizes read-only checks, one targeted configuration change, and verification at each stage.

## 2. Issue: Registry Test Event Was Not Initially Visible

### Symptoms

A registry value was created under:

`HKCU:\Software\WazuhLabEvent13Test`

However, the expected Sysmon Event ID 13 was not initially found in the local event query.

### Investigation

The active Sysmon configuration was inspected at:

`C:\Tools\Sysmon\sysmonconfig.xml`

The registry-event include block was located under:

`EventFiltering > RuleGroup > RegistryEvent`

The XPath used to locate the block was:

`//EventFiltering/RuleGroup/RegistryEvent[@onmatch="include"]`

The include block contained 113 `TargetObject` rules before the test rule was added.

An existing screensaver-related rule matched registry target objects containing `\Control Panel\Desktop`.

### Resolution

A targeted rule was added to the existing include block:

`<TargetObject condition="contains">WazuhLabEvent13Test</TargetObject>`

A timestamped backup was created:

`C:\Tools\Sysmon\sysmonconfig.backup-20261021-080929.xml`

Sysmon was then instructed to load the updated configuration and returned:

- Configuration file validated.
- Configuration updated.

A new registry modification was generated, and a matching local Event ID 13 was subsequently confirmed.

### Lesson

When a targeted Sysmon event is missing, inspect the active filtering configuration before changing services or rebuilding the monitoring environment.

## 3. Issue: Backup Filename Already Existed

### Symptoms

The initial PowerShell script stopped because the designated backup file already existed.

### Resolution

The script was adjusted to use a timestamped backup filename and to stop if that filename was already present.

The existing backup was not overwritten.

The subsequent configuration change completed successfully, and the resulting XML rule was verified.

### Lesson

Configuration changes should fail safely when a backup destination already exists. Preserve the existing backup and choose a distinct filename rather than overwriting it.

## 4. Issue: Confirming the Sysmon Configuration Change

### Validation

The following command was used:

`& "C:\Tools\Sysmon\Sysmon64.exe" -c "C:\Tools\Sysmon\sysmonconfig.xml"`

Observed output:

- Loading configuration file with schema version `4.50`.
- Sysmon schema version `4.91`.
- Configuration file validated.
- Configuration updated.

### Result

Sysmon accepted and applied the updated configuration.

The local registry test then produced a matching Event ID 13.

## 5. Issue: Wazuh Agent Log Search Returned No Matches

### Symptoms

The Wazuh Agent service check showed:

- Status: `Running`
- Startup type: `Automatic`

A search of the last 60 lines of `ossec.log` returned no matching lines for the selected connection and error keywords.

### Interpretation

The result is inconclusive. The agent could be running while the inspected log segment contains no matching messages.

The absence of matching lines does not independently prove a connectivity failure.

### Next Checks

- Inspect a broader portion of the agent log using PowerShell.
- Look for timestamps and explicit connection or error messages.
- Check whether the exact test marker appears in Wazuh.
- If it is absent, validate event collection and delivery before modifying configuration.

Do not restart the agent merely because a keyword search returned no output.

## 6. Issue: Wazuh Ingestion Not Confirmed

### Known Evidence

The following facts were established:

- Sysmon recorded the test modification locally.
- The Wazuh Agent service was running.
- The Wazuh dashboard displayed an alerts index for the current date.
- The available dashboard screenshot did not conclusively demonstrate ingestion of the exact test event.

### Important Distinction

These are separate stages:

1. The registry value changes.
2. Sysmon records Event ID 13.
3. The Wazuh Agent collects the event.
4. The event reaches the Wazuh manager.
5. The manager processes and indexes the event.
6. An alert may be generated if applicable rules match.

Evidence of one stage should not be treated as proof of all subsequent stages.

### Next Checks

- Search the appropriate index for `WazuhLabEvent13Test`.
- Use a time range covering the test at approximately `2026-10-21 08:10:57` local time.
- If the marker is absent, determine whether raw archives are available.
- If raw data is present but alerts are absent, investigate rule matching separately.
- Avoid assuming that every Sysmon event must generate a Wazuh alert.

## 7. Troubleshooting Principles

- Use PowerShell for Windows endpoint investigation.
- Prefer read-only validation before making changes.
- Preserve the active configuration before editing it.
- Make one targeted change at a time.
- Validate XML before applying it.
- Verify local event generation before troubleshooting SIEM ingestion.
- Do not restart services without evidence that a restart is necessary.
- Treat missing search results as inconclusive until the relevant data source and time range have been validated.
- Keep confirmed observations separate from hypotheses.

## 8. Current Troubleshooting Status

| Check | Status |
|---|---|
| Active Sysmon configuration located | Complete |
| Existing registry include block identified | Complete |
| Existing backup preserved | Complete |
| Targeted test rule added | Complete |
| Sysmon configuration validated and applied | Complete |
| Registry test value written | Complete |
| Local Sysmon Event ID 13 confirmed | Complete |
| Wazuh Agent service checked | Complete |
| Exact event found in Wazuh | Pending |
| Root cause of any ingestion gap | Undetermined |

## 9. Conclusion

The initial local collection issue was resolved by adding a targeted registry-event filter to the existing Sysmon configuration.

The remaining task is to validate delivery and indexing of the same event in Wazuh. No additional configuration changes should be made until the missing stage, if any, is identified.
