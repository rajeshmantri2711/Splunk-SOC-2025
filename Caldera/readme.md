
<h2 align="center">CALDERA</h2>

CALDERA is an open source adversary emulation platform developed by MITRE. It is designed to simulate real world attack behavior using techniques mapped to the MITRE ATT and CK framework. 
CALDERA enables security teams to safely execute controlled attack scenarios in order to evaluate detection and response capabilities.

The platform operates using a command and control server and endpoint agents. These agents execute predefined techniques as part of adversary profiles, generating realistic attack telemetry for SOC validation and testing.

### Purpose and Value

1. Emulates real attacker behavior aligned with MITRE ATT&CK
2. Validates SOC detection and response workflows
3. Generates realistic attack telemetry for SIEM analysis
4. Improves incident response readiness through adversary simulation
5. Enhances blue team skills in a controlled environment

### Operational Overview

1. CALDERA server manages operations and adversary logic
2. Endpoint agents execute attack techniques
3. Abilities represent individual attack actions
4. Adversary profiles define attack sequences
5. Operations automate full attack chains
6. Results can be forwarded to SIEM platforms such as Splunk

### Installation and Configuration

Official repository  
https://github.com/mitre/caldera

1. Deploy CALDERA on a Linux based system
2. Install Python version 3.8 or higher
3. Clone the CALDERA repository
4. Install required dependencies
5. Start the CALDERA server
6. Access the web interface for management

Configuration includes enabling plugins, creating adversary profiles, deploying agents to endpoints and integrating logs with Splunk for detection analysis.

### SOC Integration

CALDERA is used within SOC environments to simulate advanced attack scenarios and validate security monitoring. The generated telemetry supports rule testing, alert tuning and end to end incident 
response exercises in Splunk based SOC architectures.
