<h2 align="center">Splunk SIEM Implementation</h2>

<h2 align="center">Overview</h2>

Splunk Enterprise serves as the central analytics engine of this SOC lab. Universal Forwarders (UF) are deployed on endpoints to securely send logs to the indexer.

---

<h2 align="center">Universal Forwarder Deployment</h2>

### **Linux Installation (CLI)**
*Used for LINUX endpoint.*

**Download & Install:**
```bash
wget -O splunkforwarder.deb "https://download.splunk.com/products/universalforwarder/releases/9.3.2/linux/splunkforwarder-9.3.2-d8bb32809498-linux-2.6-amd64.deb"
sudo dpkg -i splunkforwarder.deb
```

---

### **Windows Installation**
*Used for Windows endpoint.*

**Download MSI:**  
[Download Splunk Universal Forwarder](https://www.google.com/url?sa=E&source=gmail&q=https://download.splunk.com/products/universalforwarder/releases/9.3.2/windows/splunkforwarder-9.3.2-d8bb32809498-x64-release.msi&authuser=1)

---

<h2 align="center">Configuration: Adding Monitors</h2>

## Splunk can monitor log sources in two ways.  

---

### **Method 1: CLI Command (Quick & Simple)**

#### **Linux: Watch syslog**
```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog -index main
```

#### **Windows: Watch a JSON file**
```powershell
.\splunk add monitor "C:\path\to\log.json" -index main
```

---

### **Method 2: Editing inputs.conf (Reliable & Persistent)**

---

### **Windows Configuration**
Path:
```
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

```ini
[default]
host = DESKTOP-6DL9VU5

# SYSMON: Ingest native XML to retain rich event data
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = main
renderXml = 1

# SURICATA: Ingest Network IDS alerts
[monitor://C:\Program Files\Suricata\log\eve.json]
disabled = 0
index = main
sourcetype = suricata:json
```

---

### **Linux Configuration**
Path:
```
/opt/splunkforwarder/etc/system/local/inputs.conf
```

```ini
[default]
host = LINUX-MINT-LAB

# Sysmon for Linux logs to syslog by default
[monitor:///var/log/syslog]
disabled = 0
index = main
sourcetype = syslog

# AUTH LOGS: SSH, sudo usage
[monitor:///var/log/auth.log]
disabled = 0
index = main
sourcetype = linux_secure
```

---

<h2 align="center">Troubleshooting</h2>

### **A. Future Logs Time Zone Fix**

**Issue:** Logs appeared 12 hours in the future due to UTC mismatch.  
**Fix:** Add timezone in `splunk-launch.conf`.

```
TZ = Asia/Kolkata
```

---

<h2 align="center">Search Processing Language (SPL) Library</h2>

---

### **A. Unified Normalization Query**

```spl
index=main 
| where _time < now() + 300
| eval LogType=case(
    match(sourcetype, "suricata"), "NETWORK (Suricata)", 
    match(source, "Sysmon") OR match(sourcetype, "Sysmon") OR match(source, "syslog"), "ENDPOINT (Sysmon)", 
    true(), "OTHER")
| search LogType!="OTHER"
| rex "\<Data Name=['\"]Image['\"]\>(?<WinImage>[^\<]+)"
| rex "\<Data Name=['\"]CommandLine['\"]\>(?<WinCLI>[^\<]+)"
| rex "process_name=(?<LinuxImage>\S+)"
| rex "command_line=(?<LinuxCLI>.+)"
| eval Image=coalesce(WinImage, LinuxImage, Image)
| eval CommandLine=coalesce(WinCLI, LinuxCLI, CommandLine)
| eval Activity_Description=case(
    LogType=="ENDPOINT (Sysmon)", "PROCESS: " . coalesce(Image, "Unknown") . " || CMD: " . coalesce(CommandLine, "N/A"), 
    LogType=="NETWORK (Suricata)", "ALERT: " . coalesce(alert_signature, "Unknown"), 
    true(), "Unknown Activity")
| table _time, LogType, host, Activity_Description
```

---

### **B. Threat Hunting**

```spl
index=main source="*Sysmon*" EventCode=1 
| where _time > relative_time(now(), "-60m")
| rex "\<Data Name=['\"]Image['\"]\>(?<ExtractedImage>[^\<]+)" 
| rex "\<Data Name=['\"]CommandLine['\"]\>(?<ExtractedCLI>[^\<]+)"
| eval Image=coalesce(ExtractedImage, Image)
| eval CommandLine=coalesce(ExtractedCLI, CommandLine)
| eval CLI_Lower=lower(CommandLine)
| eval Keywords=""
| eval Keywords=if(match(CLI_Lower, "bypass|hidden|-enc"), mvappend(Keywords, "Obfuscation"), Keywords)
| eval Keywords=if(match(CLI_Lower, "http:|https:|wget|curl"), mvappend(Keywords, "Web Request"), Keywords)
| eval Keywords=if(match(CLI_Lower, "whoami|ipconfig|net user|systeminfo|sc query"), mvappend(Keywords, "Discovery"), Keywords)
| eval Keywords=if(match(CLI_Lower, "\\\\temp\\\\|\\\\downloads\\\\|appdata"), mvappend(Keywords, "Suspicious Path"), Keywords)
| table _time, Image, CommandLine, Keywords
| sort -_time
```

---

### **C. Shell Activity Tracker**

```spl
index=main (source="*Sysmon*" OR sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational") EventCode=1 
| rex "\<Data Name=['\"]Image['\"]\>(?<ExtractedImage>[^\<]+)" 
| eval Image=coalesce(ExtractedImage, Image)
| search Image="*powershell.exe*" OR Image="*cmd.exe*" OR Image="*pwsh*"
| eval ShellType=if(match(Image, "cmd.exe"), "CMD", "PowerShell")
| timechart count by ShellType
```
