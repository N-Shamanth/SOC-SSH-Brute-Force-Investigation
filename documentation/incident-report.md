# SSH Brute-Force Investigation

## 1. Incident Overview

A controlled SSH authentication attack was simulated against a Kali Linux virtual machine running in a VirtualBox lab environment.

The objective of this project was to generate controlled SSH authentication failures, collect the resulting security events, investigate the activity, identify indicators of compromise, determine the authentication outcome, map the observed behavior to MITRE ATT&CK, and document the findings using a basic SOC investigation workflow.

## 2. Environment

- Operating System: Kali Linux
- Virtualization Platform: VirtualBox
- SSH Server: OpenSSH
- Log Source: systemd journal
- Protocol: SSH
- SSH Destination Port: 22
- Kali Linux IP Address: 10.0.2.15

The SSH service was configured and started on the Kali Linux virtual machine for the purpose of this controlled security lab.

## 3. Detection

The SSH service status was checked using:

    sudo systemctl status ssh

The SSH service was confirmed to be active and running.

The SSH listening port was verified using:

    sudo ss -tlnp | grep :22

The system showed SSH listening on TCP port 22.

SSH authentication events were investigated using the systemd journal:

    sudo journalctl -u ssh --no-pager

Authentication-related events were filtered using:

    sudo journalctl -u ssh --no-pager | grep -Ei "failed password|invalid user"

The investigation identified multiple authentication-related events involving an invalid username and failed passwords.

## 4. Observed Activity

The following SSH authentication activity was observed during the controlled test:

- Target username: fakeuser
- Source IP address: 10.0.2.15
- SSH destination port: 22
- Source ports observed: 45054 and 43812
- Activity occurred between approximately 20:46 and 20:48
- 2 Invalid user events were observed
- 6 Failed password events were observed

The source IP address belongs to the Kali Linux virtual machine because the authentication attempts were intentionally generated locally as part of the lab.

The activity generated multiple SSH authentication failure events in the systemd journal.

## 5. Investigation

SSH authentication failures were filtered from the systemd journal using:

    sudo journalctl -u ssh --no-pager | grep -Ei "failed password|invalid user"

The logs contained events such as:

    Invalid user fakeuser from 10.0.2.15
    Failed password for invalid user fakeuser from 10.0.2.15

These events indicate repeated authentication failures involving an invalid username.

The source IP addresses associated with the authentication failures were extracted using:

    sudo journalctl -u ssh --no-pager | grep -Ei "failed password|invalid user" | grep -oE 'from [0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c

The investigation identified 10.0.2.15 as the source IP associated with the observed authentication failures.

The targeted username was identified using:

    sudo journalctl -u ssh --no-pager | grep "Invalid user" | sed -E 's/.*Invalid user ([^ ]+).*/\1/' | sort | uniq -c

The result identified:

    2 fakeuser

This indicates that fakeuser appeared in two recorded invalid-user events during the investigated activity.

## 6. Authentication Outcome

The SSH logs were checked for successful authentication using:

    sudo journalctl -u ssh --since "30 minutes ago" --no-pager | grep -Ei "Accepted|Failed password|Invalid user"

The investigation showed multiple failed authentication events.

No Accepted authentication event was observed in the investigated activity.

Therefore, the controlled authentication attempts did not demonstrate a successful SSH login.

The activity is classified as an attempted authentication attack rather than a confirmed system compromise.

## 7. Attack Timeline

| Time | Event |
|---|---|
| 20:46:00 | Invalid SSH user fakeuser observed |
| 20:46:41 | Failed password attempt observed |
| 20:46:50 | Failed password attempt observed |
| 20:46:58 | Failed password attempt observed |
| 20:46:59 | Connection closed |
| 20:47:33 | Invalid SSH user fakeuser observed |
| 20:47:41 | Failed password attempt observed |
| 20:47:51 | Failed password attempt observed |
| 20:48:04 | Failed password attempt observed |
| 20:48:05 | Connection closed |

The timestamps above were obtained from the SSH systemd journal during the controlled lab activity.

## 8. Indicators of Compromise

| Indicator | Value |
|---|---|
| Source IP | 10.0.2.15 |
| Target Username | fakeuser |
| Destination Port | 22 |
| Source Ports | 45054, 43812 |
| Protocol | SSH |
| Activity | Authentication failures |
| Time Window | Approximately 20:46–20:48 |

Because this was a controlled lab, 10.0.2.15 represents the Kali Linux virtual machine that generated the authentication attempts.

## 9. MITRE ATT&CK Mapping

### T1110 - Brute Force

The observed activity is mapped to the MITRE ATT&CK technique:

T1110 - Brute Force

The controlled simulation involved repeated authentication attempts against an SSH service using an invalid username and incorrect passwords.

This demonstrates how repeated authentication failures can be identified and investigated by a SOC analyst.

The technique mapping is included to demonstrate the relationship between observed authentication activity and a commonly used threat behavior framework.

## 10. Impact Assessment

No successful SSH authentication was observed during the investigated activity.

There was no evidence of a successful compromise during this controlled simulation.

The activity therefore represents an authentication attack attempt rather than a confirmed system compromise.

The primary security concern demonstrated by this exercise is the ability of repeated authentication attempts to generate detectable security events that should be monitored by a SOC.

## 11. Recommended Defensive Actions

### Authentication Security

- Use strong and unique passwords.
- Prefer SSH key-based authentication where appropriate.
- Disable direct root SSH login where appropriate.
- Disable unnecessary user accounts.
- Remove unused SSH access.
- Use multi-factor authentication where supported.

### Network Security

- Restrict SSH access to trusted networks where possible.
- Use firewall rules to limit access to TCP port 22.
- Avoid exposing SSH directly to the public internet unless required.
- Consider VPN-based access for administrative SSH connections.

### Monitoring and Detection

- Monitor SSH authentication failures.
- Alert on repeated authentication failures from the same source.
- Monitor attempts involving invalid usernames.
- Centralize authentication logs for SOC monitoring.
- Create detection rules for abnormal authentication patterns.
- Correlate authentication failures with successful logins and other system activity.

### Incident Response

- Investigate suspicious source IP addresses.
- Determine whether authentication was successful.
- Identify targeted accounts.
- Review additional system activity following suspicious authentication events.
- Block or restrict malicious sources when appropriate.
- Document the investigation and preserve relevant evidence.

## Conclusion

This project demonstrated a basic Security Operations Center investigation workflow using a Kali Linux virtual machine.

The investigation included:

1. Setting up an SSH service in a VirtualBox lab.
2. Verifying that SSH was running.
3. Verifying that SSH was listening on TCP port 22.
4. Generating controlled SSH authentication failures.
5. Collecting SSH authentication events using systemd journal.
6. Detecting failed authentication activity.
7. Identifying the source IP address.
8. Identifying the targeted username.
9. Checking whether authentication was successful.
10. Extracting indicators of compromise.
11. Creating an attack timeline.
12. Mapping the observed activity to MITRE ATT&CK.
13. Assessing the potential impact.
14. Documenting defensive recommendations.

The investigation identified multiple failed SSH authentication events involving the fakeuser account from the Kali Linux lab IP address 10.0.2.15.

No successful authentication was observed during the investigated activity.

This project demonstrates practical skills in Linux log analysis, SSH monitoring, systemd journal analysis, authentication event analysis, IOC identification, MITRE ATT&CK mapping, basic incident response, and security documentation.
