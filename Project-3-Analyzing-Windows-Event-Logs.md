# Windows Security Event Log Auditing & Threat Hunting

## Overview
This module details the methodology for forensic analysis of Windows Event Logs to detect account compromise, lateral movement, and persistence mechanisms. The workflow utilizes the native Event Viewer (`eventvwr.msc`) and **Log Parser Studio** to query high-volume log datasets for Indicators of Compromise (IOCs).

## Environment & Prerequisites
- **Target System:** Windows 10/11 or Windows Server 2016+
- **Log Source:** Security, System, and Application Logs (`.evtx`)
- **Tools:** Windows Event Viewer, Log Parser Studio (LPS)

## Forensic Analysis Workflow

### Phase 1: Security Log Initialization

Establish a baseline by accessing the Security log channel, which records auditing events.

1. **Access Control Console:**
    * Run `eventvwr.msc` via `Win + R`.
    * Navigate to **Windows Logs** > **Security**.
    * *Objective:* Assess log retention policy (ensure logs are not being overwritten too quickly).

### Phase 2: IOC Identification & Filtering

Filter the log stream to isolate critical security events associated with the "Authentication" and "Account Management" audit subcategories.

1. **Apply Critical Event Filters:**
    Use "Filter Current Log" to isolate the following high-priority Event IDs:
    * **4624:** Successful Logon (Check Logon Type: 3 for Network, 10 for RDP).
    * **4625:** Failed Logon (Brute Force Indicator).
    * **4720:** User Account Created (Persistence Indicator).
    * **4726:** User Account Deleted.

2. **Targeted Search:**
    Search for specific compromised usernames or suspicious Source IPs in the "Find" dialogue.

### Phase 3: Event Correlation & Deep Dive

Analyze specific event payloads to reconstruct the attack timeline.

1. **Payload Analysis (Event 4625):**
    * **Failure Reason:** Analyze "Sub Status" codes (e.g., `0xC000006A` = Bad Password).
    * **Source Network Address:** Identify the IP initiating the connection.
    * **Workstation Name:** Identify the machine name of the attacker (if on LAN).

2. **Attack Pattern Recognition:**
    * *Scenario:* A high volume of **4625** events followed immediately by a single **4624** event indicates a successful brute-force attack.

### Phase 4: Advanced SQL-Based Analysis (Log Parser Studio)

Utilize **Log Parser Studio** to run SQL-like queries against massive log files for statistical analysis.

1. **Setup:**
    Import `.evtx` files into Log Parser Studio.

2. **Top Offender Query (Brute Force Source):**
    Identify the Source IPs generating the most failed login attempts.
    ```sql
    /* Top 10 IP Addresses for Failed Logons */
    SELECT TOP 10 
        EXTRACT_TOKEN(Strings, 19, '|') AS SourceIP, 
        COUNT(*) AS TotalFailures
    FROM '[LOGFILEPATH]'
    WHERE EventID = 4625
    GROUP BY SourceIP
    ORDER BY TotalFailures DESC
    ```

3. **Account Target Analysis:**
    Identify which usernames are being targeted most frequently.
    ```sql
    /* Top Target Accounts */
    SELECT TOP 10 
        EXTRACT_TOKEN(Strings, 5, '|') AS TargetUser, 
        COUNT(*) AS Attempts
    FROM '[LOGFILEPATH]'
    WHERE EventID = 4625
    GROUP BY TargetUser
