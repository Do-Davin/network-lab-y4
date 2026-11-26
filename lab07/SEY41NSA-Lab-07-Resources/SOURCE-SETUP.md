# Client source setup
Work on the CLIENT .20 console after the server has a usable upstream and the post-install client backup is verified. Do not change the server's upstream sources. Record every changed file in setup-record.md.

## 1. Inspect before editing
```sh
sudo cat /etc/chrony/chrony.conf
sudo chronyd -p -f /etc/chrony/chrony.conf
sudo grep -R -n -E '^[[:space:]]*(server|pool|peer|include|confdir|sourcedir)[[:space:]]' /etc/chrony
chronyc -n sources -v
```
The grep lists candidate lines and filenames; it does not prove every file is active. The parser expands include/confdir configuration, but sourcedir contents need separate inspection. Follow the directives actually read from chrony.conf. Comments begin with #. Keep the paths in your record.

## 2. Follow the active sources
- server: one source address/name. pool: sources discovered from a pool name. peer: a peer association. Temporarily comment active alternative lines by adding # at the start; preserve the original text.
- include: reads matching files. confdir: reads .conf files in the stated directory. Open active included files and comment only their server/pool/peer lines. Preserve other settings; do not comment a whole mixed-purpose include/confdir just to remove a source.
- sourcedir: reads .sources files containing only source definitions, often including dynamically supplied DHCP sources. Inspect the actual listed directories/files. Temporarily comment the active sourcedir directive(s) for this exercise, recording their exact original lines. Do not edit a generated /run file that can be recreated later.
- If a mixed include/confdir leads to another include/sourcedir, follow it too. If any active file lies outside the copied /etc/chrony tree, back it up separately to an unused path before editing, recording source/destination. Get help if you cannot establish the active chain.

On a typical Ubuntu image the main file, conf.d, sources.d and /run/chrony-dhcp may be involved, but do not assume they exist or are all enabled. Use your observed directives.

## 3. Edit the identified files
Open main configuration:
```sh
sudo nano /etc/chrony/chrony.conf
```
Open any active included .conf file separately using its observed path. Add exactly one active line in chrony.conf:
```text
server 192.168.11.10 iburst
```
Keep drift/log settings, other non-source options and existing permissions. Do not replace the file with a one-line sample. Avoid creating a new file; editing recorded originals makes restoration simpler. Nano: Ctrl+O, Enter, Ctrl+X.

## 4. Parse, restart, inspect
```sh
sudo chronyd -p -f /etc/chrony/chrony.conf
sudo systemctl restart chrony
chronyc -n sources -v
chronyc tracking
```
If parsing reports an error, fix it BEFORE restart. After restart the source list should contain only the intended lab source for this exercise; check again after measurements arrive. A selected Internet source is not evidence of .10 synchronisation. If alternative entries remain, inspect active source definitions/includes again instead of accepting a convenient success. Record any blocker honestly.
Use RESTORE.md after observations. Backups are the post-install originals; retain them.
