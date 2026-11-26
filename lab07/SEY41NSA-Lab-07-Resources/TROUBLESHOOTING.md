# Chrony troubleshooting
Start from the relevant VM, actual source address and evidence. Do not manually move clocks or add a local reference mode to fabricate synchronisation.
- Daemon inactive: inspect systemctl status chrony --no-pager -l and sudo journalctl -u chrony -n 30 --no-pager. Run the configuration parser; fix only the reported/supported cause before restart. Check competing clock services with the teacher.
- Server has no selected upstream: inspect its retained sources, NAT route, upstream name resolution and network policy for the configured protocol. New samples take time. A working package download or ping is not proof that upstream NTP works. If campus policy blocks it, record the limitation and use a teacher-rehearsed upstream arrangement; keep the practical result incomplete.
- Client ^? or reach 0: inspect correct .10 source, server selection/tracking, client route/source, server allow and UDP123 listener. Check the actual scoped firewall policy. Wait for measurements, then recheck. Neither ping success nor a listed firewall rule proves NTP success.
- Selected source is not .10: inspect source/include/sourcedir chains using SOURCE-SETUP.md. Retain other settings; identify and temporarily disable alternative source definitions.
- Small reach value: this is recent reply history. A newly selected source need not have 377. Record repeated observations and units.
- Offset varies: virtual-machine scheduling, measurement and host clock integration may affect it. Record the result. Ask the teacher about the prepared VM image rather than changing host/hypervisor clock policies during the lab.
- Restoration differs: compare only files changed using recorded exact backups. Dynamic pool membership may differ even after correct configuration restoration.
