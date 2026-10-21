# Investigation Notes 
## 1. Screensaver Registry Inspection

The following registry values were inspected:

- `SCRNSAVE.EXE`
- `ScreenSaveActive`
- `ScreenSaveTimeOut`
- `ScreenSaverIsSecure`

Observed output:

| Registry value | Observed value |
|---|---|
| `SCRNSAVE.EXE` | Blank |
| `ScreenSaveActive` | `1` |
| `ScreenSaveTimeOut` | Blank |
| `ScreenSaverIsSecure` | Blank |

### Interpretation

The observed `ScreenSaveActive` value indicates that the screensaver activation setting was enabled.

The other inspected values were blank in the captured output. Their blank state does not establish that all screensaver settings are absent or that a persistence mechanism is present.

Additional context would be required to determine the configured screensaver executable, timeout, security behavior, and whether any setting was modified unexpectedly.

## 2. Existing Sysmon Event Review

Two existing Event ID 13 records were visible in the supplied evidence.

### Event A — McAfee Registry Update

- Local timestamp: `2026-10-21 06:54:45`
- UTC timestamp: `2026-10-21 01:24:45.838`
- Event ID: `13`
- Record ID: `1494311`
- Process: `mc-fw-host.exe`
- Process path: `C:\Program Files\McAfee\wps\1.40.161.1\mc-fw-host.exe`
- Target object: `HKLM\SOFTWARE\McAfee\wps\etw\volatile\last-prune-time`
- User: `NT AUTHORITY\SYSTEM`
- Rule name: `Suspicious,ImageBeginWithBackslash`

**Assessment:** The record documents a registry value modification by a McAfee-related process. The process name, path, and target are useful investigation context, but the event alone does not establish malicious behavior.

### Event B — Inventory-Related Registry Update

- Local timestamp: `2026-10-21 06:45:49`
- UTC timestamp: `2026-10-21 01:15:49.214`
- Event ID: `13`
- Record ID: `1494014`
- Rule name: `InvDB`

The supplied evidence contains only part of this event's message. The process and target details should be recovered from the original event if further analysis is necessary.

**Assessment:** The available excerpt confirms another registry value modification was recorded. It does not provide enough information to determine the complete activity or its security significance.

## 3. Sysmon Configuration Review

The active configuration was located at:

`C:\Tools\Sysmon\sysmonconfig.xml`

The registry-event include block was identified under the `EventFiltering` section. It contained 113 `TargetObject` rules before the targeted test rule was added.

An existing rule named `Lab113-Screensaver` matched registry target objects containing:

`\Control Panel\Desktop`

The targeted test rule was added to the existing include block:

`<TargetObject condition="contains">WazuhLabEvent13Test</TargetObject>`

The configuration was backed up before the change:

`C:\Tools\Sysmon\sysmonconfig.backup-20261021-080929.xml`

Sysmon subsequently reported:

- Configuration file validated.
- Configuration updated.

## 4. Controlled Registry Test

A test registry value was written under:

`HKCU:\Software\WazuhLabEvent13Test`

The value name was `LabMarker`.

The observed value was:

`Event13-Test-20261021-081057`

### Local Sysmon Evidence

- Event ID: `13`
- Record ID: `1496536`
- Local timestamp: `2026-10-21 08:10:57`
- UTC timestamp: `2026-10-21 02:40:57.726`
- Process: PowerShell (`pwsh.exe`)
- Process ID: `19980`
- Target object: `HKU\S-1-5-21-51198790-337801975-3228388354-1001\Software\WazuhLabEvent13Test\LabMarker`
- Event details: `Event13-Test-20261021-081057`
- User: `DESKTOP-9MMM37V\Dell`

### Assessment

The local Sysmon event confirms that the registry modification was captured after the configuration update.

Sysmon represented the current user's registry location under `HKU\<SID>` in the event's `TargetObject`, rather than using the PowerShell `HKCU:` notation.

This is consistent with the distinction between a PowerShell registry provider path and the target path reported by Sysmon.

## 5. Wazuh Validation

The Windows-side service check returned:

- Service: `WazuhSvc`
- Status: `Running`
- Startup type: `Automatic`

The last 60 lines of the agent log did not produce matching connection, disconnection, error, warning, or port `1514` lines using the selected search expression.

This output is inconclusive. It does not establish that the agent is disconnected because relevant messages may not have been present in the inspected portion of the log.

The supplied dashboard evidence showed an index named:

`wazuh-alerts-4.x-2026.10.21`

The dashboard evidence also showed an agent document associated with agent ID `000` and name `localhost.localdomain`. This identifies the manager-side context visible in that screenshot; it does not prove that the test event was ingested.

### Current Status

- Local Sysmon Event ID 13: Confirmed.
- Wazuh Agent service running: Confirmed.
- Exact test event indexed in Wazuh: Not confirmed.
- Wazuh alert generated for the test event: Not confirmed.

