# Advanced Endpoint Forensics with Sysmon & Sysinternals

## Overview
This module details the deployment and utilization of **Sysmon (System Monitor)** for granular endpoint visibility. Unlike standard Windows Event Logs, Sysmon provides detailed telemetry on process creation, network connections, and file modifications, enabling the detection of sophisticated malware techniques including DLL injection and C2 (Command & Control) beaconing.

## Environment & Prerequisites
- **Target System:** Windows 10/11 or Windows Server 2019+
- **Agent:** Sysinternals Sysmon v14.0+
- **Configuration Standard:** SwiftOnSecurity High-Fidelity Config
- **Tools:** PowerShell 5.1+, Windows Event Viewer

## Forensic Analysis Workflow

### Phase 1: Agent Deployment & Hardening

Deploy Sysmon with a filtered configuration to minimize noise and maximize high-fidelity threat data.

1.  **Installation:**
    Execute the following with administrative privileges to install the driver and load the configuration schema.
    ```powershell
    sysmon.exe -accepteula -i sysmonconfig-export.xml
    ```

2.  **Verification:**
    Confirm the service is active and writing to the dedicated operational log.
    * **Log Path:** `Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational`

### Phase 2: Process Hierarchy Analysis (Event ID 1)

Detect "Living off the Land" (LotL) attacks by analyzing parent-child process relationships.

1.  **Objective:**
    Identify suspicious command execution chains (e.g., MS Word spawning PowerShell).

2.  **Hunt Logic:**
    * Filter for **Event ID 1** (Process Create).
    * *Suspicious Pattern:* `winword.exe` (Parent) -> `cmd.exe` (Child).
    * *Suspicious Pattern:* `services.exe` (Parent) -> `powershell.exe` (Child).

### Phase 3: Network Beaconing Detection (Event ID 3)

Identify Command & Control (C2) channels by auditing outbound network connections initiated by non-browser processes.

1.  **Objective:**
    Detect malware "phoning home" to an attacker's server.

2.  **Hunt Logic:**
    * Filter for **Event ID 3** (Network Connection).
    * *Suspicious Pattern:* `notepad.exe` or `svchost.exe` initiating a connection to a public IP on port 443 or 8080.
    * *Action:* Cross-reference Destination IPs with Threat Intelligence feeds (e.g., VirusTotal).

### Phase 4: Malware Persistence Auditing (Event ID 11)

Track the creation of malicious files in startup folders or temporary directories.

1.  **Objective:**
    Detect "Dropper" malware writing executables to disk.

2.  **Hunt Logic:**
    * Filter for **Event ID 11** (File Create).
    * *Target Directories:* `C:\Users\Public\`, `C:\Windows\Temp\`, `AppData\Roaming`.
    * *Extensions:* Monitor for `.exe`, `.vbs`, `.ps1` or `.bat` files created by browser processes.

### Phase 5: Memory Forensics & DLL Injection (Event ID 7)

Detect advanced evasion techniques where malware hides inside legitimate processes.

1.  **Objective:**
    Identify "Reflective DLL Injection" attempts.

2.  **Hunt Logic:**
    * Filter for **Event ID 7** (Image Loaded).
    * *Suspicious Pattern:* Unsigned DLLs loaded by critical system processes (e.g., `lsass.exe` or `spoolsv.exe`).

## Outcome
By mastering Sysmon telemetry, the analyst can reconstruct the full "Kill Chain" of an attack, proving exactly how an attacker entered, moved, and persisted within the network.
