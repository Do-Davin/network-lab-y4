# Restore the exercise changes
Use the exact paths recorded in setup-record.md. Keep chrony installed. The exercise restores its post-install configuration; it does not automatically undo package replacement of a previous time service.
1. CLIENT: restore EACH edited original from its corresponding backup, including any included .conf file or separately backed-up external file. Example when the default backup was used:
```sh
sudo cp -a /etc/chrony.before-lab07/chrony.conf /etc/chrony/chrony.conf
sudo cmp /etc/chrony.before-lab07/chrony.conf /etc/chrony/chrony.conf
```
For another edited file, use its exact original/backup paths; do not repeat the main-file command as if it restores everything. cmp normally prints nothing for identical files. Stop on copy/comparison errors. Remove only a recorded new lab-only file with the teacher, if one was created; do not remove a whole directory.
2. SERVER: restore the original file from which you added the lab allow line. If the line was already there, do not remove it. Restore any other recorded exercise-only edit from its exact backup.
3. If you added a NEW firewall rule, use the current sudo ufw status numbered listing. Identify its interface, source .20, destination .10, UDP123 and SNA-Lab07 comment. Delete only that current rule number. Preserve pre-existing rules and the original UFW state.
4. BOTH: parse configuration with sudo chronyd -p -f /etc/chrony/chrony.conf, correct errors, restart chrony and inspect sources/tracking. Check the original source arrangement is back; a DNS pool may produce different numeric servers. Record two-stage measurements and any remaining failure rather than claiming success from active alone.
5. Confirm original NAT defaults, lab /24 addresses, nginx and SSH remain available. Keep your backups/change records. If restoring fails, retain console access and request help with the exact affected file; do not reinstall packages or erase networking as a shortcut.
