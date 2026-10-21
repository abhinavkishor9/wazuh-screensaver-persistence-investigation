# Investigation Timeline — Lab 113: Wazuh Screensaver Persistence

All local timestamps below use the Windows endpoint's local time. UTC timestamps are included where present in the source event messages.

## 1. Timeline of Observed Activity

| Local timestamp | UTC timestamp | Event or action | Result |
|---|---|---|---|
| 2026-10-21 06:45:49 | 2026-10-21 01:15:49.214 | Existing Sysmon Event ID 13, Record ID `1494014` | Inventory-related registry modification observed; the supplied event excerpt was incomplete |
| 2026-10-21 06:54:45 | 2026-10-21 01:24:45.838 | Existing Sysmon Event ID 13, Record ID `1494311` | McAfee-related registry value modification observed |
| 2026-10-21 06:54 onward | Not available | Screensaver persistence investigation and registry inspection | `ScreenSaveActive` observed as `1`; other inspected values were blank |
| During configuration review | Not available | Active Sysmon registry-event include block inspected | 113 `TargetObject` rules identified before the targeted rule was added |
| During configuration update | Not available | Existing backup collision encountered | Initial script stopped safely without changing the configuration |
| 2026-10-21 08:09:29 | Not applicable | Timestamped backup created | `sysmonconfig.backup-20261021-080929.xml` preserved the pre-change configuration |
| Before 08:10:57 | Not available | Targeted `WazuhLabEvent13Test` include rule added | XML rule saved and verified |
| Before 08:10:57 | Not available | Sysmon configuration validation and application | Sysmon reported that the configuration was validated and updated |
| 2026-10-21 08:10:57 | 2026-10-21 02:40:57.726 | Registry value `LabMarker` written | Value `Event13-Test-20261021-081057` created or updated |
| 2026-10-21 08:10:57 | 2026-10-21 02:40:57.726 | Sysmon Event ID 13, Record ID `1496536` | Targeted registry modification confirmed locally |
| After the local test | Not available | Wazuh Agent service checked | `WazuhSvc` was running and configured for automatic startup |
| After the local test | Not available | Agent log searched for connection and error keywords | No matching lines were returned from the last 60 lines inspected |
| After the local test | Not available | Wazuh dashboard context reviewed | An alerts index was visible, but the exact test event was not confirmed as indexed |

## 2. Key Event Details

### Existing McAfee-Related Event

- Event ID: `13`
- Record ID: `1494311`
- Target: `HKLM\SOFTWARE\McAfee\wps\etw\volatile\last-prune-time`
- Process: `mc-fw-host.exe`
- User: `NT AUTHORITY\SYSTEM`

The event establishes that a registry value modification was recorded. It does not establish malicious intent.

### Targeted Registry Test

- Event ID: `13`
- Record ID: `1496536`
- Process: PowerShell (`pwsh.exe`)
- Process ID: `19980`
- Target: `HKU\S-1-5-21-51198790-337801975-3228388354-1001\Software\WazuhLabEvent13Test\LabMarker`
- Test value: `Event13-Test-20261021-081057`

The event confirms local Sysmon collection of the controlled registry modification.

## 3. Milestones

1. Screensaver-related registry values inspected.
2. Existing registry telemetry reviewed.
3. Sysmon registry-event filtering examined.
4. Backup collision handled without overwriting the existing backup.
5. Targeted registry filter added and verified.
6. Sysmon configuration validated and applied.
7. Controlled registry modification generated.
8. Matching local Event ID 13 confirmed.
9. Wazuh ingestion left pending further verification.

## 4. Current State

**Confirmed:** The Windows endpoint generated the expected Sysmon Event ID 13 after the targeted filter was added.

**Not yet confirmed:** The exact event was received and indexed by Wazuh.

The next milestone is to establish the event's presence or absence in the relevant Wazuh data source and investigate the corresponding stage of the telemetry pipeline.
