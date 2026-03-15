# 08 — Endpoint Security & Analysis

Endpoints are a primary target and a primary source of evidence. This section covers endpoint protection concepts, detection and response platforms, host-based monitoring, and tools for investigating endpoint activity at scale.

## Tools in this section

- [AV / EPP / EDR / XDR — Concepts](#av--epp--edr--xdr--concepts)
- [osquery](#osquery)
- [Sysinternals Suite](#sysinternals-suite)
- [Sysmon](#sysmon)
- [Velociraptor](#velociraptor)
- [Wazuh](#wazuh)

---

## AV / EPP / EDR / XDR — Concepts

- **Description:** These terms describe an evolution in endpoint security capabilities, each generation adding more detection and response depth.

  | Term | Focus |
  |------|-------|
  | **AV (Antivirus)** | Detect known malware via signatures and basic heuristics |
  | **EPP (Endpoint Protection Platform)** | Prevention-focused: ML, behavior analysis, exploit prevention, device control |
  | **EDR (Endpoint Detection & Response)** | Detection and response: deep endpoint telemetry, threat hunting, isolation, remediation |
  | **XDR (Extended Detection & Response)** | Correlates telemetry across endpoints, network, cloud, email, and identity for unified attack chain visibility |

- **Blue Team use:**
  - Prevent and detect threats at the endpoint before lateral movement occurs
  - Triage alerts, investigate endpoint activity, and hunt for threats using the vendor's query interface
  - Isolate compromised hosts and run response actions from the management console
  - Integrate endpoint telemetry with SIEM for broader correlation
  - Tune detection policies to reduce false positives without creating blind spots

- **Common commercial solutions:**
  - Microsoft Defender for Endpoint (MDE)
  - CrowdStrike Falcon
  - SentinelOne Singularity
  - Palo Alto Cortex XDR
  - Carbon Black (Broadcom)
  - Sophos Intercept X
  - Trend Micro Vision One

- **Type:** Endpoint security platforms (agent-based, cloud-managed)
- **Platform:** Windows, macOS, Linux (agent deployment via management console)

- **Notes:** Understanding your specific EDR/XDR platform's query language and telemetry model is as important as knowing the tool exists. Most EDRs expose a threat hunting interface using SQL-like or KQL-style queries — invest time learning it. Key capabilities to understand in your platform: process tree visualization, parent-child relationship analysis, network connection attribution, and live response / remote shell. Open-source alternatives like Wazuh provide some EDR-like capabilities but require more manual integration work.

---

## osquery

- **Description:** Open-source framework that exposes the OS as a relational database, queryable with SQL. Provides deep, cross-platform endpoint visibility for threat hunting, compliance auditing, and IR.
- **Blue Team use:** Query running processes, network connections, loaded modules, file hashes, user sessions, persistence mechanisms, and hardware state using standard SQL — on a single host or across a managed fleet.
- **Website:** [osquery.io](https://osquery.io/)
- **Type:** Endpoint instrumentation and query engine
- **Platform:** Linux, Windows, macOS, FreeBSD

> Full documentation, usage examples, and SQL queries are in [07 — Miscellaneous Defensive Tools](./07-miscellaneous-defensive-tools.md#osquery).

**Fleet management:** In production, osquery runs as a daemon (`osqueryd`) with results forwarded to a central log system or fleet manager. Consider [Fleet](https://fleetdm.com/) (open source) or [Kolide](https://www.kolide.com/) for managing osquery at scale.

---

## Sysinternals Suite

- **Description:** Microsoft's collection of free Windows analysis utilities. Essential for manual endpoint investigation and malware analysis on Windows systems.
- **Blue Team use:** Process investigation, real-time activity monitoring, persistence analysis, network connection attribution, string extraction from binaries.
- **Website:** [Microsoft Sysinternals](https://docs.microsoft.com/en-us/sysinternals/)
- **Type:** Windows analysis utilities (GUI + CLI)
- **Platform:** Windows

> Full tool list, usage examples, and Sysmon config references are in [07 — Miscellaneous Defensive Tools](./07-miscellaneous-defensive-tools.md#sysinternals-suite).

---

## Sysmon

- **Description:** Windows system service and device driver that logs detailed endpoint activity to the Windows Event Log. Provides telemetry far beyond native Windows logging — used as a foundational data source for SIEM-based detection and threat hunting.
- **Blue Team use:**
  - Log detailed process creation events (command line, hashes, parent process)
  - Record network connections initiated by processes (destination IP, port, process)
  - Track file creation and modification events
  - Monitor registry changes and driver/image loading
  - Forward logs to SIEM for detection rules and threat hunting queries
- **Website:** [Microsoft Sysinternals — Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
- **Type:** Windows monitoring service / event log data source
- **Platform:** Windows
- **Installation:**
  ```bash
  # Install with a configuration file
  sysmon64.exe -accepteula -i sysmon_config.xml

  # Update configuration without reinstalling
  sysmon64.exe -c new_config.xml

  # Check current configuration
  sysmon64.exe -s

  # Uninstall
  sysmon64.exe -u
  ```

**Key Event IDs:**

| ID | Event |
|----|-------|
| 1 | Process creation (includes command line, hashes, parent) |
| 2 | Process changed a file creation time (timestomping) |
| 3 | Network connection |
| 5 | Process terminated |
| 6 | Driver loaded |
| 7 | Image/DLL loaded |
| 8 | CreateRemoteThread (injection indicator) |
| 10 | ProcessAccess (LSASS access indicator) |
| 11 | File created |
| 12/13/14 | Registry object created/modified/deleted |
| 15 | File stream created (ADS) |
| 17/18 | Pipe created/connected |
| 22 | DNS query |
| 23 | File deleted |
| 25 | Process tampering |

- **Alternatives:** Native Windows logging (less detail), commercial EDR telemetry (more integrated but proprietary), auditd (Linux)
- **Notes:** Configuration quality determines usefulness. Start with [SwiftOnSecurity's config](https://github.com/SwiftOnSecurity/sysmon-config) or [olafhartong's modular config](https://github.com/olafhartong/sysmon-modular). Both are well-maintained and balance coverage against noise. Forwarding to a SIEM is essential — Sysmon logs sitting in the local Event Log are only useful for live response, not proactive detection. Event ID 1 (process creation with command line) and ID 3 (network connection) alone provide substantial detection coverage.

---

## Velociraptor

- **Description:** Open-source endpoint visibility and response platform using VQL (Velociraptor Query Language). Operates with a client-server architecture — agents deployed on endpoints, central server for management, queries, and collection. Built for scale: can run queries and collect artifacts across thousands of endpoints simultaneously.
- **Blue Team use:**
  - Query endpoint state across a fleet (processes, files, registry, network, OS artifacts)
  - Collect forensic artifacts remotely without physically accessing endpoints
  - Hunt for IOCs or suspicious patterns across all managed endpoints simultaneously
  - Monitor endpoints for specific events in real time
  - Run automated response actions based on detection logic
  - Replace or supplement KAPE for remote artifact collection at scale
- **Website:** [docs.velociraptor.app](https://docs.velociraptor.app/) / [GitHub](https://github.com/Velocidex/velociraptor)
- **Type:** Endpoint visibility and response platform (client/server, web UI, VQL)
- **Platform:** Agents for Windows, Linux, macOS. Server primarily on Linux (Windows/macOS also supported).
- **Installation:** Download server and client binaries from [GitHub releases](https://github.com/Velocidex/velociraptor/releases). Docker deployment available. Follow the [deployment guide](https://docs.velociraptor.app/docs/deployment/).
- **Usage (VQL examples via web UI shell or hunt):**
  ```sql
  -- List running processes with hashes
  SELECT Pid, Name, Exe, CommandLine, Hash.SHA256
  FROM pslist()

  -- Find processes making network connections
  SELECT Pid, Name, Laddr, Raddr, Status
  FROM netstat()

  -- Hunt for a specific file hash across the fleet
  SELECT FullPath, Size, Mtime
  FROM glob(globs="C:/Users/**")
  WHERE hash(path=FullPath).SHA256 = "YOUR_HASH_HERE"

  -- Collect Windows event logs from a specific channel
  SELECT EventTime, EventID, Message
  FROM parse_evtx(filename="C:/Windows/System32/winevt/Logs/Security.evtx")
  WHERE EventID = 4625
  LIMIT 100

  -- Run a built-in artifact (e.g., Autoruns equivalent)
  SELECT * FROM Artifact.Windows.Sysinternals.Autoruns()
  ```
- **Alternatives:** GRR Rapid Response (Google, similar architecture), KAPE (local/single host collection), osquery + Fleet (query-focused, less forensic collection), commercial EDR remote response
- **Notes:** VQL is the core skill for getting value from Velociraptor — the [VQL reference](https://docs.velociraptor.app/vql_reference/) is worth studying. The built-in artifact library covers most common collection and hunting needs without writing custom VQL. For remote IR at scale, Velociraptor is one of the most capable open-source options available. Combine with Wazuh or a SIEM for continuous monitoring, and use Velociraptor for targeted deep-dive investigations.

---

## Wazuh

- **Description:** Open-source unified security platform covering SIEM, XDR, and HIDS capabilities. Agents deployed on endpoints collect logs, file integrity data, vulnerability information, and configuration state. Central server analyzes and alerts. Web UI (OpenSearch-based dashboard) for investigation and management.
- **Blue Team use:**
  - Centralize log collection and analysis from endpoints and network devices
  - File Integrity Monitoring (FIM) — detect changes to critical system files
  - Vulnerability detection on endpoints via CVE database integration
  - Security Configuration Assessment (SCA) against CIS benchmarks
  - Host-based intrusion detection (rule-based, signature + behavior)
  - Active response — automatically trigger actions on agents based on alerts (e.g., block an IP, kill a process)
  - Compliance reporting (PCI-DSS, HIPAA, GDPR, NIST)
- **Website:** [wazuh.com](https://wazuh.com/) / [GitHub](https://github.com/wazuh/wazuh)
- **Type:** Open-source security platform (XDR/SIEM/HIDS) — agent/server architecture with web UI
- **Platform:** Agents for Windows, Linux, macOS, Solaris, AIX. Server components on Linux.
- **Installation:** Multiple options — all-in-one, distributed, Docker, cloud images. Follow the [official installation guide](https://documentation.wazuh.com/current/installation-guide/index.html). Typical stack: Wazuh Manager + Wazuh Indexer (OpenSearch) + Wazuh Dashboard.
- **Usage:** Manage agents and review alerts via the Wazuh Dashboard. Key sections:
  - **Security Events** — alert stream with severity, rule ID, and agent
  - **Integrity Monitoring** — file change events
  - **Vulnerabilities** — CVE findings per agent
  - **SCA** — configuration compliance results
  - **MITRE ATT&CK** — alerts mapped to tactics and techniques

  ```bash
  # Check agent status on the manager
  /var/ossec/bin/agent_control -l

  # Restart the Wazuh manager
  systemctl restart wazuh-manager

  # Test a rule against a log line
  /var/ossec/bin/ossec-logtest

  # Check active response configuration
  cat /var/ossec/etc/ossec.conf | grep -A5 "active-response"
  ```

- **Alternatives:** Elastic Security (Elastic Stack-native), Security Onion (includes Wazuh and other tools), Splunk + Security Essentials, commercial SIEM/XDR platforms
- **Notes:** Wazuh requires tuning to be useful — default rules generate significant noise. Start by suppressing false positives on known-good processes and configurations before building detection logic. FIM and SCA are immediately useful with minimal tuning. Wazuh can ingest Sysmon logs from Windows agents, combining host-based telemetry from both sources in one platform.

---
