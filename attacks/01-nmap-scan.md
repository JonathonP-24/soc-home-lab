\# Attack 1 — Nmap Port Scan



\## Attack

Ran an Nmap TCP scan from the Kali attacker against the Ubuntu victim, to

simulate the reconnaissance phase of an attack — scanning a target for open

ports and running services before deciding how to proceed.



\## ATT\&CK ID

T1046 — Network Service Scanning (Discovery tactic)



\## Commands used

From Kali (192.168.56.20), against the Ubuntu victim (192.168.56.30):



&#x20;   nmap -sV -O 192.168.56.30

&#x20;   nmap -p 1-1000 192.168.56.30



The first scan (service/OS detection) produced no visible detection by

default. The second, wider scan (1000 ports) was used to generate enough

blocked-connection volume to properly test a detection rule.



\## Initial finding: no detection by default

A plain Nmap scan produced nothing in Wazuh — Wazuh's default rules watch

log files and system state, not raw network traffic, and a fresh Ubuntu

install has no firewall logging enabled, so a port scan leaves no trace

in the OS logs unless something is explicitly configured to record it.



\## Fix: enable and log with ufw

On the Ubuntu victim:



&#x20;   sudo ufw logging on

&#x20;   sudo ufw enable



This made every blocked connection attempt get written to /var/log/ufw.log,

giving Wazuh something to actually read. (Note: enabling ufw without first

allowing SSH also temporarily locked out SSH access to this machine — see

the Hydra writeup for how that was fixed.)



\## Wazuh configuration

Confirmed /var/log/ufw.log was being monitored by the agent

(wazuh-logcollector: "Analyzing file: '/var/log/ufw.log'"), but raw log

lines still didn't appear as alerts — Wazuh only indexes events that match

a rule, and no default rule recognized a UFW block line as alert-worthy.



\## Custom detection rule written

Added to /var/ossec/etc/rules/local\_rules.xml on the Wazuh server:



&#x20;   <rule id="100100" level="5">

&#x20;       <location>/var/log/ufw.log</location>

&#x20;       <match>UFW BLOCK</match>

&#x20;       <description>UFW firewall event detected</description>

&#x20;       <group>firewall,</group>

&#x20;   </rule>



&#x20;   <rule id="100101" level="10" frequency="8" timeframe="20">

&#x20;       <if\_matched\_sid>100100</if\_matched\_sid>

&#x20;       <description>Possible port scan detected: multiple UFW blocks in a short window</description>

&#x20;       <mitre>

&#x20;         <id>T1046</id>

&#x20;       </mitre>

&#x20;       <group>recon,attack,</group>

&#x20;   </rule>



Rule 100100 flags any individual UFW block as a low-severity event. Rule

100101 escalates to a real alert (level 10) when 8+ blocks occur within 20

seconds — the actual signature of a port scan, as opposed to one normal

blocked connection.



\## Log evidence

Raw /var/log/ufw.log entries showed repeated SYN packets from the Kali IP

(192.168.56.20) hitting dozens of different destination ports on the victim

(192.168.56.30) within the same second — e.g. ports 111, 49167, 587, 995,

53, 143, 993, 135, 1723, 23, 110, 199 all probed within a \~1 second window.



\## Alert triggered

Yes. After the fix: "Possible port scan detected: multiple UFW blocks in a

short window" — level 10, tagged MITRE T1046.



\## Troubleshooting notes

\- Initial decoder-based approach (attempting to extract srcip/dstip from

&#x20; the UFW log line with a custom decoder) repeatedly failed due to XML

&#x20; syntax errors and decoder-parent chaining issues. Simplified to a

&#x20; frequency-based rule with no IP extraction, since the lab only has one

&#x20; attacker machine — same detection outcome with far less complexity.

\- This is a good example of knowing when to simplify a detection rather

&#x20; than over-engineer it.



\## Screenshots

/screenshots/nmap _custom_detection_rule 1001001 alert.png

