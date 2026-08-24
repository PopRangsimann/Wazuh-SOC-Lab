# Wazuh-SOC-Lab

Welcome to my Wazuh SOC Lab repository!
This project documents my journey of deploying, configuring, and using different security tools to build a functional Security Operations Center (SOC) monitoring and detection environment.

## 🎯 Skills Demonstrated

SIEM/XDR deployment and administration (Wazuh) · endpoint monitoring (Sysmon, Windows Event Logs) · File Integrity Monitoring · log ingestion and correlation · attack simulation (recon + brute force) · alert triage and incident investigation · threat intelligence enrichment (VirusTotal) · network IDS (Suricata) · firewall log integration (pfSense).

**Tools:** Wazuh (SIEM/XDR), Suricata (IDS/IPS), pfSense (Firewall), Sysmon, VirusTotal API, Hydra, Nmap, Wireshark — on VMware, with a Windows 11 endpoint and a Kali Linux attacker.

## 📊 Project Status

> Update each row to reflect your current build. Only mark a phase ✅ once it is running on your live lab **and** you can explain it in an interview.

| Phase | Component | Status |
|-------|-----------|--------|
| 1 | Wazuh SIEM (Manager, Indexer, Dashboard) | ✅ Complete |
| 2 | Windows agent + Sysmon ingestion | ✅ Complete |
| 3 | File Integrity Monitoring (FIM) | ✅ Complete |
| 4 | Attack simulation (recon + SSH brute force) | ✅ Complete |
| 5 | Custom detection rules | 🔄 In progress |
| 6 | Suricata IDS/IPS integration | 📄 Documented — verify on current build |
| 7 | pfSense firewall integration | 📄 Documented — verify on current build |
| – | VirusTotal enrichment | 📄 Documented — verify on current build |

## 🏗️ Lab Architecture

The lab is built using VMware VMs and includes the following components:

- **Wazuh Server** — Runs the central Wazuh Manager, Indexer, and Dashboard. Collects and correlates logs from agents, Suricata, and pfSense.
- **Windows Endpoint (Windows 11 Pro)** — Runs the Wazuh Agent for system monitoring and log forwarding.
- **Attacker Machine (Kali Linux)** — Used to simulate threats.
- **pfSense Firewall** — Provides firewall logs; integrated into Wazuh for anomaly detection.
- **Suricata IDS/IPS** — Monitors network traffic; sends IDS alerts to Wazuh.

![Lab Architecture](Lab_architecture.png)

*Figure 1: SOC Lab Architecture.*

## 🗂️ Repository Index

- **Setup & configuration guides** → [`docs/`](docs/)
- **Incident investigation write-ups** → [`incidents/`](incidents/)
- **Custom detection rules & sample configs** → [`detection-rules/`](detection-rules/)
- **Attack test log** → [`attack-log.md`](attack-log.md)

## 🛠️ Wazuh Setup

[Step 1 — Wazuh Server & Agent setup 📄 PDF Guide](docs/Wazuh_configuration.pdf)

**Summary:**
- Deploy Wazuh in a virtualized environment using the official OVA package.
- Configure and troubleshoot Wazuh services, then access the Dashboard for monitoring.
- Install and register endpoint agents to collect logs and centralize security visibility.

## 🔌 Implementation & Configuration

[Suricata Integration 📄 PDF Guide](docs/Suricata_integration.pdf)

**Summary:**
- Use IDS for passive detection and IPS for active blocking of threats.
- Install and configure Suricata on Windows with Npcap and detection rules.
- Integrate Suricata logs with Wazuh to centralize monitoring and alerts.

[pfSense Integration 📄 PDF Guide](docs/Pfsense_integration.pdf)

**Summary:**
- Deploy pfSense as a virtual firewall in VMware to control and monitor network traffic.
- Configure remote logging and forward pfSense events into Wazuh for analysis.
- Create custom decoders and rules in Wazuh to detect allowed, blocked, and authentication events.

[VirusTotal Integration 📄 PDF Guide](docs/VirusTotal_integration.pdf)

**Summary:**
- Obtain a VirusTotal API key and configure it in the Wazuh Manager for integration.
- Set up Wazuh agents to monitor directories in real time and trigger VirusTotal lookups.
- Enrich alerts with VirusTotal reputation data to speed up triage and threat analysis.

[File Integrity Monitoring 📄 PDF Guide](docs/File_integrity_monitoring.pdf)

**Summary:**
- Configure Wazuh File Integrity Monitoring (FIM) on Windows by defining directories in the agent's `ossec.conf`.
- Enable real-time monitoring with recursion and change reporting for files and subdirectories.
- Validate by creating, modifying, and deleting files to confirm Wazuh generates alerts for each action.

[Logs & Sysmon Ingestion 📄 PDF Guide](docs/Logs&Sysmon_ingestion.pdf)

**Summary:**
- Understand Windows Event Logs, key categories, and critical Event IDs for visibility into system and security activities.
- Deploy Sysmon to capture detailed system events and enhance detection of suspicious or attacker behavior.
- Ingest Sysmon logs into Wazuh for centralized monitoring, correlation, and custom rule-based threat detection.

## 🔐 Brute Force Attack: Simulation, Detection & Defense

[Brute Force Attack Simulation & Wazuh Investigation 📄 PDF Guide](docs/SSH_Brute_Force.pdf)

**Summary:**
- Simulate an SSH brute force attack in a controlled lab using Hydra to generate repeated failed login attempts.
- Detect malicious activity in Wazuh through alerts, Windows Event Logs (e.g. Event ID 4625), and correlation rules highlighting authentication failures.
- Apply defensive measures such as strong passwords, MFA, account lockouts, and Wazuh active responses to prevent and mitigate brute force threats.

**Important:** Perform these activities only in your isolated lab environment or on systems you own / are authorized to test. Never run brute-force activity against third-party or production systems.

## 📝 Incident Write-ups

Full investigation reports reconstructing each simulated attack — timeline, alert analysis, impact, and recommendations:

- [SSH Brute-Force Attack and Post-Compromise Activity](incidents/2026-08-24-ssh-bruteforce.md) — recon (undetected) → brute force → account lockout → successful login → post-exploitation, mapped to Wazuh rules and MITRE ATT&CK.

## ✅ Conclusion

This SOC home lab project demonstrates how open-source tools can be combined to build a functional security monitoring and detection environment. By integrating **Wazuh** as the central SIEM, **pfSense** as the firewall, **Suricata** as the IDS/IPS, and **Sysmon** for endpoint visibility, the lab replicates key components of a modern SOC. The addition of **VirusTotal** enrichment and **File Integrity Monitoring** further enhances detection capabilities and contextual analysis.

Through threat-simulation exercises (such as brute-force attack detection), the lab validates that the system can not only ingest and correlate logs but also generate meaningful alerts — reflecting a realistic analyst workflow: detecting, investigating, and proposing defensive countermeasures.

Beyond technical skills, this project reinforces core SOC analyst practices: log analysis, alert triage, rule tuning, and threat-hunting queries.

**Note:** This project is for educational purposes only. Do not use these techniques for unauthorized activities.

## 📄 Full Documentation

Download the complete SOC Home Lab guide here:
[📥 SOC_Home_Lab_Guide.pdf](docs/Soc_Home_LAB.pdf)

## 📌 Connect with Me

[LinkedIn](https://www.linkedin.com/in/golsaf-bensekhar-1b153b1ab/) · [Medium](https://medium.com/@golsafbensekhar)
