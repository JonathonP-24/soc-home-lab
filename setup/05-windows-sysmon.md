\# Windows Victim — Sysmon Setup



\## What was installed

\- Sysmon64, with the SwiftOnSecurity community config (sysmonconfig-export.xml)

\- Config path: C:\\Users\\jonathon\\Downloads\\Sysmon

\- Installed as a service: Sysmon64 (confirmed via Get-Service)



\## Wazuh integration

\- Added a <localfile> block to C:\\Program Files (x86)\\ossec-agent\\ossec.conf:

&#x20; <localfile>

&#x20;   <location>Microsoft-Windows-Sysmon/Operational</location>

&#x20;   <log\_format>eventchannel</log\_format>

&#x20; </localfile>

\- Confirmed in ossec.log: "Analyzing event log: 'Microsoft-Windows-Sysmon/Operational'"

\- Restarted WazuhSvc to apply



\## Verification

\- Confirmed Sysmon logging locally via Event Viewer (Applications and Services Logs >

&#x20; Microsoft > Windows > Sysmon > Operational) — process creation and network connect

&#x20; events both present

\- Confirmed events reaching Wazuh server: search `agent.name:"windows-victim" AND

&#x20; rule.groups:"sysmon"` returned results

\- First real alert: "Executable file dropped in folder commonly used by malware" —

&#x20; triaged and confirmed as a false positive (Sysmon64.exe itself sitting in the

&#x20; Downloads folder from setup, not malicious activity)



\## Finding: detection gap on network-connect events

\- Sysmon Event ID 3 (Network Connect) logged locally when testing with

&#x20; Test-NetConnection to the Wazuh server (192.168.56.10:443)

\- This event did NOT appear in Wazuh's dashboard search, despite the agent

&#x20; actively reading the Sysmon channel and other Sysmon event types (process

&#x20; creation, file creation) matching rules successfully

\- Root cause: Wazuh only indexes events that match a rule by default (same

&#x20; behavior observed earlier with UFW firewall logs); no default rule flags

&#x20; this specific Sysmon network-connect pattern, so it's collected but not

&#x20; alerted on or searchable

\- Opportunity: write a custom rule for Sysmon Event ID 3 to catch

&#x20; network-connect activity, similar to the UFW port-scan rule already built

