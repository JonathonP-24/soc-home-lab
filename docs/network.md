# Network Plan

Lab network: 192.168.56.0/24 (VirtualBox host-only, no internet)

| Machine | Role | IP |
|---|---|---|
| Windows 11 host | Runs VirtualBox | 192.168.56.1 |
| Wazuh server | SIEM | 192.168.56.10 |
| Kali | Attacker | 192.168.56.20 |
| Ubuntu victim | Target | 192.168.56.30 |
| Windows victim | Target | 192.168.56.40 |