<h2 align="center">Splunk SIEM Implementation</h2>

<h2 align="center">Overview</h2>

Splunk Enterprise serves as the central analytics engine of this SOC lab. Universal Forwarders (UF) are deployed on endpoints to securely send logs to the indexer.

---
### Navigation
- [Universal Forwarder Deployment](#universal-forwarder-deployment)
- [Configuration: Adding Monitors](#configuration-adding-monitors)
- [Troubleshooting](#troubleshooting)
- [SPL Library](#search-processing-language-spl-library)
- [Dashboard](#dashboard)

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

### SPL for Normalized or Combined Showcase of logs
```
index=* ( sourcetype=linux_sysmon_xml OR sourcetype=XmlWinEventLog )
| eval image=coalesce(Image, image)
| eval command=coalesce(CommandLine, command)
| table _time host EventID image command
| sort -_time
```
- [output](Images/Normalized_output.png)

### How to recreate it

#### SPL Without Extraction
```
index=* sourcetype=linux_sysmon_xml EventID=1
| rex field=_raw "<Data Name=\"Image\">(?<image>[^<]+)</Data>"
| rex field=_raw "<Data Name=\"CommandLine\">(?<command>[^<]+)</Data>"
| eval time=strftime(_time,"%Y-%m-%d %H:%M:%S")
| table time host EventID image command
```
#### SPL With Extraction
```
index=* sourcetype=linux_sysmon_xml EventID=1
| table _time host EventID image command
```

#### Steps to Create Field Extractions in Splunk Web UI

Create each field extraction separately using the following steps:

1. Navigate to: ` Settings → Fields → Field Extractions → New `
2. Destination app: search
3. Apply to: sourcetype → `linux_sysmon_xml`
4. Type: inline

Field Extraction 1: EventID
- Name: `linux_sysmon_EventID`
- Extraction/Transformation:
```
<EventID>(?<EventID>\d+)</EventID>
```

Field Extraction 2: Image
- Name:`linux_sysmon_Image`
- Extraction/Transformation:
```
<Data Name="Image">(?<image>[^<]+)</Data>
```
Field Extraction 3: CommandLine
- Name: `linux_sysmon_CommandLine`
- Extraction/Transformation:
```
<Data Name="CommandLine">(?<command>[^<]+)</Data>
```

### Dashboard
---


