# Wazuh SOC Lab

Welcome to my Wazuh SOC Lab repository.

This project documents my journey of deploying, configuring, and using different security tools to build a functional Security Operations Center (SOC) monitoring and detection environment — from log collection through detection engineering to automated response.

---

## 🎯 Skills Demonstrated

SIEM/XDR deployment and administration (Wazuh) · endpoint monitoring (Sysmon, Windows Event Logs) · File Integrity Monitoring · log ingestion and correlation · custom decoder and rule writing · attack simulation (recon + brute force) · alert triage and incident investigation · automated active response · threat intelligence enrichment (VirusTotal) · network IDS (Suricata) · firewall log integration (pfSense).

**Tools:** Wazuh (SIEM/XDR), Suricata (IDS/IPS), pfSense (Firewall), Sysmon, VirusTotal API, Hydra, Nmap, Wireshark — on VMware, with a Windows 11 endpoint and a Kali Linux attacker.

---

## 📊 Project Status

| Phase | Component | Status |
|-------|-----------|--------|
| 1 | Wazuh SIEM (Manager, Indexer, Dashboard) | ✅ Complete |
| 2 | Windows agent + Sysmon ingestion | ✅ Complete |
| 3 | File Integrity Monitoring (FIM) | ✅ Complete |
| 4 | Attack simulation (recon + SSH brute force) | ✅ Complete |
| 5 | Custom detection rules | ✅ Complete |
| 6 | Suricata IDS/IPS integration | 🔄 In progress |
| 7 | pfSense firewall integration | 📄 Documented — verify on current build |
| 8 | **Automated active response** | ✅ Verified — attack disables account live |
| – | VirusTotal enrichment | 📄 Documented — verify on current build |
| – | SCA + Vulnerability Detection | ✅ Complete |
| – | Second attack scenario (MITRE ATT&CK mapped) | ⏳ Planned |

---

## 🏗️ Lab Architecture

The lab runs on VMware on an isolated virtual network:

| VM | Role |
|---|---|
| **Wazuh Server** | Wazuh Manager, Indexer, and Dashboard. Collects and correlates logs from agents, Suricata, and pfSense. |
| **Windows 11 Endpoint** | Runs the Wazuh Agent for system monitoring and log forwarding. Also hosts Sysmon and Suricata. |
| **Kali Linux** | Attacker machine used to simulate threats. |
| **pfSense** | Virtual firewall providing traffic and authentication logs. |
