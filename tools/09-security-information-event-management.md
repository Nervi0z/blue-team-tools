# SIEM Ops & Detection Engineering

Technical documentation and implementation notes for Security Information and Event Management (SIEM) systems, log lifecycle management, and detection engineering.

## Architecture & Components

### 1. Elastic Security (ELK Stack)
A unified platform for search, analysis, and security operations.
* **Ingestion**: Filebeat (logs), Winlogbeat (Windows Events), Packetbeat (Network).
* **Processing**: Logstash for filtering, normalization (ECS mapping), and enrichment.
* **Storage/Search**: Elasticsearch as the distributed analytics engine.
* **Visualization**: Kibana for dashboarding and alert management.
* **Note**: Advanced features like ML-based anomaly detection require specific licensing (Gold/Platinum).

### 2. Graylog
Log management focused on high-speed indexing and processing pipelines.
* **Dependencies**: Requires MongoDB for metadata and OpenSearch/Elasticsearch for storage.
* **Pipelines**: Rule-based data manipulation prior to indexing.
* **GELF**: Graylog Extended Log Format for structured logging across distributed systems.

### 3. Security Onion
A specialized Linux distribution for Network Security Monitoring (NSM).
* **Engines**: Integrates Suricata (IDS), Zeek (NSM), and Wazuh (HIDS).
* **Storage**: Custom Elastic Stack implementation optimized for security event retention.
* **Analyst Tools**: Pre-configured suite including CyberChef, NetworkMiner, and Wireshark.

### 4. Wazuh
XDR and SIEM platform focused on endpoint visibility and compliance.
* **Manager**: Centralized server for rule processing and alert generation.
* **Indexer**: Scalable search engine for security event data.
* **Dashboard**: Web interface for compliance reporting (PCI-DSS, GDPR, NIST 800-53).

## Detection Engineering

### Log Normalization
Standardization is required for cross-platform correlation. This repository follows the **Elastic Common Schema (ECS)** for field naming conventions:
* `source.ip` / `destination.ip`
* `user.name`
* `event.action`
* `process.executable`

### Sigma Standard
Vendor-agnostic detection logic. Use `sigmac` or `pySigma` to translate these into backend-specific queries (Splunk SPL, Elastic KQL, etc.).

**Detection Example (PowerShell Suspicious Download):**
```yaml
title: Suspicious PowerShell Download
status: stable
description: Detects PowerShell net.webclient download patterns often used in malware staging.
logsource:
    product: windows
    service: powershell
detection:
    selection:
        EventID: 4104
        ScriptBlockText|contains|all:
            - 'Net.WebClient'
            - 'DownloadFile'
    condition: selection
level: high
```

## Implementation Comparison

| System | Primary Use Case | Scaling Method | License |
|---|---|---|---|
| **Elastic** | Enterprise Search / SIEM | Cluster nodes (Sharding) | Elastic/SSPL |
| **Graylog** | Log Aggregation / IT Ops | Load-balanced inputs | GPL/Enterprise |
| **Security Onion** | Network Monitoring / IDS | Distributed sensors | GPL |
| **Wazuh** | Endpoint Security / XDR | Manager-Worker nodes | GPL |
| **Splunk** | Advanced Analytics | Indexer clustering | Proprietary (500MB Free) |

## Operational Notes

* **Hardware Sizing**: SIEM clusters are I/O and RAM intensive. A minimum of **16GB RAM** is recommended for standalone lab environments.
* **Parser Tuning**: Regularly update decoders and extractors to prevent "log leakage," ensuring all fields are correctly indexed for correlation.
* **Egress Costs**: For cloud-native deployments (Azure Sentinel, Elastic Cloud), optimize ingestion filters at the source to minimize data volume costs.

## License
MIT
