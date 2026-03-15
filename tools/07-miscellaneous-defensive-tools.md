# 07 — Miscellaneous Defensive Tools

Tools and standards that support Blue Team operations but don't fit cleanly into the previous categories: system hardening, endpoint visibility, software supply chain security, container scanning, and CTI data formats.

## Tools in this section

- [Lynis](#lynis)
- [osquery](#osquery)
- [Snyk](#snyk)
- [STIX / TAXII](#stix--taxii)
- [Sysinternals Suite](#sysinternals-suite)
- [Trivy](#trivy)

---

## Lynis

- **Description:** Security auditing and hardening tool for Unix-like systems. Runs locally on a host, checks system configuration, missing patches, insecure settings, and produces a prioritized list of recommendations with references.
- **Blue Team use:**
  - Periodic security health checks on Linux, macOS, and BSD systems
  - Identify hardening gaps before they become vulnerabilities
  - Assist with compliance checks (PCI-DSS, ISO 27001, CIS Benchmarks)
  - Audit new systems before production deployment
- **Website:** [cisofy.com/lynis](https://cisofy.com/lynis/) / [GitHub](https://github.com/CISOfy/lynis)
- **Type:** CLI security auditing and hardening tool
- **Platform:** Linux, macOS, BSD, AIX, Solaris
- **Installation:**
  ```bash
  # Clone for the latest version (recommended)
  git clone https://github.com/CISOfy/lynis.git
  cd lynis

  # Or via package manager (may be older)
  sudo apt install lynis    # Debian/Ubuntu
  ```
- **Usage:**
  ```bash
  # Full audit of the local system
  sudo ./lynis audit system

  # Quick, non-interactive scan
  sudo ./lynis audit system --quick --quiet

  # Audit and save report to a custom path
  sudo ./lynis audit system --report-file /tmp/lynis_report.dat

  # View the generated report
  cat /var/log/lynis-report.dat

  # Check hardening index score
  grep "hardening_index" /var/log/lynis-report.dat
  ```
- **Alternatives:** CIS-CAT (CIS Benchmark-based checks), OpenSCAP, commercial compliance scanners
- **Notes:** Lynis provides suggestions, not a pass/fail verdict. Prioritize findings based on your environment's risk profile — not every recommendation is equally applicable. Run from a cloned repo rather than the package manager version to get the latest checks. Enterprise version available with additional reporting features.

---

## osquery

- **Description:** Open-source framework from Meta that exposes the operating system as a relational database. Write SQL queries to retrieve live data about processes, network connections, users, files, hardware, installed software, and more — from a single host or a fleet.
- **Blue Team use:**
  - Query endpoint state using SQL during incident response or threat hunting
  - Hunt for IOCs across a fleet of endpoints via scheduled queries
  - Audit system configurations against security policies
  - Monitor for persistence mechanisms, unusual processes, or suspicious network activity
  - Integrate query results into SIEM for continuous monitoring
- **Website:** [osquery.io](https://osquery.io/) / [GitHub](https://github.com/osquery/osquery)
- **Type:** Endpoint instrumentation and query engine (daemon + `osqueryi` interactive shell)
- **Platform:** Linux, Windows, macOS, FreeBSD
- **Installation:** Download installers and packages from [osquery.io/downloads](https://osquery.io/downloads/).
- **Usage (`osqueryi` interactive shell):**
  ```sql
  -- Processes listening on network ports (excluding loopback)
  SELECT pid, name, port, address, protocol
  FROM listening_ports
  WHERE address != '127.0.0.1' AND address != '::1';

  -- Processes with no binary on disk (potential fileless malware)
  SELECT pid, name, path, cmdline
  FROM processes
  WHERE on_disk = 0;

  -- Recently modified files in system directories
  SELECT path, mtime, size
  FROM file
  WHERE path LIKE '/etc/%'
  AND mtime > (strftime('%s','now') - 3600);

  -- Autorun entries (startup persistence)
  SELECT name, path, source, status
  FROM startup_items;

  -- Users with active login sessions
  SELECT username, tty, host, time
  FROM logged_in_users;

  -- USB devices currently connected
  SELECT vendor, model, serial, removable
  FROM usb_devices;

  -- Installed packages (Linux)
  SELECT name, version, arch
  FROM deb_packages
  ORDER BY name;
  ```
- **Alternatives:** Sysinternals Suite (Windows, different approach), auditd (Linux native), commercial EDR agents (often include similar query capabilities)
- **Notes:** In production, osquery runs as a daemon (`osqueryd`) with scheduled queries and centralized log collection. `osqueryi` is for exploration and query development. Browse the full table schema at [osquery.io/schema](https://osquery.io/schema/). Query performance varies — test complex queries on `osqueryi` before deploying as scheduled queries.

---

## Snyk

- **Description:** Developer security platform for finding and fixing vulnerabilities in open-source dependencies, container images, Infrastructure as Code configurations, and application code. Integrates into IDEs, Git repos, and CI/CD pipelines.
- **Blue Team use:**
  - Scan application dependencies for known CVEs before or after deployment
  - Identify vulnerable OS packages and libraries in container images
  - Detect misconfigurations in Terraform, CloudFormation, and Kubernetes manifests
  - Integrate into CI/CD to block builds with critical vulnerabilities
  - Monitor deployed projects for newly disclosed vulnerabilities
- **Website:** [snyk.io](https://snyk.io/)
- **Type:** Developer security platform (web UI, CLI, IDE plugins, API) — commercial with free tier
- **Platform:** Web; CLI works on Windows, macOS, Linux
- **Installation:**
  ```bash
  npm install -g snyk
  snyk auth    # Authenticate with your Snyk account
  ```
- **Usage:**
  ```bash
  # Test dependencies in the current project directory
  snyk test

  # Test and show only critical/high findings
  snyk test --severity-threshold=high

  # Monitor a project (sends to Snyk dashboard, alerts on new CVEs)
  snyk monitor

  # Scan a container image
  snyk container test nginx:latest

  # Scan container image and show only fixable issues
  snyk container test nginx:latest --exclude-base-image-vulns

  # Scan IaC files for misconfigurations
  snyk iac test ./terraform/

  # Output results as JSON for integration
  snyk test --json > snyk_results.json
  ```
- **Alternatives:** OWASP Dependency-Check (dependencies), Trivy (containers and filesystems), Grype (containers), SonarQube (SAST), GitHub Advanced Security
- **Notes:** Free tier is generous for open-source projects. For container scanning, Trivy is a strong open-source alternative without requiring an account. Snyk's value increases with CI/CD integration — catching vulnerabilities at build time is more efficient than scanning deployed images.

---

## STIX / TAXII

- **Description:** Open standards for structured threat intelligence. STIX (Structured Threat Information Expression) is a JSON-based language for representing CTI objects — threat actors, malware, indicators, campaigns, TTPs, and relationships between them. TAXII (Trusted Automated Exchange of Intelligence Information) is the HTTPS-based protocol for exchanging STIX data between systems.
- **Blue Team use:**
  - Understand the data format used by MISP, OpenCTI, and commercial TIPs
  - Configure integrations between CTI platforms using TAXII feeds
  - Consume structured threat intelligence from government CERTs and ISACs
  - Produce and share structured intelligence with partners
- **Reference:** [OASIS CTI Technical Committee](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=cti) (governing standards body)
- **Type:** Open standards (language and protocol)
- **Platform:** N/A — implemented by CTI tools and libraries
- **Key STIX object types:**

  | Object | Description |
  |--------|-------------|
  | `indicator` | A pattern to detect a threat (e.g., file hash, IP, domain) |
  | `malware` | A malware family or instance |
  | `threat-actor` | A person or group conducting attacks |
  | `campaign` | A set of related adversary behaviors |
  | `attack-pattern` | A TTP (maps to MITRE ATT&CK) |
  | `relationship` | Links between any two objects |
  | `bundle` | A container for a set of STIX objects |

- **Usage (Python `stix2` library):**
  ```python
  from stix2 import Indicator, Malware, Relationship, Bundle

  malware = Malware(name="Emotet", malware_types=["trojan"])

  indicator = Indicator(
      name="Emotet C2 IP",
      pattern="[ipv4-addr:value = '192.0.2.1']",
      pattern_type="stix",
      valid_from="2024-01-01T00:00:00Z"
  )

  rel = Relationship(relationship_type="indicates", source_ref=indicator, target_ref=malware)
  bundle = Bundle(objects=[malware, indicator, rel])
  print(bundle.serialize(pretty=True))
  ```
- **Notes:** You don't need to write STIX manually — platforms like MISP and OpenCTI handle it for you. Understanding the object model helps when troubleshooting integrations or evaluating the quality of incoming feeds. STIX 2.1 is the current version; avoid STIX 1.x (XML-based, largely deprecated).

---

## Sysinternals Suite

- **Description:** Collection of over 70 free Windows utilities from Microsoft for advanced system management, troubleshooting, and analysis. Several tools are essential for Blue Team endpoint investigation and incident response.
- **Blue Team use:**
  - Investigate running processes, DLLs, handles, and network connections per process
  - Monitor file system, registry, and process activity in real time during malware analysis
  - Identify persistence mechanisms (autorun entries, scheduled tasks, services)
  - Deploy Sysmon for detailed Windows event logging to support SIEM detection
  - Inspect network connections and owning processes
- **Website:** [Microsoft Sysinternals](https://docs.microsoft.com/en-us/sysinternals/)
- **Type:** Windows analysis and troubleshooting utilities (GUI + CLI)
- **Platform:** Windows
- **Installation:** Download individual tools or the full suite ZIP from [Microsoft](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite). No installation required — run executables directly. Can also access live via `\\live.sysinternals.com\tools\`.

**Key tools for Blue Team work:**

| Tool | Purpose |
|------|---------|
| **Process Explorer** | Advanced task manager: process trees, loaded DLLs, handles, network connections per process |
| **Process Monitor (ProcMon)** | Real-time logging of file, registry, network, and process activity — essential for malware behavior analysis |
| **Autoruns** | Comprehensive view of all persistence mechanisms: startup entries, scheduled tasks, services, browser extensions |
| **Sysmon** | Windows service that logs detailed events (process creation, network connections, file changes) to the Event Log |
| **TCPView** | Live view of all TCP/UDP connections with owning process |
| **Strings** | Extracts printable strings from binary files |
| **Handle** | Shows open file handles per process |
| **PsExec** | Remote execution (know it because attackers use it) |

- **Usage:**
  ```bash
  # ProcMon: run with a filter to reduce noise
  Procmon.exe /filter "Process Name,is,suspicious.exe,include"

  # Autoruns: hide known Microsoft-signed entries to focus on unknowns
  # In GUI: Options > Scan Options > Check VirusTotal.com

  # Sysmon: install with a config file
  sysmon64.exe -accepteula -i sysmon_config.xml

  # Sysmon: update config without reinstalling
  sysmon64.exe -c new_config.xml

  # Strings: extract strings from a binary
  strings.exe -n 8 suspicious.exe > strings_output.txt

  # TCPView: command-line equivalent
  tcpvcon.exe -a
  ```
- **Alternatives:** Built-in Windows tools (Task Manager, Resource Monitor, Event Viewer — less detail), osquery (cross-platform SQL-based), commercial EDR
- **Notes:** Sysmon is the most impactful single tool here for ongoing detection — deploy it on all Windows endpoints with a well-maintained config. Use [SwiftOnSecurity's config](https://github.com/SwiftOnSecurity/sysmon-config) or [olafhartong's modular config](https://github.com/olafhartong/sysmon-modular) as starting points. In Autoruns, enable VirusTotal checking and hide Microsoft-signed entries to quickly surface suspicious persistence.

---

## Trivy

- **Description:** Open-source vulnerability and misconfiguration scanner from Aqua Security. Scans container images, filesystems, Git repositories, and IaC files. Fast, easy to use, and well-suited for CI/CD integration.
- **Blue Team use:**
  - Scan container images for CVEs in OS packages and application dependencies before deployment
  - Scan local project directories for vulnerable libraries
  - Detect misconfigurations in Terraform, CloudFormation, Kubernetes manifests, and Dockerfiles
  - Integrate into build pipelines to fail builds on critical vulnerabilities
  - Generate SBOMs (Software Bill of Materials) for deployed images
- **Website:** [aquasecurity.github.io/trivy](https://aquasecurity.github.io/trivy/) / [GitHub](https://github.com/aquasecurity/trivy)
- **Type:** CLI vulnerability and misconfiguration scanner
- **Platform:** Linux, macOS, Windows (binaries and container image)
- **Installation:**
  ```bash
  brew install aquasecurity/trivy/trivy    # macOS
  sudo apt install trivy                   # Debian/Ubuntu (via repo, check docs)
  # Or download binary from GitHub releases
  ```
- **Usage:**
  ```bash
  # Scan a container image
  trivy image nginx:latest

  # Scan and show only HIGH and CRITICAL findings
  trivy image --severity HIGH,CRITICAL nginx:latest

  # Scan a local filesystem directory
  trivy fs /path/to/project

  # Scan IaC files for misconfigurations
  trivy config ./terraform/

  # Scan a Git repository
  trivy repo https://github.com/example/repo

  # Output results as JSON
  trivy image --format json --output results.json nginx:latest

  # Generate a CycloneDX SBOM
  trivy image --format cyclonedx --output sbom.json nginx:latest

  # Update vulnerability database
  trivy image --download-db-only
  ```
- **Alternatives:** Grype (similar scope, Anchore), Clair (registry-integrated), Snyk Container (commercial features), Docker Scout (Docker-native)
- **Notes:** Keep the vulnerability database updated before running scans (`--download-db-only` or it updates automatically). For CI/CD, use `--exit-code 1` to fail the pipeline on findings above a threshold. Trivy's SBOM generation is useful for supply chain security tracking. Combine with Snyk if you need developer-facing remediation guidance rather than just detection.

---
