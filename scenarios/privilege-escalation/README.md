\# Privilege Escalation / Account Manipulation



\## Objective



Simulate a controlled account manipulation scenario on a Linux endpoint and investigate how Wazuh detects the creation of a new user, privilege assignment, and subsequent account modification.



The objective is to demonstrate how multiple Wazuh detection mechanisms can provide complementary evidence of potentially suspicious privilege escalation activity.



The scenario follows the SOC workflow:



\*\*Simulate → Detect → Investigate → Respond → Verify\*\*



\---



\## Environment



| Component                | Details            |

| ------------------------ | ------------------ |

| Wazuh Manager            | `wazuh-server`     |

| Wazuh Endpoint           | `soc-server`       |

| Endpoint OS              | Ubuntu 26.04.1 LTS |

| Wazuh Version            | 4.14.7             |

| Endpoint IP              | `192.168.1.36`     |

| Manager IP               | `192.168.1.35`     |

| User used for simulation | `jose`             |



The endpoint collects system events through `journald`, which allows Wazuh to monitor authentication and account-management activity.



\---



\## Scenario



A privileged user creates a new local account and subsequently adds that account to the `sudo` group.



From a defensive perspective, this sequence can be relevant because unauthorized account creation followed by privilege assignment can provide an attacker with persistent or elevated access.



The activity was intentionally generated in a controlled lab environment.



\---



\## Attack / Event Simulation



\### 1. Create a new user



On `soc-server`:



```bash

sudo useradd wazuh-test-user

```



This generated the following system events:



```text

new group: name=wazuh-test-user, GID=1001

new user: name=wazuh-test-user, UID=1001, GID=1001,

home=/home/wazuh-test-user, shell=/bin/sh

```



Wazuh detected the account creation with:



```text

Rule: 5902

Level: 8

New user added to the system.

```



A corresponding group creation was also detected:



```text

Rule: 5901

Level: 8

New group added to the system.

```



\---



\### 2. Add the user to the sudo group



The test account was then granted administrative privileges:



```bash

sudo usermod -aG sudo wazuh-test-user

```



The underlying system generated:



```text

add 'wazuh-test-user' to group 'sudo'

add 'wazuh-test-user' to shadow group 'sudo'

```



Wazuh also detected the `sudo` command through its standard rule:



```text

Rule: 5402

Level: 3

Successful sudo to ROOT executed.

```



\---



\## Custom Detection Rule



A custom Wazuh rule was created to specifically identify the addition of a user to the `sudo` group.



File:



```text

/var/ossec/etc/rules/local\_rules.xml

```



Rule:



```xml

<rule id="100101" level="10">

&#x20; <if\_sid>5402</if\_sid>

&#x20; <match>usermod -aG sudo</match>

&#x20; <description>Local: User added to sudo group via usermod.</description>

&#x20; <mitre>

&#x20;   <id>T1098</id>

&#x20; </mitre>

&#x20; <group>account\_manipulation,privilege\_escalation,</group>

</rule>

```



The rule was validated with:



```bash

sudo /var/ossec/bin/wazuh-analysisd -t

```



and the Wazuh Manager was restarted successfully.



\---



\## Detection



The custom rule generated:



```text

Rule: 100101

Level: 10

Local: User added to sudo group via usermod.

```



The alert included the exact command:



```text

command: /usr/sbin/usermod -aG sudo wazuh-test-user

```



This provides useful investigative context because the alert identifies both the privileged operation and the command responsible for it.



\---



\## Additional Evidence



The scenario generated several independent indicators that can be investigated together.



\### Account creation



```text

Rule 5901 — New group added to the system.

Rule 5902 — New user added to the system.

```



\### Privileged command execution



```text

Rule 5402 — Successful sudo to ROOT executed.

```



\### Privilege assignment



```text

Rule 100101 — Local: User added to sudo group via usermod.

```



\### File Integrity Monitoring



Wazuh FIM detected modifications to:



```text

/etc/gshadow

```



The changes occurred as a consequence of the account and group modifications.



\### Account deletion



After the test was completed, the account was removed:



```bash

sudo userdel -r wazuh-test-user

```



Wazuh detected this with:



```text

Rule: 5903

Level: 3

Group (or user) deleted from the system.

```



\---



\## Investigation



The events provide a timeline that can be reconstructed from the Wazuh alerts:



```text

User creation

&#x20;   ↓

Rule 5901 / 5902

&#x20;   ↓

sudo execution

&#x20;   ↓

Rule 5402

&#x20;   ↓

User added to sudo group

&#x20;   ↓

Rule 100101

&#x20;   ↓

/etc/gshadow modified

&#x20;   ↓

FIM Rule 550

&#x20;   ↓

Test account removed

&#x20;   ↓

Rule 5903

```



This demonstrates how several independent telemetry sources can contribute to the investigation of a potential account-manipulation event.



The activity itself was authorized and intentionally generated as part of the lab exercise.



\---



\## Response



No destructive automated response was configured for this scenario.



The appropriate response in a real environment would depend on the investigation and could include:



1\. Validate whether the account creation was authorized.

2\. Identify the user or process responsible for the privileged operation.

3\. Review the account's group memberships.

4\. Review authentication activity associated with the account.

5\. Disable or remove an unauthorized account if confirmed.

6\. Investigate additional persistence or privilege-escalation activity.

7\. Preserve relevant logs and evidence.



The lab intentionally focuses on \*\*detection and investigation\*\* rather than automatically deleting or disabling accounts.



\---



\## Verification



After the simulation, the test account was removed:



```bash

sudo userdel -r wazuh-test-user

```



The Wazuh alerts confirmed the account lifecycle:



```text

5902 → Account created

100101 → Account added to sudo

550 → /etc/gshadow modified

5903 → Account deleted

```



This verified that Wazuh successfully monitored the relevant account-management activity throughout the scenario.



\---



\## MITRE ATT\&CK



The custom detection is mapped to:



\* \*\*T1098 — Account Manipulation\*\*



The scenario demonstrates how modifying account privileges can be monitored as potential account manipulation activity.



\---



\## Lessons Learned



\* Wazuh can detect Linux account creation and deletion using built-in rules.

\* `sudo` activity provides useful command-level evidence for investigations.

\* FIM adds an independent source of evidence by detecting changes to sensitive files such as `/etc/gshadow`.

\* Custom Wazuh rules can increase detection specificity and severity for security-relevant commands.

\* Combining multiple alerts provides better investigative context than relying on a single event.

\* Not every detected privilege modification is malicious; authorization and context must be established during investigation.



\---



\## Scenario Status



\*\*Completed\*\*



\### Detection capabilities demonstrated



\* \[x] New group detection

\* \[x] New user detection

\* \[x] Privileged command detection

\* \[x] Custom privilege-assignment detection

\* \[x] File Integrity Monitoring

\* \[x] Account deletion detection

\* \[x] MITRE ATT\&CK mapping

\* \[x] Investigation timeline

\* \[x] Controlled verification



\---



\## Next Steps



Potential future improvements include:



\* Detecting suspicious modifications to other privileged groups.

\* Monitoring changes to `/etc/sudoers` and `/etc/sudoers.d/`.

\* Creating additional custom rules for privilege escalation techniques.

\* Correlating account creation with subsequent authentication activity.

\* Expanding the scenario to include persistence mechanisms.

\* Adding Windows account and privilege-manipulation scenarios.



