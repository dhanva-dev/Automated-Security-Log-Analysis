# Apache Web Server Log Forensics & Anomaly Detection

## Overview
This module focuses on the manual and automated analysis of Apache Web Server logs (`access.log` and `error.log`). The objective is to identify Indicators of Compromise (IOCs), such as unauthorized access attempts, scanning activity, and denial-of-service precursors.

## Environment & Prerequisites
- **Target System:** Ubuntu Server 22.04 LTS (Apache2 Service)
- **Log Location:** `/var/log/apache2/`
- **Tools:** Bash, AWK, Grep, Python 3.x

## Forensic Analysis Workflow

### Phase 1: Data Acquisition & Structure Review

Before automated parsing, analysts must verify log integrity and structure.

1. **Navigate to Log Directory:**
    ```bash
    cd /var/log/apache2/
    ls -lh
    ```

2. **Log Structure Validation:**
    Use `head` or `less` to inspect the latest entries and verify the Common Log Format (CLF).
    ```bash
    head -n 20 access.log
    ```
    *Key Data Points Identified:* Source IP, Timestamp, HTTP Method (GET/POST), Resource Path, HTTP Status Code, User Agent.

### Phase 2: Threat Hunting & IOC Filtering

This phase involves isolating specific traffic patterns indicative of reconnaissance or attacks.

1. **Targeted IP Investigation:**
    Isolate traffic from suspicious IP addresses identified by the NIDS or firewall.
    ```bash
    grep '192.168.1.100' access.log
    ```

2. **Scanning Activity Detection (404 Floods):**
    Filter for high volumes of `404 Not Found` errors, which often indicate directory enumeration tools (e.g., Dirbuster, Gobuster).
    ```bash
    grep ' 404 ' access.log | head -n 20
    ```

3. **Complex Correlation:**
    Correlate a specific Source IP with error codes to confirm an active scan against that target.
    ```bash
    grep '192.168.1.100' access.log | grep ' 404 '
    ```

### Phase 3: Error Log Auditing

Analyze `error.log` for system-level failures or script exploits.

1. **Review Critical Errors:**
    ```bash
    cat error.log | grep -i "error" | tail -n 50
    ```
    *Focus Areas:* Permission denied errors, PHP script failures, and potential SQL injection warnings logged by the server.

### Phase 4: Statistical Anomaly Detection (Bash Profiling)

Use `awk` for quick statistical baselining of traffic to identify outliers.

1. **Top Talkers (Source IP Volume):**
    Identifies potential DoS sources or heavy scrapers.
    ```bash
    awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -n 10
    ```

2. **Traffic Frequency Analysis:**
    Break down request volume by timestamp/day to spot spikes in traffic.
    ```bash
    awk '{print $4}' access.log | cut -d: -f1 | sort | uniq -c
    ```

3. **Resource Targeting:**
    Identify the most frequently accessed endpoints to detect "login" page brute-forcing.
    ```bash
    awk '{print $7}' access.log | sort | uniq -c | sort -nr | head -n 10
    ```

## Outcome
This workflow provides a rapid triage capability for web server incidents, allowing the security team to quickly isolate malicious IPs and understand attack vectors before deploying full-scale remediation.
