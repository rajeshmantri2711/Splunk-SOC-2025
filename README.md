<h1 align="center">SOC Automation Lab</h1>

<hr>

<h2 align="center">Project Overview</h2>

This project simulates a fully functional Security Operations Center (SOC) environment designed to engineer detection pipelines, analyze adversarial behavior, and perform digital forensics and incident response.

The objective was to build a complete telemetry lifecycle from log generation on mixed OS endpoints to ingestion, normalization, and visualization in a centralized Splunk SIEM and simultae real time attacks on the endpoints.

<hr>
<h2 align="center">Lab Architecture</h2>
<div align="center">

| Role           | OS               | Components / Services |
|----------------|------------------|------------------------|
| **Central Server** | Ubuntu 22.04 LTS | Splunk Enterprise (Indexer/Search Head)<br>Velociraptor Server<br>Caldera C2 Server |
| **Linux Agent**    | Linux Mint       | Splunk Universal Forwarder<br>Sysmon for Linux<br>Suricata (NIDS)<br>Velociraptor Client<br>Caldera Agent (Sandcat)  |
| **Windows Agent**  | Windows 10 LTS   | Splunk Universal Forwarder<br>Sysmon (SwiftOnSecurity Config)<br>Suricata (NIDS)<br>Velociraptor Client<br>Caldera Agent (Sandcat) |

</div>

<h2 align="center">Tools</h2>

### [ Splunk Enterprise & Universal Forwarder ](Splunk)
**Role: Security Information & Event Management (SIEM)**  
Serves as the central data loging ground of the SOC. It ingests multi source logs (XML, JSON, Syslog) via the Universal Forwarder. A custom Master SPL query was developed to normalize log formats into a unified Endpoint data model.

### [ Suricata ](Suricata)
**Role: Network Intrusion Detection System (NIDS)**  
Deployed on the Windows and Linux endpoints using to inspect network traffic. Configured with the Emerging Threats open ruleset to detect signature based network threats and generate  alerts for Splunk log ingestion.

### [ Sysmon](Sysmon)
**Role: Advanced Endpoint Telemetry**  
Installed on both Windows and Linux endpoints. It provides deep visibility into process creation, network connections, and file modifications. On Windows, it utilizes the SwiftOnSecurity configuration to filter noise and highlight high fidelity threats such as PowerShell obfuscation.

### [ MITRE Caldera ](Caldera)
**Role: Adversary Emulation (Red Teaming)**  
An automated C2 framework used to simulate realistic attacks. It executes adversary profiles (e.g., Discovery, Hunter) to validate that the SOC pipeline correctly detects and alerts on malicious behavior mapped to the MITRE ATT&CK framework.

### [ Velociraptor ](Velociraptor) 
**Role: Digital Forensics & Incident Response (DFIR)**  
A powerful forensic tool used for endpoint hunting and evidence collection. It supports targeted acquisition of artifacts such as MFT, Registry, and Event Logs, enabling deep investigation of incidents identified by the SIEM.

### Atomic Red Team
**Role: Atomic Test Simulation**  
A library of small, scripted tests used for validating specific detections like T1003 Credential Dumping before performing full scale emulated attack campaigns with Caldera.

<hr>


