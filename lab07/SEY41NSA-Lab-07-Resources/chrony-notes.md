# Chrony terms and reading practice
NTP: Network Time Protocol. UTC: Coordinated Universal Time. Timezone: local display convention.
chronyd: daemon controlling/synchronising the clock. chronyc: monitoring/control client.
sources ^: server source. *: selected. +: acceptable source combined with selected source. ?: not selectable, with multiple possible reasons. Read the address and tracking together.
Reach: octal register covering the last eight transmissions; 377 means eight valid replies, not 377 percent or an accuracy certificate. LastRx is time since the last good sample. Poll is a base-2 interval exponent; e.g. 6 means 64 seconds. Offsets have explicit ns/us/ms/s units.
Stratum describes the time hierarchy. A client of a synchronised stratum-3 server normally reports a higher level, but stratum is not a measured accuracy guarantee. Normal leap status supports current synchronisation state; it does not independently certify upstream correctness.

Fictional reading examples, NOT outputs to copy as live results:
- A: service active; source ^? 192.0.2.10, reach 0; tracking Leap status Not synchronised.
- B: service active; source ^* 192.0.2.10, reach 17; tracking Leap status Normal.
- C: service active; source ^* 192.0.2.30 although the assigned source was .10; tracking Normal.
For each, explain what is supported, what is still uncertain and your next check. Documentation addresses deliberately differ from the live lab addresses.

Official references:
https://chrony-project.org/doc/4.5/chronyc.html
https://chrony-project.org/doc/4.5/chrony.conf.html
https://chrony-project.org/doc/4.5/chronyd.html
https://ubuntu.com/server/docs/about-time-synchronisation/
