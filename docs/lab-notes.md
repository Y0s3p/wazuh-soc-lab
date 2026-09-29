# Wazuh SOC Lab — Lab Notes

## 2026-09-29 — Initial Laboratory Setup and Vulnerability Detection

### Completed

* Created the initial Wazuh SOC laboratory environment.
* Deployed a Wazuh 4.14.7 server using the official Wazuh OVA.
* Deployed an Ubuntu 26.04.1 LTS endpoint as `soc-server`.
* Installed and registered the Wazuh Agent on the Ubuntu endpoint.
* Verified communication between the endpoint and Wazuh Manager.
* Enabled and tested File Integrity Monitoring (FIM).
* Enabled Security Configuration Assessment (SCA).
* Verified system and software inventory through Syscollector.
* Enabled Vulnerability Detection.
* Investigated a vulnerability reported by Wazuh.
* Created the initial GitHub repository for the laboratory.

### Environment

| Host           | Operating System     | IP Address     | Role                                 |
| -------------- | -------------------- | -------------- | ------------------------------------ |
| `wazuh-server` | Amazon Linux 2023.12 | `192.168.1.35` | Wazuh Manager, Indexer and Dashboard |
| `soc-server`   | Ubuntu 26.04.1 LTS   | `192.168.1.36` | Wazuh Agent / monitored endpoint     |

Wazuh version:

**4.14.7**

### File Integrity Monitoring

FIM was tested by creating and modifying:

```text
/etc/wazuh-test.txt
```

Wazuh successfully detected the file integrity change and generated an alert associated with the modification.

This confirmed that the endpoint was correctly communicating with the Wazuh Manager and that FIM was operational.

### Vulnerability Detection

Vulnerability Detection was enabled on the Wazuh Manager and the endpoint inventory was evaluated against Wazuh vulnerability intelligence.

One of the vulnerabilities initially reported was:

```text
CVE-2026-27456
```

The vulnerability was investigated by comparing:

* Installed package versions.
* Upstream affected versions.
* Ubuntu security status.
* Wazuh vulnerability intelligence.
* The vulnerability state reported by the Wazuh Dashboard.

The investigation showed that vulnerability status can depend on vendor-specific security information and that an upstream version comparison alone is not always sufficient to determine whether a package is vulnerable.

### Issue: Vulnerability Feed Update

During a vulnerability intelligence update, the Wazuh server ran out of available disk space.

The Wazuh content updater reported errors related to writing the vulnerability feed to disk.

The problem was traced to insufficient storage capacity on the Wazuh virtual machine.

### Resolution

The Wazuh server virtual disk was increased from:

```text
25 GB → 80 GB
```

After rebooting the virtual machine, the operating system detected the full disk capacity.

The vulnerability content updater subsequently performed a snapshot download and processed the updated vulnerability intelligence.

The temporary download data was cleaned up after processing.

### Verification

After the vulnerability feed update completed, the vulnerability state in the Wazuh Dashboard changed.

`CVE-2026-27456`, which had previously appeared among the most prominent vulnerabilities, was no longer present in the current top vulnerability list.

This demonstrated that Wazuh vulnerability results can change when vulnerability intelligence is updated and endpoint data is reevaluated.

### Lessons Learned

* Vulnerability detection depends on both endpoint inventory and vulnerability intelligence.
* Vendor security status may differ from upstream software version information.
* Wazuh vulnerability content updates can require significant temporary disk space.
* Storage capacity should be considered when deploying Wazuh in a virtual laboratory.
* Security findings should be investigated using multiple sources rather than relying on a single version comparison.
* Maintaining a technical investigation log makes troubleshooting and future incident analysis easier.

### Current Status

The Wazuh infrastructure is operational.

The laboratory currently provides:

* Endpoint monitoring.
* File Integrity Monitoring.
* Security Configuration Assessment.
* System inventory.
* Vulnerability Detection.

The initial laboratory documentation has been created and the project repository has been published to GitHub.

### Next Step

Begin structured SOC detection scenarios, starting with:

**SSH authentication monitoring and controlled brute-force simulation.**
