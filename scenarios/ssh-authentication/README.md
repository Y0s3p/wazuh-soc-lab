\# SSH Authentication Monitoring



\## Objective



Simulate repeated SSH authentication attempts against a monitored Linux endpoint and validate Wazuh's ability to:



\* Detect invalid SSH authentication attempts.

\* Correlate multiple authentication failures.

\* Identify a brute-force pattern.

\* Map the activity to MITRE ATT\&CK.

\* Automatically block the source IP.

\* Remove the block after the configured timeout.



The scenario follows the SOC workflow:



\*\*Simulate → Detect → Investigate → Respond → Verify\*\*



\---



\## Environment



| Component                | Details             |

| ------------------------ | ------------------- |

| Wazuh Manager            | `wazuh-server`      |

| Wazuh Manager IP         | `192.168.1.35`      |

| Monitored endpoint       | `soc-server`        |

| Endpoint IP              | `192.168.1.36`      |

| Attacker/simulation host | Windows workstation |

| Source IP                | `192.168.1.34`      |

| Protocol                 | SSH                 |

| SSH port                 | `22`                |

| Wazuh version            | `4.14.7`            |

| Endpoint OS              | Ubuntu 26.04.1 LTS  |



\---



\## Attack / Event Simulation



The simulation was performed from the Windows workstation against the SSH service running on `soc-server`.



Several connection attempts were made using non-existent usernames:



```text

usuario10

usuario11

usuario12

usuario13

usuario14

```



The SSH service generated events such as:



```text

Invalid user usuario10 from 192.168.1.34

Invalid user usuario11 from 192.168.1.34

Invalid user usuario12 from 192.168.1.34

Invalid user usuario13 from 192.168.1.34

```



The endpoint was configured to collect system logs through `journald`.



\---



\## Detection



\### Rule 5710 — Invalid User



Individual attempts were detected by Wazuh using:



```text

Rule ID: 5710

Level: 5

Description: sshd: Attempt to login using a non-existent user

```



Example:



```text

Src IP: 192.168.1.34

User: usuario2

```



This demonstrates that Wazuh successfully receives SSH authentication events from `journald` and applies the appropriate SSH decoder and detection rule.



\### Rule 5712 — Brute Force Detection



After multiple authentication attempts from the same source, Wazuh correlated the events and generated:



```text

Rule ID: 5712

Level: 10

Description: sshd: brute force trying to get access to the system. Non existent user.

```



The alert contained the previous SSH authentication failures, demonstrating event correlation rather than isolated event detection.



The rule is mapped by Wazuh to:



```text

MITRE ATT\&CK

T1110 — Brute Force

Credential Access

```



\---



\## Investigation



The investigation identified the following characteristics:



| Field                | Value                       |

| -------------------- | --------------------------- |

| Source IP            | `192.168.1.34`              |

| Target               | `soc-server`                |

| Target IP            | `192.168.1.36`              |

| Protocol             | SSH                         |

| Source port          | Dynamic                     |

| Target port          | `22`                        |

| Attempted users      | Multiple non-existent users |

| Initial detection    | Rule 5710                   |

| Correlated detection | Rule 5712                   |

| Severity             | Level 10                    |



The Wazuh alert showed multiple SSH events grouped into the brute-force detection.



Example event:



```text

Invalid user usuario13 from 192.168.1.34

```



The associated correlated alert included previous attempts from the same source.



\---



\## Response



An Active Response was configured in the Wazuh Manager to respond specifically to Rule 5712:



```xml

<active-response>

&#x20; <disabled>no</disabled>

&#x20; <command>firewall-drop</command>

&#x20; <location>local</location>

&#x20; <rules\_id>5712</rules\_id>

&#x20; <timeout>300</timeout>

</active-response>

```



\### Response behavior



When Rule 5712 is triggered:



1\. Wazuh Manager sends the Active Response command to the affected agent.

2\. `firewall-drop` is executed locally on `soc-server`.

3\. The source IP is blocked.

4\. The block remains active for 300 seconds.

5\. Wazuh removes the temporary block after the timeout.



\---



\## Response Verification



The Active Response log on `soc-server` confirmed execution of `firewall-drop`.



The response generated an `add` action for:



```text

192.168.1.34

```



The log also confirmed:



```text

Rule ID: 5712

Level: 10

Program: active-response/bin/firewall-drop

Command: add

```



During the test, the SSH session from the Windows workstation was terminated and subsequent connection attempts resulted in a timeout.



After approximately five minutes, the Active Response log recorded:



```text

Command: delete

```



for the same source IP.



SSH connectivity was then restored, confirming that the temporary block had expired as configured.



\---



\## Evidence



The main evidence collected during the scenario was:



\### Detection



```text

Rule 5710

Level 5

sshd: Attempt to login using a non-existent user

```



\### Correlation



```text

Rule 5712

Level 10

sshd: brute force trying to get access to the system. Non existent user.

```



\### MITRE ATT\&CK



```text

T1110 — Brute Force

```



\### Active Response



```text

firewall-drop

Source IP: 192.168.1.34

Action: add

Timeout: 300 seconds

Action: delete

```



\### Verification



The source IP was temporarily unable to establish an SSH connection during the response window and connectivity was restored after the timeout.



\---



\## Lessons Learned



This scenario demonstrated several important SOC concepts:



\* Linux authentication logs can be collected through `journald`.

\* Wazuh can detect individual SSH authentication failures.

\* Wazuh correlation rules can identify repeated authentication attempts as a brute-force pattern.

\* Rule severity increases when multiple related events are correlated.

\* Wazuh integrates detection with automated response through Active Response.

\* Responses can be executed locally on the affected endpoint.

\* Temporary blocking provides an automated containment mechanism while avoiding a permanent firewall rule.

\* The full SOC workflow can be validated in a controlled lab environment.



\---



\## Scenario Status



\*\*Completed\*\*



The scenario successfully demonstrated:



\*\*Simulation → Detection → Correlation → Investigation → Automated Response → Verification\*\*



\---



\## Next Steps



Potential extensions for this scenario include:



\* Test brute-force attempts against an existing user.

\* Create a custom Wazuh detection rule.

\* Add additional SSH-related detections.

\* Investigate successful authentication after multiple failures.

\* Correlate SSH activity with privilege escalation.

\* Build a dashboard for SSH authentication events.

\* Add screenshots of the detection and Active Response to the project documentation.



