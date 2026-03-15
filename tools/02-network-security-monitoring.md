# 02 — Network Security Monitoring (NSM)

Network Security Monitoring involves collecting and analyzing network traffic to detect intrusions, anomalous behavior, and policy violations. This section covers packet capture, protocol analysis, flow logging, and intrusion detection.

## Tools in this section

- [Ettercap](#ettercap)
- [Snort](#snort)
- [Suricata](#suricata)
- [tcpdump](#tcpdump)
- [Wireshark / TShark](#wireshark--tshark)
- [Zeek](#zeek)

---

## Ettercap

- **Description:** Suite built around man-in-the-middle (MITM) attacks on LANs, including ARP poisoning and DNS spoofing. Blue Team value here is primarily educational — understanding how these attacks work in order to detect and defend against them.
- **Blue Team use:**
  - Understand ARP poisoning and DNS spoofing mechanics in a lab environment
  - Inform configuration of switch-level defenses (Dynamic ARP Inspection, port security)
  - Understand why unencrypted protocols are a risk on internal networks
- **Website:** [ettercap-project.org](https://www.ettercap-project.org/) / [GitHub](https://github.com/Ettercap/ettercap)
- **Type:** GUI + CLI MITM suite
- **Platform:** Linux (primary); macOS and Windows ports exist but are less stable
- **Installation:**
  ```bash
  sudo apt install ettercap-graphical    # Debian/Ubuntu
  ```
- **Usage:** Lab and educational use only. Not used in production Blue Team operations.
- **Alternatives:** Dedicated ARP monitoring tools, Snort/Suricata/Zeek for detection, Wireshark for traffic analysis
- **Notes:** Only use on networks you own or have explicit written permission to test. The main Blue Team takeaway from Ettercap is understanding the attack surface, not running the tool operationally.

---

## Snort

- **Description:** Open-source network IDS/IPS using a rule-based language for real-time traffic analysis and packet logging. Detects known threats, protocol anomalies, and policy violations via signature matching.
- **Blue Team use:**
  - Detect known threats using community and commercial rule sets
  - Log packets that trigger alerts for deeper analysis
  - Enforce network security policies
  - Run as a passive IDS or inline IPS depending on deployment
- **Website:** [snort.org](https://www.snort.org/)
- **Type:** Network IDS/IPS
- **Platform:** Linux, BSD, macOS, Windows (best performance on Linux/BSD)
- **Installation:** Via package manager or source. Snort 3 recommended — check official docs for current install procedure.
  ```bash
  sudo apt install snort    # Debian/Ubuntu (verify version available)
  ```
- **Usage:**
  ```bash
  # IDS mode: monitor interface eth0, log to /var/log/snort, print alerts to console
  sudo snort -c /etc/snort/snort.conf -l /var/log/snort -i eth0 -A console
  ```
- **Alternatives:** Suricata (multi-threaded, better performance on modern hardware), Zeek (protocol analysis approach), commercial IDS/IPS
- **Notes:** Effective detection depends heavily on rule quality and update frequency. Use PulledPork or similar to manage rule updates (Snort Community Rules, Emerging Threats, Talos). Tune rules to reduce false positives — an untuned ruleset generates more noise than signal.

---

## Suricata

- **Description:** High-performance open-source IDS/IPS/NSM engine developed by OISF. Multi-threaded design gives significant throughput advantages on multi-core hardware compared to single-threaded engines.
- **Blue Team use:**
  - High-throughput network intrusion detection and prevention
  - Protocol detection on non-standard ports
  - File extraction from HTTP, SMTP, FTP, NFS, SMB traffic for offline analysis
  - JSON-based logging (Eve JSON) for SIEM integration (Elastic Stack, Splunk)
  - Custom detection logic via Lua scripting
- **Website:** [suricata.io](https://suricata.io/)
- **Type:** Network IDS/IPS/NSM engine
- **Platform:** Linux, BSD, macOS, Windows
- **Installation:**
  ```bash
  # Ubuntu — OISF PPA (recommended)
  sudo add-apt-repository ppa:oisf/suricata-stable
  sudo apt update && sudo apt install suricata
  ```
- **Usage:**
  ```bash
  # IDS mode on interface eth0
  sudo suricata -c /etc/suricata/suricata.yaml -i eth0

  # Update rules
  sudo suricata-update
  ```
- **Alternatives:** Snort (single-threaded, large rule ecosystem), Zeek (protocol logging focus), commercial IDS/IPS
- **Notes:** Rule management via `suricata-update` — keep rules current. For high-traffic environments, tune CPU affinity and capture method (AF_PACKET or PF_RING). Eve JSON output integrates well with most SIEMs and makes Suricata a natural fit in a log pipeline alongside Zeek.

---

## tcpdump

- **Description:** Lightweight command-line packet analyzer. Captures and displays packets on a network interface using BPF filter syntax. Found pre-installed on most Unix-like systems and essential for quick captures in headless or scripted environments.
- **Blue Team use:**
  - Quick packet captures during incident response or troubleshooting
  - Capture traffic to `.pcap` files for analysis in Wireshark
  - Scriptable capture in environments without a GUI
  - Filter traffic to specific hosts, ports, or protocols during collection
- **Website:** [tcpdump.org](https://www.tcpdump.org/)
- **Type:** CLI packet analyzer
- **Platform:** Linux, macOS, BSD, other Unix-like systems (pre-installed on most). Windows version available (requires Npcap).
- **Installation:**
  ```bash
  sudo apt install tcpdump    # Debian/Ubuntu (often pre-installed)
  ```
- **Usage:**
  ```bash
  # Basic capture on eth0, disable name/port resolution
  sudo tcpdump -i eth0 -nn

  # Capture traffic to/from a specific host, save to file
  sudo tcpdump -i eth0 host 192.168.1.50 -w capture.pcap

  # Capture only DNS traffic
  sudo tcpdump -i eth0 port 53 -nn

  # Capture traffic to/from a subnet
  sudo tcpdump -i eth0 net 192.168.1.0/24 -w subnet_capture.pcap

  # Capture HTTP traffic and display payload in ASCII
  sudo tcpdump -i eth0 port 80 -A
  ```
- **Alternatives:** TShark (Wireshark CLI, better for protocol dissection and field extraction), ngrep (pattern matching on packet content)
- **Notes:** Learn BPF filter syntax — it's used by tcpdump, Wireshark capture filters, and other tools. Use `-nn` to skip DNS lookups during capture (avoids latency and keeps the output clean). Always use `-w` when capturing for later analysis rather than relying on terminal scrollback.

---

## Wireshark / TShark

- **Description:** Network protocol analyzer with deep dissection capabilities for hundreds of protocols. Wireshark provides a GUI for interactive analysis; TShark is the CLI equivalent for scripted capture, field extraction, and use on servers without a display.
- **Blue Team use:**
  - Deep inspection of packet captures during incident response
  - Stream reconstruction (TCP, HTTP, TLS after decryption)
  - Protocol-aware filtering to isolate relevant traffic
  - Statistics and flow analysis
  - Scripted analysis and field extraction via TShark
- **Website:** [wireshark.org](https://www.wireshark.org/)
- **Type:** GUI network analyzer (Wireshark) + CLI network analyzer (TShark)
- **Platform:** Windows, macOS, Linux
- **Installation:**
  ```bash
  sudo apt install wireshark    # Debian/Ubuntu — installs both Wireshark and TShark
  ```
- **Usage:**

  **Wireshark (GUI):** Open a capture file or start a live capture. Apply display filters to narrow down traffic, then follow streams to reconstruct sessions.

  Common display filters:
  ```
  ip.addr == 192.168.1.50
  http.request.method == "POST"
  dns.qry.name contains "suspicious"
  tcp.flags.syn == 1 && tcp.flags.ack == 0
  tls.handshake.type == 1
  ```

  **TShark (CLI):**
  ```bash
  # Live capture on eth0, write to file
  sudo tshark -i eth0 -w capture.pcapng

  # Read capture, filter DNS queries, extract fields
  tshark -r capture.pcapng -Y "dns.flags.response == 0" \
    -T fields -e frame.time -e ip.src -e dns.qry.name

  # Extract HTTP request URIs from a capture
  tshark -r capture.pcapng -Y "http.request" \
    -T fields -e ip.src -e http.host -e http.request.uri
  ```
- **Alternatives:** tcpdump (capture focus, lighter), NetworkMiner (file/artifact extraction from pcaps), ngrep (CLI pattern matching)
- **Notes:** Capture filters (BPF syntax) are applied at collection time — use them to limit capture size. Display filters (Wireshark syntax) are applied after capture — use them for analysis. Following streams (`Follow > TCP Stream`, `Follow > HTTP Stream`) is one of the most useful features for understanding a session. TShark field extraction with `-T fields -e` is very useful for building analysis pipelines.

---

## Zeek

- **Description:** Open-source network security monitoring framework. Rather than signature matching, Zeek deeply parses network protocols and generates structured logs describing network activity — connections, DNS queries, HTTP requests, TLS certificates, file transfers, and more. Highly extensible via its scripting language.
- **Blue Team use:**
  - Generate high-fidelity protocol logs for incident response and threat hunting (`conn.log`, `dns.log`, `http.log`, `ssl.log`, `files.log`)
  - Detect behavioral patterns and anomalies via custom Zeek scripts
  - Extract files transferred over HTTP, SMTP, FTP, SMB for analysis
  - Feed structured logs into a SIEM for correlation and long-term retention
  - Understand encryption posture (TLS versions, certificate details)
- **Website:** [zeek.org](https://zeek.org/)
- **Type:** Network security monitoring framework / network analysis tool
- **Platform:** Linux, BSD, macOS
- **Installation:** Via package manager or source. Check the [official docs](https://docs.zeek.org/) for current recommended method.
  ```bash
  sudo apt install zeek    # Debian/Ubuntu (verify repo setup in official docs)
  ```
- **Usage:**
  ```bash
  # Monitor live interface eth0 with default scripts
  sudo zeek -i eth0

  # Analyze an existing pcap file
  zeek -r capture.pcapng

  # Parse conn.log to review connection summaries
  cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto service duration

  # Find all DNS queries in dns.log
  cat dns.log | zeek-cut ts id.orig_h query qtype_name answers

  # Find HTTP requests with non-standard user agents
  cat http.log | zeek-cut id.orig_h id.resp_h uri user_agent
  ```
- **Alternatives:** Snort/Suricata (signature-based, complement Zeek well), commercial NSM platforms, SIEMs (consume Zeek logs)
- **Notes:** Zeek generates substantial log volume — plan for storage and ship logs to a SIEM (Elastic Stack and Splunk both have good Zeek integrations). Zeek and Suricata are commonly deployed together: Suricata for signature-based alerting, Zeek for protocol-level logging. Learning the Zeek scripting language is worth the investment for custom detections. Understand what each log file contains before building detection logic on top of it.

---
