\# Wazuh SOC Lab



Home SOC lab focused on security monitoring, detection, investigation and incident response using Wazuh.



\## Overview



This project consists of a cybersecurity home lab built with virtual machines to practice tasks commonly performed by SOC analysts.



Wazuh is used as the central security platform to collect endpoint information, detect security events, monitor system changes, identify vulnerabilities and investigate potential security incidents.



The goal is not only to deploy the tools, but to understand the complete security workflow:



\*\*Monitor → Detect → Investigate → Respond → Verify\*\*



\---



\## Objectives



The main objectives of this laboratory are:



\* Understand the architecture and operation of Wazuh.

\* Monitor Linux endpoints.

\* Collect and analyse security events.

\* Detect file modifications using File Integrity Monitoring (FIM).

\* Assess endpoint security configuration using Security Configuration Assessment (SCA).

\* Collect system and software inventory using Syscollector.

\* Identify vulnerabilities using Vulnerability Detection.

\* Investigate alerts from a SOC analyst perspective.

\* Simulate controlled security scenarios.

\* Create and test custom Wazuh detection rules.

\* Map detections to MITRE ATT\&CK.

\* Practice basic incident response and verification procedures.

\* Document procedures, findings and lessons learned.



\---



\## Architecture



The current laboratory consists of two virtual machines connected to the same local network.



```text

&#x20;                        LOCAL NETWORK

&#x20;                      192.168.1.0/24

&#x20;                             │

&#x20;               ┌─────────────┴─────────────┐

&#x20;               │                           │

&#x20;        192.168.1.35                 192.168.1.36

&#x20;               │                           │

&#x20;      ┌────────▼────────┐          ┌───────▼────────┐

&#x20;      │  wazuh-server   │          │   soc-server   │

&#x20;      │                 │          │                │

&#x20;      │ Amazon Linux    │          │ Ubuntu 26.04   │

&#x20;      │ 2023            │          │                │

&#x20;      │                 │          │ Wazuh Agent    │

&#x20;      │ Wazuh Manager   │◄─────────│                │

&#x20;      │ Wazuh Indexer   │          │ FIM            │

&#x20;      │ Wazuh Dashboard │          │ SCA            │

&#x20;      │                 │          │ Syscollector    │

&#x20;      └─────────────────┘          └────────────────┘

```



\### Communication



The Wazuh Agent installed on `soc-server` communicates with the Wazuh Manager running on `wazuh-server`.



The Manager processes the collected security information and makes the resulting data available through the Wazuh Indexer and Dashboard.



\---



\## Lab Environment



| Host           | Operating System     | IP Address     | Role                                 |

| -------------- | -------------------- | -------------- | ------------------------------------ |

| `wazuh-server` | Amazon Linux 2023.12 | `192.168.1.35` | Wazuh Manager, Indexer and Dashboard |

| `soc-server`   | Ubuntu 26.04.1 LTS   | `192.168.1.36` | Monitored endpoint / Wazuh Agent     |



\### Wazuh Version



\*\*Wazuh 4.14.7\*\*



| Component       | Version |

| --------------- | ------- |

| Wazuh Manager   | 4.14.7  |

| Wazuh Indexer   | 4.14.7  |

| Wazuh Dashboard | 4.14.7  |

| Wazuh Agent     | 4.14.7  |



\---



\## Security Capabilities



The current laboratory includes the following capabilities:



\### File Integrity Monitoring



Wazuh monitors selected system directories and detects changes to monitored files.



A controlled modification of `/etc/wazuh-test.txt` was used to verify that FIM was working correctly.



\### Security Configuration Assessment



Security Configuration Assessment (SCA) is enabled on the Ubuntu endpoint to evaluate security configuration against defined checks.



\### System Inventory



Syscollector provides information about the endpoint, including operating system, hardware, network configuration and installed software.



\### Vulnerability Detection



Wazuh Vulnerability Detection is enabled to identify vulnerabilities affecting installed software.



The laboratory has also been used to investigate how vulnerability intelligence updates affect the current vulnerability state reported for an endpoint.



\---



\## Detection Scenarios



The following scenarios are planned for the laboratory:



\* \[x] File Integrity Monitoring

\* \[ ] SSH authentication attacks

\* \[ ] Failed authentication monitoring

\* \[ ] Privilege escalation

\* \[ ] Persistence mechanisms

\* \[ ] Suspicious process and command execution

\* \[x] Vulnerability detection and investigation

\* \[ ] Custom Wazuh detection rules

\* \[ ] MITRE ATT\&CK mapping

\* \[ ] Windows endpoint monitoring

\* \[ ] Controlled attack simulation from a dedicated attacker VM

\* \[ ] Full attack → detection → investigation → response workflow



Each scenario will document:



1\. Objective

2\. Environment

3\. Attack or event simulation

4\. Expected detection

5\. Evidence collected

6\. Investigation

7\. Response

8\. Verification

9\. Lessons learned



\---



\## Roadmap



\### Phase 1 — Endpoint Hardening



\* \[x] Ubuntu server deployment

\* \[x] User and privilege configuration

\* \[x] SSH hardening

\* \[x] UFW configuration

\* \[x] Service review

\* \[x] Security audit scripts



\### Phase 2 — Wazuh Deployment



\* \[x] Wazuh server deployment

\* \[x] Wazuh Agent deployment

\* \[x] Agent registration

\* \[x] Manager communication

\* \[x] Dashboard access



\### Phase 3 — Security Monitoring



\* \[x] File Integrity Monitoring

\* \[x] Security Configuration Assessment

\* \[x] System inventory

\* \[x] Vulnerability Detection



\### Phase 4 — SOC Detection Scenarios



\* \[ ] SSH attacks

\* \[ ] Authentication monitoring

\* \[ ] Privilege escalation

\* \[ ] Persistence

\* \[ ] Suspicious processes

\* \[ ] Custom detection rules

\* \[ ] MITRE ATT\&CK mapping



\### Phase 5 — Advanced Lab



\* \[ ] Windows endpoint

\* \[ ] Sysmon

\* \[ ] Dedicated attacker VM

\* \[ ] Controlled attack simulations

\* \[ ] Multi-stage detection scenarios

\* \[ ] Full incident response exercises



\---



\## Lessons Learned



This section will be updated throughout the project with technical problems encountered, their investigation and their resolution.



Examples include:



\* Wazuh vulnerability feed update and disk-space management.

\* Investigating discrepancies between upstream software versions and vendor security status.

\* Understanding the relationship between endpoint inventory and vulnerability detection.

\* Analysing Wazuh alerts and their underlying logs.



\---



\## Project Status



\*\*Current stage:\*\* Security monitoring and vulnerability detection.



The basic Wazuh infrastructure is operational and the laboratory is ready to begin structured SOC detection scenarios.



The next planned scenario is \*\*SSH authentication monitoring and controlled brute-force simulation\*\*.



