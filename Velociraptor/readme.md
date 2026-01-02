<h2 align="center">VELOCIRAPTOR</h2>

Velociraptor is an open source digital forensics and incident response platform focused on endpoint visibility and threat hunting. It provides centralized control over forensic data collection across multiple operating systems, enabling rapid investigation and response during security incidents.

The platform uses a lightweight agent and a central server to collect and analyze endpoint data in real time using a powerful query language.

### Purpose and Value

1. Provides deep visibility into endpoint activity
2. Enables rapid forensic data acquisition
3. Supports large scale threat hunting operations
4. Enhances incident response efficiency
5. Complements SIEM platforms such as Splunk

### Operational Overview

1. Central server manages endpoint communications
2. Agents run on Windows Linux and macOS systems
3. Queries are executed using Velociraptor Query Language
4. Endpoint data is collected in real time
5. Results are stored and analyzed centrally
6. Telemetry can be forwarded to Splunk for correlation

### Installation and Configuration

Official documentation  
https://docs.velociraptor.app/downloads/

1. Download the appropriate Velociraptor binary
2. Generate server and client configuration files
3. Start the Velociraptor server
4. Deploy agents to endpoints
5. Access the web interface for investigations

Configuration includes client enrollment, artifact selection, scheduled hunts and integration with Splunk for SOC level visibility.

### SOC Integration

Velociraptor enables SOC teams to perform live response and forensic investigations directly from SIEM alerts. It allows rapid pivoting from detection to evidence collection, significantly reducing investigation time and improving accuracy during security incidents.

