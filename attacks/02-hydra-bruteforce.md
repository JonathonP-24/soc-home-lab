Attack: SSH brute-force

ATT\&CK ID: T1110 (Brute Force)

Command used: hydra -l jonathon -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.30 -t 4

Log evidence: repeated failed SSH logins from Kali

Detection: Wazuh’s built-in rules (5551, 40111) — no custom rule needed, contrast with the nmap/UFW case

Troubleshooting note: SSH access briefly broke because enabling ufw earlier (for the nmap detection) blocked port 22 without an explicit allow rule — fixed with sudo ufw allow 22/tcp. firewall default-deny behavior.

Alert triggered: yes, level 10

Screenshot: Attatched

