# Lab 7: Time Synchronisation with Chrony

## Objective and network
Configure a lab time server, point a client at it, and decide from evidence whether synchronisation works. NTP means Network Time Protocol. chronyd is the clock service; chronyc is its monitoring tool. A running service alone does not show a synchronised clock.
Reuse the Ubuntu Server 24.04 VMs from Labs 5-6: server 192.168.11.10/24 and client 192.168.11.20/24 on the same host-only network. Both retain NAT. Run Linux commands inside the stated VM, using its console and a named administrative account.
Time path: upstream source <-> server .10 (via NAT); server .10 <-> client .20 (NTP UDP 123 over host-only network). Both VMs retain separate NAT adapters.
## Files and records
Extract the resource ZIP. Use setup-record.md for interfaces, services, source files, backups and changes. Use observations.csv for measurements. SOURCE-SETUP.md gives the detailed client edit; RESTORE.md gives matching recovery steps.
## 1. Inspect both machines
On BOTH VMs:
```sh
hostname
ip -br address
ip route
timedatectl
systemctl status chrony systemd-timesyncd --no-pager
```
A missing unit is information; record the actual result. Note the current time service and timezone. Identify lab interfaces using the MAC records from Lab 6. The server needs a usable upstream time source through NAT; correct local addresses alone do not supply time.
On the CLIENT:
```sh
ip route get 192.168.11.10
ping -c 4 192.168.11.10
```
Resolve a broken Lab 6 baseline before changing time configuration. Ping tests ICMP, not NTP. Do not change addresses or gateways for this lab.


## 2. Prepare chrony and take backups
On BOTH VMs, run separately:
```sh
sudo apt update
sudo apt install chrony
chronyd -v
systemctl is-active chrony
systemctl is-active systemd-timesyncd
```
Review package prompts. Installing chrony may replace the earlier time service; record the change. Use one active clock synchronisation service. If another service still controls the clock or chrony is inactive, ask the teacher to resolve this baseline first.
Take the exercise backup AFTER installation. At the end restore this chrony configuration and keep chrony installed. Reinstating an earlier package/time service is a separate teacher-managed change.
On EACH VM, check the backup path is unused:
```sh
sudo test ! -e /etc/chrony.before-lab07 && echo AVAILABLE || echo CHOOSE-ANOTHER-NAME
```
Continue only after AVAILABLE. Otherwise choose a new path in every backup/restore command. Copy and compare:
```sh
sudo cp -a /etc/chrony /etc/chrony.before-lab07
sudo diff -r /etc/chrony /etc/chrony.before-lab07
```
No diff output normally means matching files. Stop if verification fails. Record paths separately for server and client. Back up any included file outside /etc/chrony before editing it too.
## 3. Enable the SERVER for the lab client
On the SERVER inspect the main file and its includes. Keep existing working upstream sources. Open:
```sh
sudo nano /etc/chrony/chrony.conf
```
Add this once, if it is not already present:
```text
allow 192.168.11.0/24
```
This permits lab-subnet NTP clients; it creates neither an upstream source nor a firewall rule. In nano, Ctrl+O then Enter saves; Ctrl+X exits. Record whether you added the line.


## 4. Verify the SERVER first
On the SERVER:
```sh
sudo chronyd -p -f /etc/chrony/chrony.conf
sudo systemctl restart chrony
systemctl is-active chrony
chronyc -n sources -v
chronyc tracking
sudo ss -lunp
```
-p prints/parses configuration and exits; it does not start another time daemon. Fix configuration errors BEFORE restart. Locate the UDP port 123 listener. A listener alone does not prove client synchronisation.
Wait a few minutes and repeat sources/tracking. In sources, ^ means server source and * means selected. -n shows numeric addresses. Confirm a usable upstream is selected and inspect tracking, normally including Leap status: Normal. If not, use TROUBLESHOOTING.md and retain the actual evidence. Do not manually change clocks or invent a selected source.
## Firewall, only if needed
On the SERVER inspect the existing policy:
```sh
sudo ufw status numbered
```
Keep the original firewall state and rules. If UFW is already active and lacks a suitable NTP rule, ask the teacher to check this narrow rule. Replace LAB_IF with the SERVER host-only interface:
```sh
sudo ufw allow in on LAB_IF from 192.168.11.20 to 192.168.11.10 port 123 proto udp comment 'SNA-Lab07'
```
Record whether it was NEW and its exact contents. Preserve any pre-existing equivalent rule. UFW inactive does not rule out other filtering; ping does not prove UDP 123 is open.
## 5. Configure the CLIENT source
Follow SOURCE-SETUP.md at the CLIENT console. Inspect main configuration, included .conf files and source directories before editing. Preserve non-source settings. Temporarily disable alternative sources and add one active lab source:
```text
server 192.168.11.10 iburst
```
iburst requests initial measurements; it does not prove success. Record every changed file/directive. Never replace the whole configuration with this one line.


## 6. Observe the CLIENT
After the recorded client edits, run on the CLIENT:
```sh
sudo chronyd -p -f /etc/chrony/chrony.conf
sudo systemctl restart chrony
chronyc -n sources -v
chronyc tracking
```
Fix parse errors before restart. Check that no unintended alternative source remains. An Internet source selected instead of .10 does not demonstrate use of the lab server. Use SOURCE-SETUP.md to find remaining sources.
Record measurements now and again after a few minutes in observations.csv. Include SERVER upstream measurements too. Keep incomplete/unsuccessful results rather than replacing them with examples.
## Read the evidence
^*: server source selected. On the client verify the address is 192.168.11.10, and interpret tracking alongside it.
^?: server source not currently selectable. Reasons can include insufficient samples, no valid replies or an unusable server. The symbol alone does not locate a cause.
Reach: octal history of valid replies, not a percentage, delay or accuracy score. 377 means valid replies for the last eight transmissions. A new working source can have a smaller value; 377 is not required to finish.
LastRx: time since the last good sample. Last sample: clock offset with units such as us or ms. Record actual values, not a copied target.
Tracking: inspect reference, stratum and leap status. Stratum is distance in the time hierarchy, not guaranteed accuracy. Timezone changes local display, not synchronisation quality.
## Explain in setup-record.md
1. Which evidence shows that the client used .10 rather than another source?
2. Why can chrony be active without a usable time source?
3. What does reach 377 establish, and what does it not establish?
4. Why must the lab server itself have a usable upstream?
5. What would you check if ping succeeds but .10 shows ^? on the client?
6. Why do matching clock displays or a correct timezone not prove this NTP path works?


## 7. Restore and verify
Follow RESTORE.md and your exact change record. Restore only changed files. Keep chrony installed; backups represent the post-install baseline.
On the CLIENT copy each changed file from its matching backup to its original path. If ONLY chrony.conf was changed and you used the example backup:
```sh
sudo cp -a /etc/chrony.before-lab07/chrony.conf /etc/chrony/chrony.conf
sudo cmp /etc/chrony.before-lab07/chrony.conf /etc/chrony/chrony.conf
```
This example is insufficient if included files were edited. Restore and compare ALL recorded changed files. Remove only a newly created lab-only file with the teacher; preserve pre-existing files and directories.
On the SERVER restore the file where you introduced allow, using its matching backup. Preserve an allow line that existed before class.
If a NEW SNA-Lab07 UFW rule was added, identify it in the CURRENT listing, then remove only that rule:
```sh
sudo ufw status numbered
sudo ufw delete NUMBER
sudo ufw status numbered
```
Replace NUMBER from the current listing. Preserve pre-existing equivalent rules and the original enabled/disabled state.
On BOTH VMs, parse before restart and verify:
```sh
sudo chronyd -p -f /etc/chrony/chrony.conf
sudo systemctl restart chrony
systemctl is-active chrony
chronyc -n sources -v
chronyc tracking
ip -br address
ip route
```
Confirm the original source arrangement is restored; pool addresses can change. Allow time for samples and record any lack of synchronisation. Keep lab .10/.20, original NAT routes, nginx and SSH usable. Save records and backups.
If upstream access is blocked, report the limitation. Fictional reading examples can support interpretation but must not be reported as your own successful configuration.
