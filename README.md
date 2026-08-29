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
| 8 | **Automated active response** | ✅ **Complete** |
| – | VirusTotal enrichment | 📄 Documented — verify on current build |
| – | SCA + Vulnerability Detection | ⏳ Planned |
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

![Lab Architecture](Lab_architecture.png)

*Figure 1: SOC Lab Architecture.*

> All offensive tooling in this project runs only against virtual machines I own, on an isolated lab network.

---

## 🚨 Featured: Automated Active Response

The most involved phase of the lab, and the one I'd most want to talk through in an interview.

**Goal:** when a brute-force pattern is detected, Wazuh automatically disables the targeted Windows account, with a timed auto-recovery so the account is not locked out permanently.

### Design decision — why disable the account instead of blocking the IP

My first plan was to block the attacker's IP at the endpoint firewall. Inspecting the actual Windows Event ID 4625 records showed that the source IP field was **not populated** for these logon failures, so there was no attacker IP available to block at that point in the pipeline.

Rather than document a control that would never fire, I changed the response: disable the account being targeted. The lesson generalises — a response is only as good as the fields your detection actually carries.

### How it works

| Component | Detail |
|---|---|
| **Trigger rule** | Custom rule `100020` — level 10, `frequency=5`, `timeframe=60`, `if_matched_sid 60122`. Fires on 5 failed logons within 60 seconds. |
| **Why correlation, not a single event** | Rule `60122` fires on *one* failed logon. Responding to that would disable an account every time a legitimate user mistyped a password. `100020` responds to the *pattern*, not the noise. |
| **Response script** | Custom `disable-account.cmd` / `.ps1` on the agent. Reads the JSON Wazuh passes on stdin: `add` → disable the account, `delete` → re-enable it. |
| **Auto-recovery** | `<timeout>120</timeout>` with `<timeout_allowed>yes</timeout_allowed>` — Wazuh re-invokes the script with `delete` after 120 seconds, restoring the account automatically. |

**Verified end to end:** Hydra attack → rule `100020` fires → manager dispatches → agent executes the script → account disabled → account automatically restored after the timeout.

### Troubleshooting case study — two stacked root causes

The response failed *silently* for a long time: the rule fired, but nothing happened on the endpoint. Working the pipeline layer by layer surfaced two independent bugs, both of which had to be fixed:

1. **The PowerShell script would not parse.** It contained smart/curly quotes instead of straight quotes, introduced by pasting into the VM without clipboard sharing. PowerShell failed to parse the file every time it was invoked, so it crashed before reaching the account command. Found by running the script by hand with the exact stdin JSON Wazuh sends.
2. **The agent had a stale shared configuration.** A `merged.mg` transfer from the manager had failed, so the agent's `ar.conf` never received the `disable-account` command and silently dropped every dispatch. Fixed by deleting the stale `merged.mg` and forcing a fresh re-sync.

**The technique that cracked it:** I added a throwaway diagnostic command with `<location>server</location>` that simply appended a line to a file on the manager. When the attack filled that file, it proved the manager's dispatch path was healthy — isolating the fault to the agent side and leading directly to the stale-config discovery. Splitting a pipeline in half to find which side is broken is the transferable part of this.

### Limitations

Automatically disabling an account is itself a denial-of-service risk: someone who knows this control exists could deliberately lock out a real user by failing five logins against their username. Mitigations worth applying in a production context: keep the timeout short, use alert-only (not auto-disable) for privileged accounts, and correlate with a log source that *does* carry source IP — such as pfSense or Suricata — so the response can target the attacker rather than the victim's account.

---

## 📚 Phase Documentation

### Setup

[**Wazuh Server & Agent Setup** 📄 PDF Guide](docs/Wazuh_configuration.pdf)
- Deploy Wazuh in a virtualized environment using the official OVA package.
- Configure and troubleshoot Wazuh services, then access the Dashboard for monitoring.
- Install and register endpoint agents to collect logs and centralize security visibility.

### Implementation & Configuration

[**Logs & Sysmon Ingestion** 📄 PDF Guide](docs/Logs&Sysmon_ingestion.pdf)
- Windows Event Logs, key categories, and the Event IDs that matter for detection.
- Deploy Sysmon to capture detailed process and system activity.
- Ingest Sysmon logs into Wazuh for correlation and custom rule-based detection.

[**File Integrity Monitoring** 📄 PDF Guide](docs/File_integrity_monitoring.pdf)
- Configure Wazuh FIM on Windows by defining directories in the agent's `ossec.conf`.
- Enable real-time monitoring with recursion and change reporting.
- Validate by creating, modifying, and deleting files to confirm alerts for each action.

[**Suricata Integration** 📄 PDF Guide](docs/Suricata_integration.pdf)
- IDS for passive detection versus IPS for active blocking.
- Install and configure Suricata on Windows with Npcap and detection rules.
- Forward Suricata EVE JSON alerts into Wazuh for centralized monitoring.

[**pfSense Integration** 📄 PDF Guide](docs/Pfsense_integration.pdf)
- Deploy pfSense as a virtual firewall in VMware to control and monitor network traffic.
- Configure remote syslog and forward pfSense events into Wazuh.
- Write custom decoders and rules to detect allowed, blocked, and authentication events.

[**VirusTotal Integration** 📄 PDF Guide](docs/VirusTotal_integration.pdf)
- Obtain a VirusTotal API key and configure the integration in the Wazuh Manager.
- Monitor directories in real time so new files trigger VirusTotal lookups.
- Enrich alerts with reputation data to speed up triage.

### Detection & Response

[**Brute Force Attack: Simulation, Detection & Defense** 📄 PDF Guide](docs/SSH_Brute_Force.pdf)
- Simulate an SSH brute force attack with Hydra to generate repeated failed logins.
- Detect the activity in Wazuh via Event ID 4625 and correlation rules.
- Apply defensive measures: strong passwords, MFA, account lockout, and active response.

**Active response** is documented in the section above; a dedicated PDF guide is in progress.

> **Important:** perform these activities only in an isolated lab environment or on systems you own and are authorized to test. Never run brute-force activity against third-party or production systems.

---

## 🗺️ Roadmap

- Publish `local_rules.xml` and `local_decoder.xml` as raw files in this repo (currently described inside the PDF guides only).
- Attack log table — timestamp, command run, expected alert, actual alert — as a hit/miss record of detection coverage.
- Enable **SCA (Security Configuration Assessment)** for CIS-benchmark hardening audits.
- Enable **Vulnerability Detection**.
- Add a second attack scenario mapped to **MITRE ATT&CK** technique IDs.
- Publish the Active Response phase guide.

---

## Conclusion

This SOC home lab demonstrates how open-source tools combine into a functional security monitoring, detection, and response environment. **Wazuh** acts as the central SIEM, **pfSense** as the firewall, **Suricata** as the IDS/IPS, and **Sysmon** for endpoint visibility, replicating the key components of a modern SOC. **VirusTotal** enrichment and **File Integrity Monitoring** add context and detection depth.

The brute-force exercise validated the full analyst workflow end to end: the environment ingests and correlates logs, raises meaningful alerts, and — with active response — acts on them automatically. Building that last piece also meant debugging a silent failure across the manager–agent pipeline, which taught more about how Wazuh actually works than any of the phases that succeeded first time.

Beyond the tooling, this project reinforced core SOC practices: log analysis, alert triage, rule tuning, detection engineering, and threat hunting.

**Note:** This project is for educational purposes only. Do not use these techniques for unauthorized activities.

---

## 📄 Full Documentation

Download the complete SOC Home Lab guide:
[📥 SOC_Home_Lab_Guide.pdf](docs/Soc_Home_LAB.pdf)

## 📌 Connect with Me

[LinkedIn](https://www.linkedin.com/in/golsaf-bensekhar-1b153b1ab/) · [Medium](https://medium.com/@golsafbensekhar)
