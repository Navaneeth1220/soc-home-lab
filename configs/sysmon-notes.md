# Sysmon configuration

**Sysmon version:** 15.22
**Config in use:** `sysmonconfig.xml` from [olafhartong/sysmon-modular](https://github.com/olafhartong/sysmon-modular), downloaded from the project's releases.

```powershell
Invoke-WebRequest https://github.com/olafhartong/sysmon-modular/releases/latest/download/sysmonconfig.xml -OutFile sysmonconfig.xml
.\Sysmon64.exe -accepteula -i sysmonconfig.xml
```

## Why sysmon-modular

- Every rule is tagged with a MITRE ATT&CK technique (`RuleName: technique_id=...`), which carries through to Splunk and makes detections easier to map.
- It logs process access to `lsass.exe` (EID 10), needed for credential-access detections.

## Include-based filtering and what it means

For process creation (EID 1) the config is include-based: a process start is logged only when it matches a technique rule. Observed behaviour:

| Action | Logged? | Why |
|---|---|---|
| `cmd /c whoami` | Yes | Matches T1059.003 (command shell) and discovery rules |
| `notepad` launched from PowerShell | No | No rule treats this as suspicious |
| OneDrive / Edge WebView starting from user paths | Yes | Execution from user-writable locations is monitored |

This keeps volume low and resembles a production deployment. The trade-off is that a technique without a matching rule produces no EID 1 event, so every detection must be validated by generating the activity and confirming the event exists. Missing events are recorded as either a coverage gap (add an include rule) or correct filtering.

## Rejected alternative: `sysmonconfig-excludes-only.xml`

The excludes-only variant (log everything except known noise) was tested for broader visibility. After loading it, process creation events (EID 1) stopped entirely while process termination events (EID 5) continued, including after a full uninstall, reboot and reinstall. The root cause was not identified; reverting to `sysmonconfig.xml` restored EID 1 immediately. See [`../troubleshooting.md`](../troubleshooting.md).
