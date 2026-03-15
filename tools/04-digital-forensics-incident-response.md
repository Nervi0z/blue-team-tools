# 04 — Digital Forensics & Incident Response (DFIR)

DFIR covers the process of acquiring, preserving, and analyzing digital evidence to understand what happened during a security incident, contain damage, and support recovery. Evidence sources include disk images, memory dumps, logs, and network captures. Cloud environments are increasingly part of this scope.

## Tools in this section

- [Autopsy](#autopsy)
- [Chainsaw / Hayabusa](#chainsaw--hayabusa)
- [ExifTool](#exiftool)
- [FTK Imager](#ftk-imager)
- [KAPE](#kape)
- [Memory Acquisition Tools](#memory-acquisition-tools)
- [Plaso / Log2Timeline](#plaso--log2timeline)
- [Sleuth Kit](#sleuth-kit)
- [Steganalysis Tools](#steganalysis-tools)
- [Volatility 3](#volatility-3)
- [X-Ways Forensics](#x-ways-forensics)

---

## Autopsy

- **Description:** Open-source digital forensics platform providing a GUI over The Sleuth Kit and other analysis tools. Used for examining disk images, file systems, and some mobile devices.
- **Blue Team use:**
  - Analyze disk images (`.dd`, `.E01`) during incident investigations
  - Timeline analysis of file system activity
  - Keyword searching and indexing across large evidence sets
  - Web artifact analysis (browser history, cache, downloads)
  - Registry analysis on Windows disk images
  - Extensible via Python modules for custom functionality
- **Website:** [autopsy.com](https://www.autopsy.com/) / [GitHub](https://github.com/sleuthkit/autopsy)
- **Type:** Digital forensics platform (GUI)
- **Platform:** Windows, macOS, Linux
- **Installation:** Download installers from the official website.
- **Usage:** Create a new case, add a data source (disk image or physical drive), let the ingest modules run, then browse the file system, review artifacts, search keywords, and build a timeline. Select only the ingest modules you need — running all of them on a large image takes considerable time.
- **Alternatives:** FTK (commercial), EnCase (commercial), X-Ways Forensics (commercial), Sleuth Kit CLI tools
- **Notes:** Autopsy is a GUI wrapper around Sleuth Kit. For scripted or automated analysis, working with Sleuth Kit directly gives more control. Performance scales with image size and selected ingest modules.

---

## Chainsaw / Hayabusa

- **Description:** Fast, complementary tools for triage and threat hunting in Windows Event Logs (`.evtx` files). Chainsaw uses Sigma rules for detection; Hayabusa uses its own built-in detection logic. Both are designed to surface suspicious activity quickly across large volumes of event logs.
- **Blue Team use:**
  - Rapid triage of Windows event logs during incident response
  - Identify suspicious activity before loading logs into a SIEM
  - Hunt for known attack patterns using Sigma rule coverage (Chainsaw)
  - Generate timeline summaries of significant events (Hayabusa)
- **Website:** [Chainsaw — GitHub](https://github.com/countercept/chainsaw) / [Hayabusa — GitHub](https://github.com/Yamato-Security/hayabusa)
- **Type:** CLI log analysis / threat hunting tools
- **Platform:** Windows, Linux, macOS (pre-compiled binaries)
- **Installation:** Download binaries from GitHub releases for each tool.
- **Usage:**
  ```bash
  # Chainsaw: hunt using Sigma rules against a directory of EVTX files
  chainsaw hunt /path/to/evtx/ -r rules/ --mapping mappings/sigma-event-log-mapping.yml

  # Chainsaw: search for specific keywords across EVTX files
  chainsaw search "mimikatz" /path/to/evtx/

  # Hayabusa: scan EVTX directory, output results to CSV
  hayabusa csv-timeline -d /path/to/evtx/ -o timeline.csv

  # Hayabusa: generate a summary of detections
  hayabusa metrics -d /path/to/evtx/
  ```
- **Alternatives:** DeepBlueCLI (PowerShell), Event Log Explorer (GUI), SIEM queries (Splunk, Elastic)
- **Notes:** Use Chainsaw and Hayabusa together — their detection coverage overlaps but is not identical. Both are well-suited for the first pass during IR before loading logs into a SIEM. Keep Sigma rules updated for Chainsaw. Hayabusa's timeline output is particularly useful for building a chronological picture of activity.

---

## ExifTool

- **Description:** Command-line utility and Perl library for reading, writing, and editing metadata across a wide range of file types — images, video, audio, documents, and more.
- **Blue Team use:**
  - Extract timestamps (creation, modification) which may differ from filesystem metadata
  - Find GPS coordinates embedded in photos or videos
  - Identify the software used to create or modify a file
  - Detect anomalies or inconsistencies in metadata during investigations
  - Batch extract metadata from large collections of files
- **Website:** [exiftool.org](https://exiftool.org/)
- **Type:** CLI metadata analysis tool
- **Platform:** Windows, macOS, Linux
- **Installation:**
  ```bash
  sudo apt install libimage-exiftool-perl    # Debian/Ubuntu
  brew install exiftool                      # macOS
  ```
- **Usage:**
  ```bash
  # Show all metadata for a file
  exiftool image.jpg

  # Show only GPS data
  exiftool -gps* image.jpg

  # Show common fields in a readable format
  exiftool -common image.jpg

  # Batch extract metadata from a directory to CSV
  exiftool -csv -r /path/to/directory > metadata.csv

  # Show all tags including duplicates, with group names
  exiftool -a -G image.jpg

  # Extract metadata from all JPGs in a directory recursively
  exiftool -r -ext jpg /path/to/directory
  ```
- **Alternatives:** OS file properties (very limited), format-specific viewers
- **Notes:** Metadata can be stripped or modified — absence of metadata can itself be an indicator. Use `-a -G` to see all tags with group names when doing thorough analysis. Useful in both forensic investigations and OSINT contexts.

---

## FTK Imager

- **Description:** Free forensic imaging and preview tool from Exterro (formerly AccessData). Creates bit-for-bit forensic copies of storage media in multiple formats, mounts images as read-only volumes, previews file systems without altering the source, and captures live memory.
- **Blue Team use:**
  - Create forensically sound disk images (`.E01`, `.dd`) with MD5/SHA1 hash verification
  - Mount images as read-only for Browse without modifying evidence
  - Preview file systems on live systems or within images during triage
  - Recover files from unallocated space
  - Capture RAM from live Windows systems
- **Website:** [exterro.com/ftk-imager](https://www.exterro.com/ftk-imager)
- **Type:** Forensic imaging and preview tool (GUI)
- **Platform:** Windows (Linux CLI version less commonly used)
- **Installation:** Download from the official website (registration required).
- **Usage:**
  - **Disk imaging:** File > Create Disk Image — select source, output format (E01 recommended), set hash verification
  - **File system preview:** File > Add Evidence Item — select image or physical drive, browse without modifying
  - **Memory capture:** File > Capture Memory — specify output path, initiate capture
  - **Mount image:** File > Image Mounting — mount `.E01` or `.dd` as read-only drive letter
- **Alternatives:** `dd`/`dc3dd` (Linux CLI imaging), Guymager (Linux GUI imaging), Belkasoft RAM Capturer / Magnet RAM Capture (memory acquisition), Arsenal Image Mounter (mounting)
- **Notes:** Use a hardware write blocker when imaging original evidence drives. Always verify hash values after imaging completes. FTK Imager is a standard tool in most IR toolkits for its reliability and simplicity.

---

## KAPE

- **Description:** Free tool from Kroll for rapid forensic artifact collection and processing on Windows systems. Uses configurable YAML targets to collect specific artifacts and modules to parse them using external CLI tools — designed for fast IR triage rather than full forensic acquisition.
- **Blue Team use:**
  - Rapid collection of forensically relevant artifacts from live Windows systems or mounted images
  - Automated parsing of collected artifacts using integrated tool modules (RegRipper, PECmd, AppCompatibilityParser, etc.)
  - Collection from Volume Shadow Copies for historical artifact recovery
  - Triage before committing to a full disk image
- **Website:** [kroll.com/kape](https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kape)
- **Type:** CLI forensic artifact collection and processing tool (GUI wrapper: GKAPE)
- **Platform:** Windows
- **Installation:** Download from Kroll's website (registration required). Regularly update Targets and Modules from [KapeFiles on GitHub](https://github.com/Kroll-Cyber-Security/KapeFiles).
- **Usage:**
  ```bash
  # Collect artifacts from C: drive, process with modules, output to C:\IR
  kape.exe --tsource C: --tdest C:\IR\triage --tflush --target BasicCollection ^
           --vss true --mdest C:\IR\parsed --mflush --module !Disabled

  # Run modules only against previously collected artifacts
  kape.exe --msource C:\IR\triage\C --mdest C:\IR\parsed --mflush --module !Disabled

  # Collect specific target (e.g., event logs only)
  kape.exe --tsource C: --tdest C:\IR\evtx --tflush --target EventLogs
  ```
- **Alternatives:** Velociraptor (full endpoint agent, remote collection), GRR Rapid Response, custom collection scripts
- **Notes:** KAPE has become a standard tool for Windows IR triage. Keep Targets and Modules updated from the KapeFiles repo — the community adds new targets regularly. Understand what each Target collects before running in production. GKAPE.exe provides a GUI for building KAPE commands.

---

## Memory Acquisition Tools

- **Description:** Tools for capturing the contents of RAM on a live system. Memory contains running processes, network connections, loaded drivers, command history, encryption keys, and injected code — all of which are lost when the system powers down.
- **Blue Team use:**
  - Capture volatile evidence before shutting down a compromised system
  - Provide memory dumps for analysis with Volatility
  - Investigate fileless malware that exists only in memory
- **Tools:**

  | Tool | Platform | Notes |
  |------|----------|-------|
  | [Magnet RAM Capture](https://www.magnetforensics.com/resources/free-tools/magnet-ram-capture/) | Windows | Free GUI/CLI, widely used |
  | [Belkasoft RAM Capturer](https://belkasoft.com/ram-capturer) | Windows | Free, handles locked processes |
  | FTK Imager | Windows | Includes memory capture feature |
  | DumpIt | Windows | Simple CLI, legacy but functional |
  | LiME (Linux Memory Extractor) | Linux | Loadable kernel module, accurate capture |
  | `avml` (Microsoft) | Linux | Userspace memory acquisition |

- **Type:** Memory acquisition utilities (GUI/CLI)
- **Platform:** Primarily Windows for most free tools; LiME and avml for Linux
- **Usage (general):** Run with administrator/root privileges. Write output to an external drive, not the target system's local disk. Record the system's total RAM beforehand to verify dump completeness.
- **Notes:** Acquire memory as early as possible — every action on the system risks overwriting evidence in RAM. Minimize interaction with the target while acquiring. Destination drive must have at least as much free space as the system has RAM. For Linux, LiME captures more accurately than `/dev/mem` or `/proc/kcore`.

---

## Plaso / Log2Timeline

- **Description:** Framework for building comprehensive timelines from digital forensic evidence. `log2timeline.py` extracts timestamps from hundreds of artifact types (file system metadata, logs, registry hives, browser history, etc.) into a unified Plaso storage file. `psort.py` then filters and exports that data for analysis.
- **Blue Team use:**
  - Build a unified timeline of activity across multiple evidence sources
  - Correlate events from different artifact types by timestamp
  - Reconstruct the sequence of attacker actions during an incident
  - Filter and search events within a specific time window
- **Website:** [github.com/log2timeline/plaso](https://github.com/log2timeline/plaso)
- **Type:** CLI timeline creation and analysis framework
- **Platform:** Linux, macOS, Windows
- **Installation:**
  ```bash
  pip install plaso
  # Docker image also available — often easier for dependency management
  ```
- **Usage:**
  ```bash
  # Create a Plaso storage file from a disk image
  log2timeline.py plaso_output.plaso /path/to/disk_image.dd

  # Export filtered timeline to CSV (events in a specific time window)
  psort.py -o csv -w timeline.csv plaso_output.plaso \
    "date > '2024-06-01 08:00:00' AND date < '2024-06-01 12:00:00'"

  # Export to JSONL for SIEM ingestion
  psort.py -o json_line -w timeline.jsonl plaso_output.plaso

  # Filter by data source type
  psort.py -o csv -w timeline.csv plaso_output.plaso \
    "source_short is 'EVT'"
  ```
- **Alternatives:** Autopsy timeline feature, Timeline Explorer (GUI for CSV/bodyfile timelines), commercial forensic suite timelines
- **Notes:** Processing large images takes time and significant disk space — plan accordingly. Filtering with `psort.py` is essential; unfiltered output from a full disk image is unmanageable. Output integrates well with Timeline Explorer (Eric Zimmerman) for visual review.

---

## Sleuth Kit

- **Description:** Collection of CLI tools and a C library for forensic analysis of disk images and file systems. The underlying engine for Autopsy. Provides granular, low-level access to file system structures, deleted files, and metadata.
- **Blue Team use:**
  - Analyze NTFS, FAT, Ext3/4, HFS+, APFS, and other file systems in detail
  - Recover deleted files from unallocated space or file system metadata
  - Examine detailed inode/file metadata
  - Generate MAC time bodyfiles for timeline analysis
  - Integrate into scripts and automated analysis pipelines
- **Website:** [sleuthkit.org](https://www.sleuthkit.org/) / [GitHub](https://github.com/sleuthkit/sleuthkit)
- **Type:** CLI digital forensics toolkit and library
- **Platform:** Linux, macOS, Windows, BSD
- **Installation:**
  ```bash
  sudo apt install sleuthkit    # Debian/Ubuntu
  brew install sleuthkit        # macOS
  ```
- **Usage:**
  ```bash
  # Show file system information
  fsstat image.dd

  # List files and directories (including deleted)
  fls -r image.dd

  # Show detailed metadata for an inode
  istat image.dd <inode_number>

  # Extract a file by inode number
  icat image.dd <inode_number> > extracted_file

  # Generate MAC time bodyfile for timeline analysis
  fls -r -m / image.dd > bodyfile.txt
  mactime -b bodyfile.txt -d > timeline.csv

  # Find all deleted files
  fls -r -d image.dd
  ```
- **Alternatives:** Autopsy (GUI front-end for Sleuth Kit), commercial forensic suites
- **Notes:** Understanding file system internals makes Sleuth Kit significantly more useful. Autopsy provides a more accessible interface for most tasks, but direct Sleuth Kit use is better for scripted workflows and when you need precise control over what you're extracting.

---

## Steganalysis Tools

- **Description:** Tools for detecting data hidden inside files using steganography. Attackers use steganography for covert communication or exfiltration by embedding data in image, audio, or video files. Detection typically involves statistical analysis of the carrier file.
- **Blue Team use:**
  - Detect potential covert channels using steganography
  - Analyze suspicious media files during investigations
  - Identify anomalies in image files that could indicate hidden payloads
- **Tools:**

  | Tool | Notes |
  |------|-------|
  | [Aletheia](https://github.com/daniellerch/aletheia) | Python framework for image steganalysis, actively maintained |
  | [StegExpose](https://github.com/b3dk7/StegExpose) | Batch steganalysis for LSB steganography in images |
  | [zsteg](https://github.com/zed-0xff/zsteg) | Detects steg data in PNG and BMP files |
  | [steghide](https://steghide.sourceforge.net/) | Can both hide and extract data — useful for understanding common techniques |
  | [stego-toolkit](https://github.com/DominicBreuker/stego-toolkit) | Docker container with multiple steg/analysis tools pre-installed |

- **Type:** Steganalysis tools (CLI, various)
- **Platform:** Primarily Linux; some cross-platform
- **Usage:**
  ```bash
  # zsteg: check PNG/BMP for hidden data
  zsteg suspicious_image.png

  # steghide: attempt to extract without a password
  steghide extract -sf suspicious_image.jpg

  # Aletheia: LSB steganalysis on an image
  python aletheia.py lsbr-attack image.png
  ```
- **Notes:** Steganalysis is statistically probabilistic — most tools produce a likelihood assessment, not a definitive answer. No tool detects all steganography methods. For unknown techniques, comparing the suspect file against a known-clean version of the same original (if available) is the most reliable approach. Steganography in the wild is relatively uncommon but worth knowing in CTF and advanced IR contexts.

---

## Volatility 3

- **Description:** The primary open-source framework for memory forensics. Analyzes RAM dumps to extract digital artifacts: running processes, network connections, loaded modules, command history, registry keys, injected code, and cached credentials.
- **Blue Team use:**
  - Investigate system state at the time of memory capture
  - Find evidence of fileless malware that leaves no disk artifacts
  - Extract running process list, open network sockets, and loaded kernel modules
  - Recover command history (cmd.exe, PowerShell)
  - Identify process injection or rootkit activity
  - Dump cached password hashes from memory
- **Website:** [volatilityfoundation.org](https://www.volatilityfoundation.org/) / [GitHub — Volatility 3](https://github.com/volatilityfoundation/volatility3)
- **Type:** CLI memory forensics framework
- **Platform:** Linux, macOS, Windows (Python-based)
- **Installation:**
  ```bash
  pip install volatility3
  # Download symbol packs for target OS versions — see docs
  ```
- **Usage:**
  ```bash
  # List running processes
  python3 vol.py -f memory.vmem windows.pslist

  # Process tree view (shows parent-child relationships)
  python3 vol.py -f memory.vmem windows.pstree

  # List network connections
  python3 vol.py -f memory.vmem windows.netscan

  # Check command line arguments for each process
  python3 vol.py -f memory.vmem windows.cmdline

  # Dump password hashes
  python3 vol.py -f memory.vmem windows.hashdump

  # Find injected code (processes with VAD anomalies)
  python3 vol.py -f memory.vmem windows.malfind

  # List loaded DLLs for all processes
  python3 vol.py -f memory.vmem windows.dlllist

  # Scan for network artifacts (older/terminated connections)
  python3 vol.py -f memory.vmem windows.netstat
  ```
- **Alternatives:** Rekall (less actively maintained), Magnet AXIOM (commercial), EnCase (commercial)
- **Notes:** Volatility 3 handles symbol detection more automatically than Volatility 2. For memory dumps from uncommon OS versions, you may need to supply symbol tables manually — check the documentation. Pipe output to `grep` to filter large result sets. `windows.malfind` and `windows.pstree` are usually among the first plugins to run during malware investigations.

---

## X-Ways Forensics

- **Description:** Commercial Windows-based forensics suite known for performance on large datasets and low-level data access. Covers disk analysis, memory analysis, keyword searching, registry analysis, and case management.
- **Blue Team use:**
  - Fast processing of large disk images in enterprise IR contexts
  - File system analysis including deleted files and alternate data streams
  - Keyword searching and indexing across large evidence sets
  - Registry and memory analysis
  - Case management and court-ready reporting
- **Website:** [x-ways.net/forensics](https://www.x-ways.net/forensics/)
- **Type:** Commercial digital forensics suite (GUI)
- **Platform:** Windows
- **Installation:** Purchase license, download installer. Requires a hardware dongle or software activation.
- **Usage:** Create or open a case, add evidence items, allow processing, then use the integrated tools to browse files, run searches, filter by criteria, analyze registry and memory artifacts, and generate reports.
- **Alternatives:** EnCase (commercial), Magnet AXIOM (commercial), Autopsy (open source), FTK Suite (commercial)
- **Notes:** X-Ways is widely used in law enforcement and corporate DFIR for its performance and precision. It has a steeper learning curve than Autopsy but handles large evidence sets more efficiently. Requires a purchased license — not a free option.

---
