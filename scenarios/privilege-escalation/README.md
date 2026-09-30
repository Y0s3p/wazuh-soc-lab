# Privilege Escalation / Account Manipulation Scenario

## Objective

Simulate account creation and privilege manipulation on an Ubuntu endpoint and verify that Wazuh can detect, correlate, and provide evidence of potentially suspicious account and privilege changes.

The scenario demonstrates the following workflow:

**Simulation → Detection → Correlation → Investigation → Verification**

---

## Environment

| Component                      | Details            |
| ------------------------------ | ------------------ |
| Wazuh Manager                  | `wazuh-server`     |
| Wazuh version                  | 4.14.7             |
| Endpoint                       | `soc-server`       |
| Endpoint IP                    | `192.168.1.36`     |
| Operating System               | Ubuntu 26.04.1 LTS |
| Agent ID                       | `001`              |
| User performing the simulation | `jose`             |

---

## Scenario

A temporary local account is created and added to the `sudo` group.

The activity is intentionally performed by the administrator to simulate behavior that could indicate account manipulation or privilege escalation in a real environment.

The test account used during the simulation was:

```text
wazuh-test-user
```

The account was removed after testing.

---

## Attack / Event Simulation

### 1. Create a local account

```bash
sudo useradd wazuh-test-user
```

Wazuh detected the account creation through the system logs.

Relevant rules:

* **5901 — New group added to the system**
* **5902 — New user added to the system**

The event contained information such as:

* Username
* UID
* GID
* Home directory
* Shell
* Source terminal

---

### 2. Add the account to the sudo group

```bash
sudo usermod -aG sudo wazuh-test-user
```

The underlying system log contained:

```text
usermod: add 'wazuh-test-user' to group 'sudo'
```

Wazuh also detected the command executed through `sudo`.

The initial detection was:

* **Rule 5402 — Successful sudo to ROOT executed**
* Level: 3

The event included the exact command:

```text
/usr/sbin/usermod -aG sudo wazuh-test-user
```

---

## Custom Detection Rule 100101

A custom Wazuh rule was created to specifically identify a user being added to the `sudo` group.

```xml
<rule id="100101" level="10">
  <if_sid>5402</if_sid>
  <match>usermod -aG sudo</match>
  <description>Local: User added to sudo group via usermod.</description>
  <mitre>
    <id>T1098</id>
  </mitre>
  <group>account_manipulation,privilege_escalation,</group>
</rule>
```

This raises the severity from the generic sudo detection to **Level 10**.

The Wazuh Dashboard displayed:

* Rule ID: `100101`
* Level: `10`
* MITRE technique: `T1098`
* MITRE technique name: `Account Manipulation`
* MITRE tactic: `Persistence`
* Command: `/usr/sbin/usermod -aG sudo wazuh-test-user`
* Source user: `jose`
* Destination user: `root`

---

## Correlation Rule 100102

A second custom rule was created to detect repeated occurrences of the custom `100101` detection.

```xml
<rule id="100102" level="12" frequency="2" timeframe="120">
  <if_matched_sid>100101</if_matched_sid>
  <description>Possible repeated account privilege escalation: user added to sudo group.</description>
  <mitre>
    <id>T1098</id>
  </mitre>
  <group>account_manipulation,privilege_escalation,</group>
</rule>
```

The rule requires:

* **2 occurrences** of rule `100101`
* Within a **120-second timeframe**
* The resulting alert is raised to **Level 12**

This was verified by executing the following command twice in quick succession:

```bash
sudo usermod -aG sudo wazuh-test-user
sudo usermod -aG sudo wazuh-test-user
```

Wazuh generated the expected correlation alert.

---

## Detection and Correlation

The resulting detection chain was:

```text
sudo usermod -aG sudo
        │
        ▼
Rule 5402
Successful sudo to ROOT
Level 3
        │
        ▼
Rule 100101
User added to sudo group
Level 10
        │
        ▼
2 matching events within 120 seconds
        │
        ▼
Rule 100102
Possible repeated account privilege escalation
Level 12
```

The Wazuh Dashboard confirmed the `100102` event with:

* Rule ID: `100102`
* Level: `12`
* Frequency: `2`
* MITRE technique: `T1098 — Account Manipulation`
* MITRE tactic: `Persistence`
* Group: `account_manipulation, privilege_escalation`
* Command:
  `/usr/sbin/usermod -aG sudo wazuh-test-user`

---

## Additional Evidence

The account lifecycle generated several independent indicators that can be correlated during an investigation.

### Account creation

* Rule `5901` — New group added
* Rule `5902` — New user added

### Privileged execution

* Rule `5402` — Successful sudo to ROOT

### Custom detection

* Rule `100101` — User added to sudo group

### Correlation

* Rule `100102` — Repeated account privilege escalation detection

### Account removal

* Rule `5903` — Group or user deleted from the system

### File Integrity Monitoring

Changes to:

```text
/etc/gshadow
```

were detected by Wazuh FIM using:

* Rule `550`
* Integrity checksum changed

This provides an additional indicator that account/group configuration was modified.

---

## Investigation

The Wazuh Dashboard was used to investigate the generated alerts rather than relying exclusively on command-line log analysis.

Relevant fields observed during investigation included:

```text
agent.name
agent.ip
data.command
data.srcuser
data.dstuser
data.tty
data.pwd
rule.id
rule.level
rule.description
rule.mitre.id
rule.mitre.tactic
rule.mitre.technique
```

The investigation established that:

1. The activity originated from `soc-server`.
2. The command was executed by the `jose` account.
3. `sudo` executed the command with `root` privileges.
4. The command added `wazuh-test-user` to the `sudo` group.
5. Wazuh generated the custom Level 10 detection.
6. Repeated executions triggered the Level 12 correlation rule.
7. The activity was intentionally generated as part of this controlled test.

---

## Response

No automated response was configured for this scenario.

This is intentional because adding a user to the `sudo` group may be legitimate administrative activity.

A real SOC investigation should first establish:

1. Whether the account creation was authorized.
2. Who performed the action.
3. Whether the privilege assignment was expected.
4. Whether the account should remain in the `sudo` group.
5. Whether additional persistence or suspicious activity occurred.

If the activity is confirmed to be unauthorized, appropriate response actions could include removing the account or privilege assignment, preserving relevant evidence, and investigating the originating session and additional activity.

---

## Verification

After completing the simulation, the temporary account was removed:

```bash
sudo userdel wazuh-test-user
```

The endpoint was therefore returned to its original account state.

The deletion itself generated additional Wazuh telemetry, including rule `5903` and FIM changes to `/etc/gshadow`.

---

## MITRE ATT&CK

The custom rules were mapped to:

* **T1098 — Account Manipulation**
* Tactic: **Persistence**

The scenario demonstrates how account and group modifications can provide security telemetry relevant to account manipulation.

---

## Lessons Learned

### 1. Generic detections can be made more useful

Rule `5402` detects successful sudo execution but is relatively generic.

The custom `100101` rule provides a more specific detection for adding an account to the `sudo` group.

### 2. Correlation increases detection severity

Rule `100102` demonstrates how repeated occurrences of a lower-level custom detection can generate a higher-severity alert.

### 3. Multiple independent indicators improve investigation

The same activity generated telemetry from:

* `useradd`
* `usermod`
* `sudo`
* PAM
* FIM
* Wazuh custom rules
* Wazuh correlation

This allows an analyst to reconstruct the account lifecycle rather than relying on a single alert.

### 4. The Dashboard is useful for investigation

The Wazuh Dashboard provided direct visibility into:

* Alert severity
* Commands
* Users
* Agents
* MITRE mappings
* Rule information
* Correlated detections

This scenario was therefore investigated using both the underlying system logs and the Wazuh Dashboard.

---

## Evidence

Recommended screenshots for the project:

1. Rule `100101` — Level 10 custom detection.
2. Rule `100102` — Level 12 correlation detection.
3. Threat Hunting view showing the sequence of `100101` and `100102`.
4. Rule details showing the MITRE `T1098` mapping.

---

## Scenario Status

**Completed**

The scenario successfully demonstrated:

* Account creation detection
* Privileged command detection
* Custom Wazuh rule creation
* Account manipulation detection
* Event correlation
* Severity escalation
* MITRE ATT&CK mapping
* Dashboard-based investigation
* Post-test cleanup

---

## Next Steps

Potential future scenarios:

* Suspicious process execution
* Persistence mechanisms
* File integrity and suspicious modification
* SSH privilege escalation
* Malware-like behavior
* Windows endpoint monitoring with Sysmon
* Custom detection and automated response
