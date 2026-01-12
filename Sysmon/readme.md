
<h1 align="center">System Monitor (Sysmon) Implementation</h1>

<h2 align="center">Goal of this Module</h2>

System Monitor (Sysmon) is a system service. It monitors system activity. It works on both Windows and Linux.

Standard logging is often not enough for deep analysis. Sysmon is better because it provides:

1.  **Hash Logging**
    It calculates MD5 and SHA256 hashes for every program. This helps match threat intelligence.
2.  **Parent-Child Trees**
    It shows which process started another process.
3.  **Command Line Arguments**
    It logs the exact script or command used by an attacker.
4.  **Network Correlation**
    It links a specific Process ID to a network connection.

<h2 align="center">Windows Installation</h2>

We use the SwiftOnSecurity configuration. This is the industry standard. It filters out safe background noise.

1.  **Download Configuration**
    Run this in PowerShell as Administrator.
    ```powershell
    Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Program Files\Sysmon\sysmonconfig.xml"
    ```

2.  **Install Sysmon**
    Navigate to the folder and run the installer.
    ```powershell
    cd "C:\Program Files\Sysmon"
    .\Sysmon64.exe -i sysmonconfig.xml
    ```

3.  **Verify Installation**
    Check if the service is running.
    ```powershell
    Get-Service Sysmon64
    Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
    ```

<h2 align="center">Linux Installation</h2>

Sysmon for Linux uses eBPF to monitor the kernel. We use the MSTIC configuration.

1.  **Register Microsoft Repository**
    ```bash
    wget -q https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
    sudo dpkg -i packages-microsoft-prod.deb
    sudo apt-get update
    ```

2.  **Install Sysmon**
    ```bash
    sudo apt-get install sysmonforlinux
    ```

3.  **Apply Configuration**
    Download and apply the config file.
    ```bash
    wget https://raw.githubusercontent.com/microsoft/MSTIC-Sysmon/main/linux/configs/main.xml -O sysmonconfig.xml
    sudo sysmon -i sysmonconfig.xml
    ```

<h2 align="center">Key Event IDs</h2>

We filter for these specific events in Splunk.

**ID 1: Process Creation**
The most important log. It shows the image path, user, and hashes. It detects malware execution.

**ID 3: Network Connection**
Maps a process to an external IP. This is critical for detecting Command & Control beacons.

**ID 11: File Create**
Logs when a file is created. This is useful for detecting downloads in temporary folders.

**ID 13: Registry Event**
Windows Only. It monitors persistence mechanisms like Run keys.

**ID 22: DNS Query**
Logs the domain name requested by a process. This spots botnets.

<h2 align="center">Splunk Integration</h2>

We must configure the Splunk Forwarder to read these logs correctly.

<h3 align="center">Windows Data Flow</h3>

Sysmon uses an XML Event Channel.
Add this to `inputs.conf`:

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
renderXml = 1
```

<h3 align="center">Linux Data Flow</h3>

Sysmon writes to the standard syslog.
Add this to `/opt/splunkforwarder/etc/system/local/inputs.conf`:

```ini
[monitor:///var/log/syslog]
disabled = 0
sourcetype = syslog
```

<h2 align="center">Data Normalization</h2>

Windows logs are XML. Linux logs are text. We use a master query to merge them.

**Extraction Strategy**
1.  **Windows:** Extract fields from XML tags.
2.  **Linux:** Extract fields using Regex.
3.  **Unification:** Use `coalesce` to merge them into shared fields.

**Master SPL Example**
```splunk
index=main 
| rex "\<Data Name=['\"]Image['\"]\>(?<WinImage>[^\<]+)" 
| rex "process_name=(?<LinuxImage>\S+)"
| eval Image=coalesce(WinImage, LinuxImage)
```
