# Linux Syslog Analysis & Authentication Auditing

## Overview
This module outlines the procedures for auditing Linux system logs (`syslog` and `auth.log`) to detect unauthorized access, privilege escalation, and system anomalies. The workflow leverages standard CLI tools to parse logging facilities and identify security events.

## Environment & Prerequisites
- **Target System:** Ubuntu 20.04+ / RHEL / CentOS
- **Log Location:** `/var/log/`
- **Core Facilities:** `syslog` (General System), `auth.log` (Authentication)
- **Tools:** `rsyslog`, `grep`, `awk`, `nano`

## Audit Workflow

### Phase 1: Configuration & Facility Review

Verify the logging daemon configuration to understand where critical security events are being routed.

1. **Inspect Rsyslog Config:**
    ```bash
    sudo nano /etc/rsyslog.conf
    ```
    *Objective:* Confirm that `auth` and `kern` (kernel) facilities are logging to standard destinations and verify if remote log forwarding is active.

### Phase 2: Log Access & Integrity Check

Access the central repository of system logs to assess volume and integrity.

1. **Navigate to Log Repository:**
    ```bash
    cd /var/log/
    ls -lh
    ```
    *Objective:* Check file sizes. An unexpectedly large `auth.log` or `syslog` often indicates a brute-force attack or a runaway process.

2. **Real-time Monitoring:**
    Use `tail` to monitor live system events during an active investigation.
    ```bash
    tail -f syslog
    ```

### Phase 3: Targeted Event Filtering

Isolate specific events based on timestamps or service identifiers to correlate with reported incidents.

1. **Timestamp Correlation:**
    Extract logs from a specific incident window (e.g., June 12th).
    ```bash
    grep 'Jun 12' syslog | head -n 20
    ```

2. **Service-Specific Auditing (SSH Daemon):**
    Filter for `sshd` interactions to isolate remote access activity.
    ```bash
    grep 'sshd' syslog | tail -n 20
    ```

3. **Complex Filtering:**
    Combine timestamp and service filters to reconstruct the timeline of a specific login session.
    ```bash
    grep 'Jun 12' syslog | grep 'sshd'
    ```

### Phase 4: Authentication Security Audit

Analyze `auth.log` (or `secure` on RHEL) to detect successful and failed entry attempts.

1. **Review Authentication Events:**
    ```bash
    less auth.log
    ```
    *Key Indicators:* Look for "Accepted password", "Failed password", "Invalid user", and `sudo` command execution.

### Phase 5: Threat Detection via Statistical Analysis

Utilize `awk` to automate the detection of brute-force patterns and anomalous user behavior.

1. **Brute Force Detection (Failed Logins):**
    Extract and count Source IPs responsible for failed password attempts. High counts indicate an active brute-force attack.
    ```bash
    awk '/sshd/ && /Failed password/ {print $11}' auth.log | sort | uniq -c | sort -nr
    ```

2. **User Access Profiling:**
    Audit successful logins to detect unauthorized account usage.
    ```bash
    awk '/sshd/ && /Accepted password/ {print $9}' auth.log | sort | uniq -c | sort -nr
    ```

3. **Message Frequency Analysis:**
    Identify system noise or spamming processes by counting message types.
    ```bash
    awk '{print $6}' syslog | sort | uniq -c | sort -nr | head -n 10
    ```

## Outcome
Completion of this workflow provides a verified audit trail of user access and system events, enabling the isolation of compromised accounts and the identification of attack sources.
