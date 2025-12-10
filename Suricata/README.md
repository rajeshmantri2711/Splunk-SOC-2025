<h1 align="center">Suricata NIDS Configuration</h1>

<h2 align="center">Goal of this Module</h2>

This module sets up Suricata. Suricata is a Network Intrusion Detection System (NIDS). It acts as a security camera for the network wire.

While Sysmon monitors the endpoint (Processes and Files), Suricata monitors the network traffic.

It provides three key functions:
1.  **Signature Detection**
    It detects malware Command & Control (C2) traffic. It catches threats even if the antivirus misses them.
2.  **Protocol Analysis**
    It logs metadata for every DNS request, HTTP call, and TLS handshake. It saves this to `eve.json`.
3.  **Correlation**
    It sends logs to Splunk. By combining Suricata with Sysmon, we achieve the "Golden Triangle" of visibility (Endpoint + Network + Logs).

<h2 align="center">Linux Installation</h2>

Use these steps for Linux sensors (Debian/Ubuntu/Mint).

1.  **Add the Repository**
    ```bash
    sudo add-apt-repository ppa:oisf/suricata-stable
    sudo apt update
    ```

2.  **Install Suricata**
    ```bash
    sudo apt install suricata -y
    ```

3.  **Update Rules**
    ```bash
    sudo suricata-update
    ```

4.  **Start the Service**
    ```bash
    sudo systemctl enable suricata
    sudo systemctl start suricata
    ```

<h2 align="center">Windows Installation</h2>

Use these steps for Windows 10 endpoints.

<h3>1. Install Npcap</h3>

* Download Npcap from the nmap website.
* Run the installer.
* **Critical:** You must check the box "Install Npcap in WinPcap API-compatible Mode".
* Suricata will fail to start without this.

<h3>2. Install Suricata</h3>

* Download the MSI installer from the Suricata website.
* Install it to the default location:
    `C:\Program Files\Suricata\`

<h3>3. Fix Configuration Paths</h3>

Windows services can crash if paths are relative. You must use absolute paths.

1.  Open the configuration file:
    `C:\Program Files\Suricata\suricata.yaml`

2.  Update these variable paths:
    ```yaml
    default-log-dir: C:\\Program Files\\Suricata\\log
    default-rule-path: C:\\Program Files\\Suricata\\rules
    classification-file: C:\\Program Files\\Suricata\\classification.config
    reference-config-file: C:\\Program Files\\Suricata\\reference.config
    ```

<h2 align="center">Running on Windows</h2>

The Windows Service is unstable with network drivers at boot. We use the Task Scheduler instead.

1.  **Create the Start Script**
    Create a file named `start_suricata.bat`.
    Paste the code below. Replace the IP with your interface IP.

    ```batch
    @echo off
    cd /d "C:\Program Files\Suricata"
    Suricata.exe -c "C:\Program Files\Suricata\suricata.yaml" -i 192.168.100.249
    ```

2.  **Configure Task Scheduler**
    * **Trigger:** At Startup.
    * **Security:** Run with highest privileges (Admin).
    * **User:** Run whether user is logged on or not.

<h2>Rule Management</h2>

Suricata detects threats using signatures.

<h3>Emerging Threats (ET) Open Ruleset</h3>

This is the industry standard open-source ruleset. It detects C2, exploit kits, and phishing.

Update it with this command:
```bash
suricata-update
```

<h3 align="center">Custom Rules</h3>

We use local rules to verify the pipeline. Add these to `local.rules`.

**1. Ping Alert (Connectivity Test)**
Triggers on any ICMP packet.
```text
alert icmp any any -> any any (msg:"LAB TEST: Ping Detected"; sid:1000001; rev:1;)
```

**2. Malware Download Simulation**
Triggers if a file named `malware.exe` is downloaded.
```text
alert http any any -> any any (msg:"POSSIBLE MALWARE DOWNLOAD"; content:"malware.exe"; http_uri; sid:1000002; rev:1;)
```

**3. Suspicious SSH Port**
Triggers on SSH traffic to port 2222.
```text
alert tcp any any -> any 2222 (msg:"SUSPICIOUS SSH Port 2222"; sid:1000003; rev:1;)
```

<h2 align="center">Output and Logs</h2>

Suricata writes to a unified JSON file. This file contains Alerts, DNS, HTTP, and Flow data.

The Splunk Universal Forwarder must monitor this file.

* **Windows Path:**
    `C:\Program Files\Suricata\log\eve.json`

* **Linux Path:**
    `/var/log/suricata/eve.json`
