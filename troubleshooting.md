# Troubleshooting log

Problems encountered while building the lab, in the order they occurred.

## 1. VirtualBox NAT Network: gateway unreachable

**Symptom:** The Ubuntu VM on a VirtualBox NAT Network (`10.0.2.0/24`) received a DHCP lease, but `ping 10.0.2.1` returned *Destination Host Unreachable* and the VM had no internet access.
**Diagnosis:** A working DHCP lease combined with an unreachable gateway pointed at VirtualBox's NAT Network service rather than the guest configuration. The netplan file and routes were verified correct, and restarting VirtualBox did not help.
**Fix:** Replaced the NAT Network with two adapters per VM: plain NAT for internet access and a host-only adapter for lab traffic. This also removed the need for port forwarding, since the host reaches the VMs directly.

## 2. Host-only adapter with a 169.254.x.x address

**Symptom:** The VirtualBox host-only adapter on the Windows host showed `169.254.42.8/16` (APIPA).
**Fix:** Configured it manually as `192.168.56.1/24`, and set the DHCP server to `192.168.56.2` with a lease range of `.100–.200`, keeping static lab addresses (`.10`, `.20`, `.30`) outside the pool.

## 3. Ubuntu installer used only half the disk

**Symptom:** The default LVM layout allocated 24 GB of a 48 GB volume group to `/`, leaving the rest unused.
**Fix:** Resized `ubuntu-lv` to the full volume group in the installer's storage screen. On an installed system, the equivalent is `sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv`.

## 4. Splunk "Permission denied" errors

**Symptom:** `systemctl status Splunkd` showed permission errors on `users.ini`, `splunkd-utility.log` and temp directories.
**Cause:** `splunk start` had initially been run as root, creating root-owned files. `enable boot-start -systemd-managed 1` then configured the service to run as the `splunk` user, which could not read them.
**Fix:** `sudo chown -R splunk:splunk /opt/splunk`. From then on the service is managed only through `systemctl`.

## 5. Splunk web login: "No users exist"

**Symptom:** The web UI loaded but reported that no users existed.
**Cause:** No admin account was created during first start. A subsequent `user-seed.conf` was ignored because its password was shorter than Splunk's 8-character minimum, and seeding only runs when `/opt/splunk/etc/passwd` does not exist.
**Fix:** Stopped Splunk, removed `etc/passwd`, wrote `user-seed.conf` with a compliant password, set ownership to `splunk`, and started the service. Splunk consumed and deleted the seed file.

## 6. Sysmon config download returned 404

**Symptom:** The raw GitHub URL for `sysmonconfig.xml` in sysmon-modular returned 404.
**Cause:** The project now publishes generated configs as release assets rather than files in the repository.
**Fix:** Download from `https://github.com/olafhartong/sysmon-modular/releases/latest/download/sysmonconfig.xml`.

## 7. Sysmon events indexed but no fields extracted

**Symptom:** `index=sysmon` returned events, but `stats count by EventCode` returned nothing. `xmlkv` could extract `EventID`, so the raw XML was intact.
**Diagnosis:** `stats count by sourcetype, source` showed source `WinEventLog:Microsoft-Windows-Sysmon/Operational`. The add-on's `props.conf` only defines stanzas for `source::XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` and sourcetype `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational`.
**Fix:** Set `source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` in the forwarder's `inputs.conf`. The forwarder ignored the matching `sourcetype` override, but the source match alone was sufficient. Events indexed before the change remain unparsed.

## 8. Process creation logging stopped after a config change

**Symptom:** After switching Sysmon from `sysmonconfig.xml` to `sysmonconfig-excludes-only.xml` (`Sysmon64.exe -c`), no new EID 1 events were written. The service and driver were running and EID 5 (process terminated) continued. An uninstall, reboot and reinstall with the same config did not help.
**Fix:** Reverted to `sysmonconfig.xml`; EID 1 resumed immediately (`cmd.exe` and `whoami.exe` logged). Root cause not determined. See [`configs/sysmon-notes.md`](configs/sysmon-notes.md).
**Lesson:** After any Sysmon config change, confirm EID 1 is still being written before relying on it.

## 9. Splunk health report: Resource Usage red

**Symptom:** The health indicator flagged Resource Usage.
**Cause:** The VM (2 vCPU, 4 GB RAM) is below Splunk's production recommendations. Expected for a lab; disk space, the indicator that would actually stop indexing, was not the trigger.
