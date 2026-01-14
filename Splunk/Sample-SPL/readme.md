
## For Showing active endpoints

```
index=* sourcetype IN ("linux_sysmon_xml","XmlWinEventLog")
| stats latest(_time) AS last_seen BY host
| where now()-last_seen <= 300
| stats dc(host) AS "Active Endpoints"
```
## For Showing Inactive endpoints
```
 index=* sourcetype IN ("linux_sysmon_xml", "XmlWinEventLog")
| stats latest(_time) AS last_seen BY host
| where now()-last_seen <= 300
| eval last_seen=strftime(last_seen, "%Y-%m-%d %H:%M:%S")
| table host last_seen
```
## SPL normalization

Before we can normalize Sysmon logs for linux we do that by 
In the Splunk Server web ui
Settings → Fields → Field Extractions → New
Destination app: search
Name: linux_sysmon
Apply to: sourcetype
Named: linux_sysmon_xml
Type: inline
extraction/transformation: <EventID>(?<EventID>\d+)</EventID>

Do the same for extracting the image and command

In extarction/transformation field, put


`<Data Name="Image">(?<image>[^<]+)</Data>`

`<Data Name="CommandLine">(?<command>[^<]+)</Data>`

Event id along with the command executed 

Without extraction SPL
```
index=* sourcetype=linux_sysmon_xml EventID=1
| rex field=_raw "<Data Name=\"Image\">(?<image>[^<]+)</Data>"
| rex field=_raw "<Data Name=\"CommandLine\">(?<command>[^<]+)</Data>"
| eval time=strftime(_time,"%Y-%m-%d %H:%M:%S")
| table time host EventID image command
```
## With extraction spl
```
index=* sourcetype=linux_sysmon_xml EventID=1
| table _time host EventID image command
```
For suricata scan
```
index=* sourcetype=suricata
| where isnotnull(src_ip) AND isnotnull(dest_port)
| bin _time span=5m
| stats dc(dest_port) AS unique_ports BY src_ip _time
| where unique_ports >= 10
| sort - unique_ports
```
For whoami 
```
index=* sourcetype=linux_sysmon_xml 
(command="*whoami*" OR image="*/whoami")
| table _time host EventID image command
| sort - _time
```

