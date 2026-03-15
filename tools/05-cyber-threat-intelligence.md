# 05 — Cyber Threat Intelligence (CTI)

CTI covers collecting, analyzing, and applying information about adversaries, their infrastructure, and their methods. For Blue Teams, this means understanding IOCs, TTPs, and threat actor behavior to prioritize defenses, improve detection, and speed up incident response.

## Tools in this section

- [abuse.ch (URLhaus, MalwareBazaar, ThreatFox)](#abusech-urlhaus-malwarebazaar-threatfox)
- [AbuseIPDB](#abuseipdb)
- [AlienVault OTX](#alienvault-otx)
- [Censys](#censys)
- [Hunchly](#hunchly)
- [Maltego](#maltego)
- [MISP](#misp)
- [OpenCTI](#opencti)
- [Recon-ng](#recon-ng)
- [SpiderFoot](#spiderfoot)
- [TheHive](#thehive)
- [YARA](#yara)

---

## abuse.ch (URLhaus, MalwareBazaar, ThreatFox)

- **Description:** Non-profit projects tracking and sharing specific threat types. URLhaus collects malicious URLs used for malware distribution; MalwareBazaar shares malware samples; ThreatFox shares IOCs associated with malware families.
- **Blue Team use:**
  - Look up suspicious URLs, file hashes, and IOCs against current threat data
  - Pull feeds to enrich SIEM alerts or populate blocklists
  - Submit newly discovered samples or URLs to contribute to the community
- **Website:** [abuse.ch](https://abuse.ch/) — [URLhaus](https://urlhaus.abuse.ch/) / [MalwareBazaar](https://bazaar.abuse.ch/) / [ThreatFox](https://threatfox.abuse.ch/)
- **Type:** Threat intelligence feeds and repositories
- **Platform:** Web portal, API
- **Installation:** N/A
- **Usage:**
  ```bash
  # ThreatFox API: search for an IOC
  curl -X POST https://threatfox-api.abuse.ch/api/v1/ \
    -H "Content-Type: application/json" \
    -d '{"query": "search_ioc", "search_term": "192.0.2.1"}'

  # MalwareBazaar API: look up a file hash
  curl -X POST https://mb-api.abuse.ch/api/v1/ \
    -H "Content-Type: application/json" \
    -d '{"query": "get_info", "hash": "YOUR_SHA256_HASH"}'

  # URLhaus API: check a URL
  curl -X POST https://urlhaus-api.abuse.ch/v1/url/ \
    -d 'url=http://suspicious-example.com/malware'
  ```
- **Alternatives:** PhishTank (phishing URLs), OTX, MISP feeds
- **Notes:** Focused on specific threat types rather than broad intelligence — best used alongside a general platform like OTX or MISP. All three projects provide bulk download options for offline integration.

---

## AbuseIPDB

- **Description:** Community-driven IP reputation database aggregating reports of malicious activity from sysadmins, security teams, and researchers. Covers spam sources, scanning activity, brute force attempts, DDoS participation, and more.
- **Blue Team use:**
  - Check IP reputation during alert triage or log analysis
  - Automate IP lookups during incident response
  - Report malicious IPs observed in your environment
- **Website:** [abuseipdb.com](https://www.abuseipdb.com/)
- **Type:** IP reputation database / threat intel feed
- **Platform:** Web portal, API
- **Installation:** N/A
- **Usage:**
  ```bash
  # API lookup for an IP address (requires free API key)
  curl -G https://api.abuseipdb.com/api/v2/check \
    --data-urlencode "ipAddress=192.0.2.1" \
    -d maxAgeInDays=90 \
    -H "Key: YOUR_API_KEY" \
    -H "Accept: application/json"

  # Report a malicious IP
  curl https://api.abuseipdb.com/api/v2/report \
    --data-urlencode "ip=192.0.2.1" \
    -d categories=18,22 \
    -d comment="Port scanning observed" \
    -H "Key: YOUR_API_KEY"
  ```
- **Alternatives:** Cisco Talos Reputation Center, VirusTotal IP lookup, commercial IP reputation services
- **Notes:** Community-sourced data means accuracy varies — weight recent reports and high confidence scores more heavily. Useful for quick triage, not as a definitive verdict. Integrate the API into SIEM workflows for automated enrichment.

---

## AlienVault OTX

- **Description:** Open community threat intelligence platform where users and organizations share IOC collections ("Pulses") related to specific threats, campaigns, and malware families. Maintained by AlienVault with contributions from the community and AlienVault Labs.
- **Blue Team use:**
  - Search for known IOCs (IPs, domains, hashes, URLs) against community data
  - Subscribe to Pulses relevant to your industry or observed threat actors
  - Pull IOC feeds via API for automated ingestion into SIEM or TIP
  - Contribute Pulses when you identify new threat infrastructure
- **Website:** [otx.alienvault.com](https://otx.alienvault.com/)
- **Type:** Community threat intelligence platform and feed
- **Platform:** Web portal, API
- **Installation:** N/A
- **Usage:**
  ```bash
  # API: look up an IP indicator
  curl https://otx.alienvault.com/api/v1/indicators/IPv4/192.0.2.1/general \
    -H "X-OTX-API-KEY: YOUR_API_KEY"

  # API: get all IOCs from a specific Pulse
  curl https://otx.alienvault.com/api/v1/pulses/PULSE_ID/indicators \
    -H "X-OTX-API-KEY: YOUR_API_KEY"

  # API: get your subscribed Pulses updated since a date
  curl "https://otx.alienvault.com/api/v1/pulses/subscribed?modified_since=2024-01-01" \
    -H "X-OTX-API-KEY: YOUR_API_KEY"
  ```
- **Alternatives:** MISP (more structured sharing), ThreatFox/abuse.ch (specific threat types), commercial TIPs
- **Notes:** Pulse quality varies significantly. Prioritize Pulses from AlienVault Labs, verified contributors, and organizations in your sector. Use the API to automate IOC ingestion rather than manually checking the portal.

---

## Censys

- **Description:** Internet-wide scanning platform that continuously indexes IPv4 space, websites, and certificates. Searchable via web or API. Similar to Shodan but with stronger emphasis on certificate transparency and structured data.
- **Blue Team use:**
  - Discover services and devices associated with your IP ranges or domains
  - Identify exposed ports, software versions, and potential misconfigurations on external assets
  - Monitor certificate transparency logs for new certificates issued for your domains
  - Research infrastructure potentially linked to threat actors
- **Website:** [search.censys.io](https://search.censys.io/)
- **Type:** Internet scan data search engine / attack surface management tool (freemium)
- **Platform:** Web portal, API
- **Installation:** N/A
- **Usage:**
  ```bash
  # Web search examples:
  # Find hosts in your IP range:
  #   ip:203.0.113.0/24

  # Find hosts with your domain in TLS cert:
  #   services.tls.certificates.leaf_data.subject.common_name:example.com

  # Find specific service on your infrastructure:
  #   services.http.response.html_title:"Company Portal" and ip:203.0.113.0/24

  # API: search for hosts in a CIDR range
  curl -X POST https://search.censys.io/api/v2/hosts/search \
    -H "Content-Type: application/json" \
    -u "API_ID:API_SECRET" \
    -d '{"q": "ip:203.0.113.0/24", "per_page": 100}'
  ```
- **Alternatives:** Shodan, Zoomeye, BinaryEdge, commercial ASM platforms
- **Notes:** Free accounts have limited daily queries. Certificate transparency monitoring is a standout feature — useful for detecting unauthorized certificate issuance for your domains. Combine with Shodan for broader external attack surface coverage.

---

## Hunchly

- **Description:** Commercial Chrome browser extension for online investigations. Automatically captures, timestamps, and cryptographically hashes every webpage visited during an investigation, creating a reproducible and auditable evidence trail.
- **Blue Team use:**
  - Document CTI research and OSINT investigations with automatic timestamped captures
  - Preserve evidence of threat actor infrastructure before it goes offline
  - Tag and annotate specific data points across captured pages
  - Generate structured investigation reports
- **Website:** [hunch.ly](https://www.hunch.ly/)
- **Type:** Browser extension / OSINT evidence capture tool (commercial)
- **Platform:** Chrome extension (Windows, macOS, Linux)
- **Installation:** Install from the Chrome Web Store, purchase a license.
- **Usage:** Start a new case in the Hunchly dashboard. Browse normally — the extension automatically captures all visited pages. Add tags and notes to specific pages or data points. Export a report at the end of the investigation.
- **Alternatives:** Manual screenshots + notes (less systematic), SingleFile extension (page saving without case management), web.archive.org (public archive, not under your control)
- **Notes:** Particularly valuable when investigating infrastructure that may disappear quickly. The automatic hash and timestamp on each capture establishes an evidence chain. Required for serious CTI investigations where documentation integrity matters.

---

## Maltego

- **Description:** Graphical link analysis and OSINT platform. Uses "Transforms" to query data sources (WHOIS, DNS, Shodan, VirusTotal, Passive DNS, and many others) and visualizes the resulting relationships between entities on an interactive graph.
- **Blue Team use:**
  - Map relationships between IOCs (domains, IPs, email addresses, hashes, ASNs)
  - Identify threat actor infrastructure patterns
  - Map your organization's external footprint graphically
  - Pivot from one indicator to related infrastructure
- **Website:** [maltego.com](https://www.maltego.com/)
- **Type:** Graphical link analysis / intelligence visualization platform (commercial, Community Edition available)
- **Platform:** Windows, macOS, Linux (Java-based)
- **Installation:** Download from the official website. Registration required. Community Edition is free with limitations.
- **Usage:** Start a new graph. Add entity nodes (Domain, IP, Email, etc.). Run Transforms on nodes to query data sources and discover related entities. Analyze the resulting graph for connections and patterns. Use the Transform Hub to install additional data source integrations.
- **Alternatives:** OpenCTI (structured data storage, less visualization focus), SpiderFoot (automation focus), custom scripting with graph visualization libraries
- **Notes:** The Community Edition caps results per Transform. Professional use generally requires a paid license for access to more data sources and higher result limits. The value depends heavily on which Transforms you have access to — configure API keys for third-party integrations (Shodan, VirusTotal, etc.).

---

## MISP

- **Description:** Open-source Threat Intelligence Platform built specifically for structured sharing of threat data — IOCs, TTPs, threat actor information, vulnerabilities, and reports — within trusted communities or organizations. Uses a standardized data model with Events, Attributes, Objects, and Galaxies.
- **Blue Team use:**
  - Store and correlate IOCs from internal investigations and external feeds
  - Share threat intelligence with trusted partners or sector communities
  - Import external feeds (abuse.ch, OTX, government CERTs) for automated enrichment
  - Link IOCs to threat actors, malware families, and TTPs via Galaxies
  - Integrate with TheHive for IR enrichment and with SIEMs for automated blocking
- **Website:** [misp-project.org](https://www.misp-project.org/) / [GitHub](https://github.com/MISP/MISP)
- **Type:** Threat intelligence platform (TIP) / sharing platform (web UI)
- **Platform:** Linux (Ubuntu/Debian). VM and Docker deployment options available.
- **Installation:** Follow the [official installation guide](https://misp.github.io/MISP/). Requires LAMP stack dependencies. Docker deployment available for faster setup.
- **Usage:**
  ```bash
  # API: search for events containing a specific attribute value
  curl https://your-misp-instance/events/restSearch \
    -H "Authorization: YOUR_API_KEY" \
    -H "Accept: application/json" \
    -H "Content-Type: application/json" \
    -X POST \
    -d '{"value": "192.0.2.1", "type": "ip-dst"}'

  # API: add an attribute to an event
  curl https://your-misp-instance/attributes/add/EVENT_ID \
    -H "Authorization: YOUR_API_KEY" \
    -H "Content-Type: application/json" \
    -X POST \
    -d '{"type": "domain", "value": "malicious-example.com", "to_ids": true}'
  ```
- **Alternatives:** OpenCTI (broader knowledge management), TheHive (IR case management), commercial TIPs (Anomali, ThreatQuotient), OTX (community feed)
- **Notes:** MISP is designed for sharing, not just storage. The value increases significantly when connected to external MISP communities or CERTs. Understand the data model (Events → Attributes → Objects → Galaxies) before building workflows around it. TheHive + MISP + Cortex is a common open-source IR stack.

---

## OpenCTI

- **Description:** Open-source platform for structuring, storing, and visualizing threat intelligence knowledge using the STIX 2 standard. Links threat actors, intrusion sets, malware, TTPs, vulnerabilities, IOCs, and reports into a unified, queryable knowledge base.
- **Blue Team use:**
  - Maintain a structured CTI knowledge base linked to STIX 2 relationships
  - Correlate malware with threat actors, campaigns, and TTPs automatically
  - Import data from MISP, threat feeds, and reports via connectors
  - Visualize relationships between threat entities
  - Export intelligence to detection tools or share with partners
- **Website:** [opencti.io](https://www.opencti.io/) / [GitHub](https://github.com/OpenCTI-Platform/opencti)
- **Type:** Threat intelligence platform (TIP) / knowledge management platform (web UI)
- **Platform:** Docker deployment (Linux host recommended)
- **Installation:** Follow the [official Docker deployment guide](https://docs.opencti.io/latest/deployment/installation/). Requires Docker and Docker Compose.
- **Usage:** Deploy via Docker Compose. Access the web UI. Configure connectors to import data from MISP, external feeds, or reports. Create entities manually (Threat Actors, Malware, Reports, Indicators). Explore relationships using graph views, dashboards, and the investigation workbench.
- **Alternatives:** MISP (sharing focus), TheHive (IR case management), Maltego (visualization focus), commercial TIPs
- **Notes:** OpenCTI and MISP serve different but complementary purposes — MISP for sharing IOC-level data, OpenCTI for building a broader knowledge graph. They integrate well together. Steeper setup and learning curve than simpler feed aggregators but more powerful for building long-term CTI programs.

---

## Recon-ng

- **Description:** Modular web reconnaissance framework in Python with a Metasploit-inspired interactive console. Each module targets a specific data source or task — DNS lookups, WHOIS queries, API integrations (Shodan, VirusTotal, HaveIBeenPwned, etc.). Collected data is stored in a local database per workspace.
- **Blue Team use:**
  - Structured, repeatable OSINT collection against your own domains and infrastructure
  - Discover subdomains, email addresses, hosts, and related infrastructure
  - Integrate with threat intelligence APIs for enrichment
  - Build automated recon workflows
- **Website:** [GitHub](https://github.com/lanmaster53/recon-ng)
- **Type:** CLI OSINT framework
- **Platform:** Linux, macOS, Windows (Python-based)
- **Installation:**
  ```bash
  git clone https://github.com/lanmaster53/recon-ng.git
  cd recon-ng
  pip install -r REQUIREMENTS
  ./recon-ng
  ```
- **Usage:**
  ```bash
  # Inside recon-ng console:

  # Create a workspace for a target
  workspaces create example_org

  # Add a domain to scope
  db insert domains example.com

  # Search available modules
  modules search dns

  # Load and run a module
  modules load recon/domains-hosts/hackertarget
  run

  # Add an API key for a module
  keys add shodan_api YOUR_SHODAN_KEY

  # View collected data
  show hosts
  show contacts
  ```
- **Alternatives:** SpiderFoot (web UI, broader automation), Maltego (GUI, visualization), theHarvester (simpler, faster email/subdomain collection)
- **Notes:** Many of the most useful modules require API keys — configure them with `keys add` before running those modules. Use workspaces to keep different target investigations separated. Good for structured, repeatable workflows where you want control over each step.

---

## SpiderFoot

- **Description:** Open-source OSINT automation platform with a web UI and CLI. Runs 200+ modules across a wide range of data sources to automatically collect intelligence on domains, IPs, email addresses, names, and other targets.
- **Blue Team use:**
  - Automate broad OSINT collection against your organization's external presence
  - Discover related infrastructure, leaked credentials, domain associations, and threat intel matches
  - Visualize relationships between discovered data points
  - Identify exposure in code repositories, paste sites, and breach databases
- **Website:** [spiderfoot.net](https://www.spiderfoot.net/) / [GitHub](https://github.com/smicallef/spiderfoot)
- **Type:** OSINT automation platform (web UI + CLI)
- **Platform:** Linux, Windows, macOS (Python-based), Docker image available
- **Installation:**
  ```bash
  pip install spiderfoot

  # Or from source
  git clone https://github.com/smicallef/spiderfoot.git
  cd spiderfoot
  pip install -r requirements.txt
  ```
- **Usage:**
  ```bash
  # Start the web UI
  python sf.py -l 127.0.0.1:5001
  # Access http://127.0.0.1:5001 — configure API keys in Settings

  # CLI scan against a domain using all modules
  python sf.py -s example.com -u ALL

  # CLI scan with specific module types only
  python sf.py -s example.com -u OSINT

  # Output to JSON
  python sf.py -s example.com -u ALL -o json -q
  ```
- **Alternatives:** Recon-ng (more manual control, Metasploit-style), Maltego (stronger visualization), theHarvester (simpler, faster for specific tasks)
- **Notes:** Configure API keys in the web UI settings before running scans — many modules degrade significantly without them. Start with a targeted module subset rather than ALL for initial investigations; the full scan takes time and generates large amounts of data. Docker is the easiest deployment method.

---

## TheHive

- **Description:** Open-source Security Incident Response Platform (SIRP) for managing cases, tracking observables, assigning tasks, and collaborating across analyst teams. Integrates with Cortex (automated analysis engine) and MISP (CTI sharing).
- **Blue Team use:**
  - Centralize incident case management across the team
  - Track IOCs (observables) associated with each case
  - Assign and monitor investigation tasks
  - Enrich observables automatically via Cortex analyzers (VirusTotal, Shodan, MISP lookups, etc.)
  - Push confirmed IOCs to MISP for broader sharing
  - Maintain investigation timelines and documentation
- **Website:** [thehive-project.org](https://thehive-project.org/) / [GitHub](https://github.com/TheHive-Project/TheHive)
- **Type:** Security incident response platform (SIRP) (web UI)
- **Platform:** Linux (packages or Docker). Web UI accessed via browser.
- **Installation:** Follow the [official installation guide](https://docs.strangebee.com/thehive/installation/). Docker deployment available. Cortex and MISP are separate but commonly deployed alongside it.
- **Usage:** Create a case for each incident or alert. Add observables (IP, domain, hash, email, URL). Assign tasks to analysts. Write log entries documenting findings. Run Cortex analyzers on observables for automatic enrichment. Close or escalate the case when complete.
- **Alternatives:** Commercial SOAR platforms (Splunk SOAR, Palo Alto XSOAR), RTIR (open-source ticketing adapted for IR)
- **Notes:** TheHive is most effective as part of a stack: TheHive (case management) + Cortex (automated analysis) + MISP (CTI sharing). Without Cortex, observable enrichment is manual. Integration setup requires time but significantly improves analyst efficiency at scale.

---

## YARA

- **Description:** Rule-based pattern matching engine for identifying files based on textual or binary patterns. Blue Teams write or import YARA rules to scan files, memory dumps, and network captures for malware signatures, attacker tools, or specific indicators.
- **Blue Team use:**
  - Detect known malware families by pattern during file scanning or memory analysis
  - Scan endpoint or file server artifacts during threat hunting
  - Integrate rules into sandbox tools, IDS, and forensic platforms for automated detection
  - Write custom rules to hunt for attacker-specific artifacts identified during IR
- **Website:** [virustotal.github.io/yara](https://virustotal.github.io/yara/) / [GitHub](https://github.com/VirusTotal/yara)
- **Type:** Pattern matching engine and rule language (CLI tool + library)
- **Platform:** Linux, Windows, macOS
- **Installation:**
  ```bash
  pip install yara-python       # Python bindings + CLI
  sudo apt install yara         # Debian/Ubuntu native package
  ```
- **Usage:**
  ```bash
  # Example YARA rule (save as detect_mimikatz.yar):
  rule Mimikatz_Strings {
      meta:
          description = "Detects Mimikatz strings"
          author = "example"
      strings:
          $s1 = "sekurlsa::logonpasswords" ascii wide nocase
          $s2 = "mimikatz" ascii wide nocase
          $s3 = "lsadump::dcsync" ascii wide nocase
      condition:
          any of them
  }

  # Scan a single file
  yara detect_mimikatz.yar suspicious.exe

  # Scan a directory recursively
  yara -r detect_mimikatz.yar /path/to/scan/

  # Scan a memory dump (useful with Volatility output)
  yara detect_mimikatz.yar memory.dmp

  # Use multiple rule files
  yara rule1.yar rule2.yar /path/to/scan/
  ```
- **Alternatives:** ClamAV (AV engine with its own signature format), Sigma (log-based detection rules), commercial EDR detection rules
- **Notes:** Public YARA rule repositories worth following: [Awesome YARA](https://github.com/InQuest/awesome-yara), [YARA-Rules project](https://github.com/Yara-Rules/rules), [Elastic's detection rules](https://github.com/elastic/detection-rules). Writing good rules requires understanding both the malware and how to avoid false positives on legitimate files. Test new rules against a clean corpus before deploying. YARA integrates natively with Volatility, allowing memory scanning during forensic analysis.

---
