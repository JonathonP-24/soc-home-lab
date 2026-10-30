\# Atomic Red Team — T1082 System Information Discovery



\## Attack

Ran a single, isolated Atomic Red Team test targeting one specific sub-technique,

rather than the full T1082 test suite, to get an unambiguous, verifiable result.



\## ATT\&CK ID

T1082 — System Information Discovery (Discovery tactic)

Specific test: T1082-1 (runs the native `systeminfo` command)



\## Command used

On the Windows victim (windows-victim, 192.168.56.40), in an Administrator

PowerShell session with Invoke-AtomicRedTeam imported:



&#x20;   Invoke-AtomicTest T1082 -TestNumbers 1



This runs Windows' built-in `systeminfo` command, which an attacker would use

immediately after gaining access to a machine to profile the OS, patch level,

hardware, and network configuration before deciding next steps.



\## Local test output (confirms the test itself ran)

\- Host Name: WINDOWS-VICTIM

\- OS: Microsoft Windows 11 Enterprise Evaluation, Build 26300

\- Boot Time, hotfixes (6 installed), network adapters and IPs, and

&#x20; Virtualization-based security status all printed correctly

\- Exit code: 0 (success)



\## Log evidence (Wazuh)

Search used:

&#x20;   agent.name:"windows-victim" AND data.win.eventdata.commandLine:\*systeminfo\*



Confirmed:

\- data.win.eventdata.commandLine: systeminfo

\- data.win.eventdata.image: C:\\\\Windows\\\\System32\\\\systeminfo.exe

\- data.win.eventdata.parentImage: C:\\\\Windows\\\\System32\\\\cmd.exe

\- Timestamp matched the exact time the test was run, confirming this Sysmon

&#x20; event is the actual test activity, not coincidental background noise



\## Why this test (and not the full suite)

An earlier run of `Invoke-AtomicTest T1082` with no test number executed every

registered sub-test under T1082, including broader enumeration tooling

(Seatbelt-based checks covering accounts, credentials, scheduled tasks). That

produced a large, mixed batch of Sysmon/Wazuh alerts, including one tagged

T1087 (Account Discovery) rather than T1082 directly. Re-running a single,

numbered sub-test isolates one specific command to one specific log entry,

removing any ambiguity about what triggered what — a basic but important

verification step before trusting a detection pipeline.



\## Troubleshooting notes

\- Had to fully disable Windows Defender (registry policy DisableAntiSpyware,

&#x20; plus manually turning off Smart App Control) before Atomic Red Team's

&#x20; installer and test scripts would run — Defender's AMSI layer flagged them

&#x20; as malicious, which is expected/correct behavior for legitimate AV against

&#x20; offensive security tooling, not a bug.

\- Confirmed Sysmon only logs FileCreate events for executables/scripts, not

&#x20; every file write (e.g. plain .txt files aren't logged) — intentional noise

&#x20; reduction in the SwiftOnSecurity config, not a detection gap.



\## Alert triggered

Yes — Sysmon process-creation event matched by Wazuh, tied to the exact

systeminfo.exe execution.



\## Screenshots

See /screenshots/atomic-t1082-1-local-output.png and

/screenshots/atomic-t1082-1-wazuh-alert.png

