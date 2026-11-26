# Lab 7 working record

> **Setup notes and deviations**
> - Labs 5–6 were not completed. The network baseline needed for Lab 7 was prepared separately and is recorded below.
> - Both VMs run **Ubuntu 26.04.1 LTS** instead of the specified Ubuntu Server 24.04 LTS.
> - Items marked *(reported)* were stated by the student but are not shown in a screenshot.
> - All times are UTC on 2026-11-26.

**Server hostname/account and lab/NAT interfaces:**

- Hostname `sna-srv01`, account `davin`; Ubuntu 26.04.1 LTS, kernel 7.0.0-30-generic, arm64, VMware; machine ID `8d457c299ef74a65b751f7e9724cceb5`.

| Interface | Role | MAC | IPv4 at baseline | IPv4 after static setup | Link-local |
|---|---|---|---|---|---|
| `enp2s0` | NAT (default route) | 00:0c:29:d4:1f:63 | 192.168.202.131/24 (DHCP) | 192.168.202.129/24 (DHCP) | fe80::20c:29ff:fed4:1f63/64 |
| `enp26s0` | Lab (host-only) | 00:0c:29:d4:1f:6d | 192.168.11.129/24 (DHCP) | **192.168.11.10/24 (static)** | fe80::20c:29ff:fed4:1f6d/64 |

**Client hostname/account and lab/NAT interfaces:**

- Hostname `sna-cli01`, account `davin`; Ubuntu 26.04.1 LTS, kernel 7.0.0-30-generic, arm64, VMware; machine ID `ee75e09dcea5437cbe36b6c2f571b7ff`.
- Hostname, `/etc/hosts` and machine ID were corrected after cloning (`/etc/hosts` change *(reported)*). The different hostnames and machine IDs are shown in vm-identity.png.

| Interface | Role | MAC | IPv4 at baseline | IPv4 after static setup | Link-local |
|---|---|---|---|---|---|
| `enp2s0` | NAT (default route) | 00:0c:29:96:68:4f | 192.168.202.132/24 (DHCP) | 192.168.202.132/24 (DHCP) | fe80::20c:29ff:fe96:684f/64 |
| `enp26s0` | Lab (host-only) | 00:0c:29:96:68:59 | 192.168.11.128/24 (DHCP) | **192.168.11.20/24 (static)** | fe80::20c:29ff:fe96:6859/64 |

![VM identity: hostnamectl and ip -br addr on client (left) and server (right)](vm-identity.png)
![Network baseline: hostname, ip -br link/address, ip route on client (left) and server (right)](network-baseline.png)

**Original lab addresses and NAT routes:**

- **Host-only network:** VMware custom network `vmnet2`, subnet 192.168.11.0/24, NAT disabled, DHCP enabled. Network Adapter 2 of each VM is connected to `vmnet2`. DHCP range 192.168.11.128–192.168.11.254 and reserved address 192.168.11.1 *(reported)*. The static addresses .10 and .20 are outside the DHCP pool.
- **NAT default route (both VMs):** `default via 192.168.202.2 dev enp2s0 proto dhcp`. Server: `src 192.168.202.129 metric 100` (src .131 at baseline). Client: `src 192.168.202.132 metric 1024`.
- **Lab route (both VMs):** `192.168.11.0/24 dev enp26s0 proto kernel scope link`. No default route on `enp26s0`.
- **Netplan at baseline:** both VMs defined only `enp2s0`, matched on an outdated MAC `00:0c:29:a2:4c:2d` that matched neither VM. `enp26s0` was not defined. The MAC was corrected and static IPs were configured on `enp26s0` (corrected Netplan content not shown).
- **Network configuration source:** at baseline both interfaces used systemd-networkd with Dracut fallback *(reported)*. After the fix, `networkctl status enp2s0` showed the server using Netplan (`/run/systemd/network/10-netplan-enp2s0.network`), while the client still used the Dracut fallback (`/run/systemd/network/zzzz-dracut-default.network`).
- **Connectivity:**
  - On the client, `ip route get 192.168.11.10` → `dev enp26s0 src 192.168.11.20`.
  - `ping -c 4 192.168.11.10` → 4/4 received, 0% loss, rtt 0.680/1.118/2.230/0.645 ms.
  - `curl -I https://ports.ubuntu.com` → `HTTP/1.1 200 OK` on both VMs.

![Host-only network: VMware vmnet2, 192.168.11.0/24, NAT disabled](host-only-network.png)
![Server Network Adapter 2 connected to vmnet2](server-network-adapter.png)
![Client Network Adapter 2 connected to vmnet2](client-network-adapter.png)
![Netplan baseline: sudo netplan get on client (left) and server (right)](netplan-baseline.png)
![Server static IP: ip -br address and ip route on sna-srv01](server-static-ip.png)
![Client static IP: ip -br address and ip route on sna-cli01](client-static-ip.png)
![Server inspection: networkctl, ip -br address, ip route, timedatectl on sna-srv01](server-inspection.png)
![Client inspection: networkctl, ip -br address, ip route, timedatectl on sna-cli01](client-inspection.png)
![Client ping: ip route get and ping to 192.168.11.10](client-ping.png)

**Time services/timezones before installation:**

- **Timezone (both VMs):** `timedatectl` showed time zone Etc/UTC (UTC, +0000), System clock synchronized: yes, NTP service: active, RTC in local TZ: no.
- **Chrony:** version 4.8 was already installed and `active` on both VMs.
- **systemd-timesyncd:** `systemctl status` reported "Unit systemd-timesyncd.service could not be found", and `is-active` reported `inactive`.
- **Chrony log (both VMs):** some sources went offline and chrony logged "Can't synchronise: no selectable s…ces" (line truncated in the screenshot). All five sources then came back online, and 185.125.190.123 was selected.
- **Recurring warning:** "System clock interference detected (another NTP client?)".
  - Server: 15:11:30, 15:15:29 and 15:29:29.
  - Client: 15:27:00, 15:34:24 and 15:36:41.
  - **The cause is unconfirmed.**
- **Other services (both VMs):** `ssh.socket` `active`; `ssh` service `inactive`; `nginx` `inactive`.

![Server services: chrony status/log, ssh/nginx state, curl on sna-srv01](server-services.png)
![Client services: chrony status/log, ssh/nginx state, curl on sna-cli01](client-services.png)
![Chrony version: chronyd -v and chrony/timesyncd state on client (left) and server (right)](chrony-version.png)
![Service check: ssh.socket/ssh/nginx state and chrony interference log on client (left) and server (right)](chrony-service-check.png)

**Package/service changes during installation:**

- Chrony 4.8 was already installed and active on both VMs before the installation step. No package change is shown.
- The APT installation output was not provided.

**Post-install baseline service/source arrangement:**

| Item | Server `sna-srv01` | Client `sna-cli01` |
|---|---|---|
| `chrony.conf` source directives | `:9 sourcedir /run/chrony-dhcp`, `:12 sourcedir /etc/chrony/sources.d`, `:48 confdir /etc/chrony/conf.d` | Same three lines |
| Pools in `sources.d/ubuntu-ntp-pools.sources` | `:7 pool 4.ntp.ubuntu.com …`, `:11 pool ntp-bootstrap.ubuntu.com …` (earlier lines cut off in screenshot) | Five pools (below) |
| Source directories | Not shown | `/run/chrony-dhcp` does not exist; `/etc/chrony/sources.d` holds `README` and `ubuntu-ntp-pools.sources` |
| Selected source (`chronyc -n sources -v`) | `^*` 185.125.190.123, stratum 2, reach 377 | `^*` 185.125.190.123, stratum 2, poll 10, reach 377, LastRx 481, `+1957us[+3204us] +/- 104ms` |
| Other sources | `^+` 185.125.190.122, 91.189.91.112, 91.189.91.113; `^-` 91.189.91.111 (all stratum 2, reach 377) | Same as server |

The client's five original pools are:
- `:4 pool 1.ntp.ubuntu.com iburst maxsources 1 nts prefer`
- `:5 pool 2.ntp.ubuntu.com iburst maxsources 1 nts prefer`
- `:6 pool 3.ntp.ubuntu.com iburst maxsources 1 nts prefer`
- `:7 pool 4.ntp.ubuntu.com iburst maxsources 1 nts prefer`
- `:11 pool ntp-bootstrap.ubuntu.com iburst maxsources 1 nts certset 1`

On the client, the original `chronyd -p` parse listed both `sourcedir` lines and no other source directive.

![Server chrony baseline: source directives and chronyc -n sources -v on sna-srv01](server-chrony-baseline.png)
![Client chrony baseline 1: original chronyd -p parse and source directories on sna-cli01](client-chrony-baseline-1.png)
![Client chrony baseline 2: source directories and source/include directives on sna-cli01](client-chrony-baseline-2.png)
![Client chrony baseline 3: original chronyc -n sources -v on sna-cli01](client-chrony-baseline-3.png)

**Server backup path and copy comparison:**

- Chrony backup: `/etc/chrony.before-lab07`. The path was checked as unused (`AVAILABLE`), then copied with `sudo cp -a /etc/chrony /etc/chrony.before-lab07`. `sudo diff -r` gave no output, so the copy matches.
- Netplan backup: `/etc/netplan.before-lab07`; comparison gave no diff output *(reported)*.

**Client backup path and copy comparison:**

- Chrony backup: `/etc/chrony.before-lab07`. The path was checked as unused (`AVAILABLE`), then copied with `sudo cp -a /etc/chrony /etc/chrony.before-lab07`. `sudo diff -r` gave no output, so the copy matches.
- Netplan backup: `/etc/netplan.before-lab07`; comparison gave no diff output *(reported)*.

![Chrony backup: name check, cp -a and diff -r on client (left) and server (right)](chrony-backup.png)

## Change inventory (repeat for each file)

### Change 1: Server

**VM and original file path:** Server `sna-srv01`, `/etc/chrony/chrony.conf`

**Active include/source role:** Main chrony configuration. It contains both `sourcedir` lines and `confdir /etc/chrony/conf.d`. No source lines were changed, so the existing upstream sources were preserved.

**Exact matching backup path:** `/etc/chrony.before-lab07/chrony.conf`

**Edit made and reason:** Added `allow 192.168.11.0/24` and a blank line after `driftfile /var/lib/chrony/chrony.drift` (`diff -u` hunk `@@ -19,6 +19,8 @@`). The reason was to let clients on the lab subnet use the server for NTP.

**Restore command/path and comparison result:**
- Restored with `sudo cp -a /etc/chrony.before-lab07/chrony.conf /etc/chrony/chrony.conf`.
- `sudo cmp /etc/chrony.before-lab07/chrony.conf /etc/chrony/chrony.conf && echo RESTORE-PASSED` printed `RESTORE-PASSED`.

![Server chrony config: diff -u backup vs current chrony.conf on sna-srv01](server-chrony-config.png)
![Server restore: cp -a, cmp and parse/restart on sna-srv01](server-restore.png)

### Change 2: Client

**VM and original file path:** Client `sna-cli01`, `/etc/chrony/chrony.conf`

**Active include/source role:** Main chrony configuration. It originally contained `sourcedir /run/chrony-dhcp` and `sourcedir /etc/chrony/sources.d`. `confdir /etc/chrony/conf.d` was not changed. No other file was edited.

**Exact matching backup path:** `/etc/chrony.before-lab07/chrony.conf`

**Edit made and reason:**
- Commented out both `sourcedir` lines.
- Added `server 192.168.11.10 iburst` and a blank line.
- A trailing blank line was also added after `confdir` (`diff -u` hunks `@@ -6,10 +6,12 @@` and `@@ -46,3 +48,4 @@`).
- The reason was to make the lab server the client's only time source.

**Restore command/path and comparison result:**
- Restored with `sudo cp -a /etc/chrony.before-lab07/chrony.conf /etc/chrony/chrony.conf`.
- `sudo cmp … && echo RESTORE-PASSED` printed `RESTORE-PASSED`.

![Client chrony config: diff -u backup vs current chrony.conf on sna-cli01](client-chrony-config.png)
![Client restore: cp -a, cmp and parse/restart on sna-cli01](client-restore.png)

**Server allow line: pre-existing or newly added:** Newly added. `allow 192.168.11.0/24` is not in the backup (shown by `diff -u`).

**Firewall original state and exact NEW rule, if any:**
- Server UFW was `inactive` (`sudo ufw status numbered`), and no firewall rule was added.
- `sudo ss -lunp` showed chronyd listening on UDP `0.0.0.0:123` (pid 3531).

![Server NTP/firewall: ss -lunp :123 and ufw status on sna-srv01](server-ntp-firewall.png)

**Client source/include directives temporarily changed:** In `/etc/chrony/chrony.conf`, `sourcedir /run/chrony-dhcp` and `sourcedir /etc/chrony/sources.d` were commented out, and `server 192.168.11.10 iburst` was added.

**Parse result before each restart:**

| Stage | VM | Result |
|---|---|---|
| After allow edit | Server | Parse output before the restart was **not captured**. After the restart, chrony was `active` and synchronised (see observations.csv). |
| After source edit | Client | `sudo chronyd -p -f /etc/chrony/chrony.conf` passed with no error; the only source directive was `server 192.168.11.10 iburst`. The restart was run with `&&` and printed no error. |
| After restoration | Both | `chronyd -p … && systemctl restart chrony` passed with no error. Output showed both original `sourcedir` lines and no `server` or `allow` line. |

![Client chrony parse: chronyd -p output on sna-cli01](client-chrony-parse.png)

**Synchronisation results during the exercise** (full values in observations.csv):

| Time | VM | Selected source (`^*`) | Reach | Tracking reference / stratum / leap |
|---|---|---|---|---|
| Not captured | Server | 91.189.91.112 | 17 | 185.125.190.123 / 3 / Normal (**mismatch**) |
| 16:04:55 | Server | 185.125.190.123 | 177 | 185.125.190.123 / 3 / Normal (match) |
| 16:25:42 | Client | 192.168.11.10 | 17 | 192.168.11.10 / 4 / Normal (match) |
| 16:28:27 | Client | 192.168.11.10 | 77 | 192.168.11.10 / 4 / Normal (match) |

The first server reading showed a mismatch, and the second confirmed that the selected source and the tracking reference matched. In both client readings, 192.168.11.10 was the only source listed, with no alternative sources.

![Server chrony verification 1: chrony state and chronyc -n sources -v on sna-srv01](server-chrony-verification-1.png)
![Server chrony verification 2: chronyc sources and tracking on sna-srv01](server-chrony-verification-2.png)
![Server upstream later 1: date -u and chronyc -n sources -v on sna-srv01](server-upstream-later-1.png)
![Server upstream later 2: chronyc sources and tracking on sna-srv01](server-upstream-later-2.png)
![Client sync initial 1: parse, restart and date -u on sna-cli01](client-sync-initial-1.png)
![Client sync initial 2: date -u and chronyc -n sources -v on sna-cli01](client-sync-initial-2.png)
![Client sync initial 3: chronyc tracking on sna-cli01](client-sync-initial-3.png)
![Client sync later 1: date -u and chronyc -n sources -v on sna-cli01](client-sync-later-1.png)
![Client sync later 2: chronyc sources and tracking on sna-cli01](client-sync-later-2.png)

## Interpretation

1. **Evidence that the client selected .10 rather than an alternative:**
   - In both client readings (16:25:42 and 16:28:27), `chronyc -n sources` showed `^* 192.168.11.10` as the only source.
   - `chronyc tracking` showed Reference ID `C0A80B0A (192.168.11.10)`.
   - The client was at stratum 4, one step further from the reference than the server (stratum 3).
   - The parsed configuration contained only `server 192.168.11.10 iburst`.

2. **Why active alone is insufficient:**
   - `active` only means that chronyd is running. It does not show that a usable source is selected.
   - For example, the server logged "Can't synchronise: no selectable s…ces" while chrony was running.
   - Synchronisation needs a `^*` source in `sources` and a matching reference with Leap status Normal in `tracking`.

3. **Meaning and limit of reach 377:**
   - Reach 377 (octal) means that the last eight polls all received valid replies. It shows recent reachability only.
   - It does not show accuracy, offset, delay or selection. For example, 91.189.91.111 had reach 377 at the client baseline but was marked `^-` (not combined).
   - Reach 377 is also not required: the client selected .10 at reach 17.

4. **Why server upstream matters:**
   - The server was the client's only source, so the client can be no more accurate than the server.
   - The server received time from stratum-2 pool servers through NAT, which made it stratum 3.
   - Without a usable upstream, the server would be unsynchronised. The client would then reject .10 as unusable or follow an unreferenced clock.

5. **Next checks if ping works but client .10 is ^?:** Ping only tests ICMP, not NTP on UDP 123. I would check:
   - on the server: `chronyc tracking` (synchronised, Leap status Normal), the `allow 192.168.11.0/24` line and a restart after editing, a UDP 123 listener (`ss -lunp`), and the firewall;
   - on the client: the configuration parse, enough time for samples, and the reach value and `chronyc ntpdata 192.168.11.10`;
   - on both VMs: whether UDP 123 packets pass on `enp26s0` (`tcpdump -ni enp26s0 udp port 123`).

6. **Why clock display/timezone is insufficient:**
   - The timezone (Etc/UTC on both VMs) only changes how the time is displayed.
   - Two clocks can match because of another time source, such as internet pools or the hypervisor. The "System clock interference detected" warnings suggest that something else may adjust the clock.
   - Only `chronyc sources` and `chronyc tracking` show that the client used .10 over UDP 123.

## Final verification

**Each changed file restored/compared:**
- `/etc/chrony/chrony.conf` was restored on both VMs from `/etc/chrony.before-lab07/chrony.conf`, and `cmp` printed `RESTORE-PASSED` on both.
- The Netplan MAC fix and static IPs were network baseline fixes, not Lab 7 time changes, so they were kept.

**Original source arrangement restored on each VM:**
- Yes. On both VMs, the parse shows both original `sourcedir` lines with no `server 192.168.11.10` and no `allow` line.
- Both VMs again list the five pool servers 185.125.190.122/123 and 91.189.91.111/112/113.

**New-only firewall rule removed and original state retained:**
- No rule was added.
- Server UFW was `inactive` when first checked and was still `inactive` at the end.

![Server final firewall: ufw status numbered on sna-srv01](server-final-firewall.png)

**Final source/tracking observations and limitations:**

| Time | VM | Selected source (`^*`) | Reach | Tracking reference / stratum / leap |
|---|---|---|---|---|
| 16:38:51 | Client | 185.125.190.123 | 177 | 185.125.190.123 / 3 / Normal |
| 16:40:38 | Server | 91.189.91.112 | 77 | 91.189.91.112 / 3 / Normal |
| 16:46:32 | Client | 185.125.190.123 | 377 | 185.125.190.123 / 3 / Normal |
| 16:47:02 | Server | 185.125.190.122 | 377 | 185.125.190.122 / 3 / Normal |

- All final readings show the selected source matching the tracking reference. The server's selection moved between pool servers, which is normal for a pool.
- **Limitations:**
  - The cause of the clock interference is unconfirmed.
  - The first server reading showed a mismatch.
  - The server's parse output before restart was not captured.

![Client final 1: date -u, chrony state and chronyc -n sources -v on sna-cli01](client-final-1.png)
![Client final 2: chronyc sources and tracking on sna-cli01](client-final-2.png)
![Server final 1: date -u, chrony state and chronyc -n sources -v on sna-srv01](server-final-1.png)
![Server final 2: chronyc sources and tracking on sna-srv01](server-final-2.png)
![Client restored later: date -u, chronyc sources and tracking on sna-cli01](client-restored-later.png)
![Server restored later: date -u, chronyc sources and tracking on sna-srv01](server-restored-later.png)

**Lab addresses, NAT routes, nginx and SSH retained:**
- **Lab addresses:** kept (server `enp26s0` 192.168.11.10/24, client `enp26s0` 192.168.11.20/24).
- **NAT routes:** kept (`default via 192.168.202.2 dev enp2s0`; server src .129 metric 100, client src .132 metric 1024).
- **Ping after restoration:** `ping -c 4 192.168.11.10` from the client → 4/4 received, 0% loss, rtt 0.519/0.821/1.410/0.359 ms.
- **SSH and nginx:** unchanged from the start. On both VMs `ssh.socket` was `active`, while the `ssh` service and `nginx` were `inactive`. **SSH connectivity was not tested, and nginx was inactive before and after the lab.**

![Client final 3: ip -br address, ip route, ssh/nginx state on sna-cli01](client-final-3.png)
![Server final 3: ip -br address, ip route, ssh/nginx state on sna-srv01](server-final-3.png)
![Client final ping: ping -c 4 192.168.11.10 after restoration](client-final-ping.png)

**Unperformed/blocked steps:**
- Labs 5–6 were not completed. The network baseline was prepared separately for this lab.
- Ubuntu 26.04.1 LTS is used instead of the specified 24.04 LTS.
- The APT installation output was not provided, because Chrony 4.8 was already installed.
- The server's parse output before restart (after the allow edit) was not captured.
- The client's UFW state and a ping from the server to the client were not captured.
- SSH connectivity was not tested, and nginx is inactive.
- The cause of the clock interference warnings is unconfirmed.
