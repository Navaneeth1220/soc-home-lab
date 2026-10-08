# Configuration

Files are grouped by the machine they belong to.

| File | Machine | Location on that machine |
|---|---|---|
| `splunk-server/netplan-splunk.yaml` | splunk | `/etc/netplan/50-cloud-init.yaml` |
| `windows-endpoint/inputs.conf` | odysseus | `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf` |
| `windows-endpoint/outputs.conf` | odysseus | `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf` |
| `sysmon-notes.md` | odysseus | Notes on the Sysmon configuration in use |

## Splunk server setup (summary)

1. Install Splunk Enterprise `.deb`, enable boot-start as a systemd service running as the `splunk` user.
2. Create the admin account via `user-seed.conf` (not committed).
3. Enable receiving on TCP 9997.
4. Create the index `sysmon`.
5. Install the Splunk Add-on for Sysmon (`Splunk_TA_microsoft_sysmon`).

Manage the service only with `systemctl` (`sudo systemctl restart Splunkd`). Running the `splunk` binary as root creates root-owned files that the service account cannot read.
