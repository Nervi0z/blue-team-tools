# 03 — Phishing Analysis & Defense

Phishing is one of the most common initial access vectors. This section covers tools for analyzing suspicious emails, URLs, and attachments, as well as running internal simulation exercises to measure user awareness.

## Tools in this section

- [Any.Run](#anyrun)
- [Browserling](#browserling)
- [Email Header Analysis](#email-header-analysis)
- [GoPhish](#gophish)
- [PhishTank](#phishtank)
- [urlscan.io](#urlscanio)
- [VirusTotal](#virustotal)

---

## Any.Run

- **Description:** Interactive online sandbox for detonating suspicious files and URLs in a safe cloud environment. Provides real-time process monitoring, network traffic capture, screenshots, and video of execution.
- **Blue Team use:**
  - Safely execute email attachments or open suspicious links
  - Observe malware behavior in real time (process tree, file changes, registry modifications)
  - Capture and review C2 traffic or secondary downloads
  - Interact with the running environment (click through installers, navigate pages)
- **Website:** [any.run](https://any.run/)
- **Type:** Online sandbox (freemium)
- **Platform:** Web (browser)
- **Installation:** N/A
- **Usage:** Upload a file or submit a URL via the web interface. Monitor execution in real time or review the completed report.
- **Alternatives:** Hybrid Analysis, Joe Sandbox (commercial), CAPE Sandbox (open source), local VM
- **Notes:** Free tier submissions are public — avoid submitting files containing sensitive data. One of the better options for dynamic analysis of unknown attachments from phishing emails.

---

## Browserling

- **Description:** Online cross-browser testing service. Blue Teams use it to safely render suspicious URLs inside a remote sandboxed browser without exposing the local machine or real IP address.
- **Blue Team use:**
  - Open potentially malicious links without risk to local systems
  - See how a phishing page renders across different browser/OS combinations
  - Observe basic page behavior without exposing your real browser fingerprint
- **Website:** [browserling.com](https://www.browserling.com/)
- **Type:** Online browser sandbox / cross-browser testing tool (freemium)
- **Platform:** Web (browser)
- **Installation:** N/A
- **Usage:** Paste the suspicious URL into the interface, select a browser and OS, and interact with the remote session.
- **Alternatives:** urlscan.io (automated analysis, no interaction), Any.Run (more detailed behavioral analysis), local VM with a browser
- **Notes:** Free tier limits session length. Good for a quick look at a suspicious page's appearance when you don't need deep behavioral analysis.

---

## Email Header Analysis

- **Description:** Technique for investigating suspicious emails by examining their full headers. Headers reveal the delivery path, originating IP, and authentication results (SPF, DKIM, DMARC), which are key indicators of spoofing or manipulation.
- **Blue Team use:**
  - Identify the true sending IP of a suspicious email
  - Verify SPF, DKIM, and DMARC authentication results
  - Trace the email's delivery path through `Received:` headers
  - Detect inconsistencies or anomalies suggesting spoofing
- **Tools:** [MXToolbox Header Analyzer](https://mxtoolbox.com/EmailHeaders.aspx), [Google Admin Toolbox Messageheader](https://toolbox.googleapps.com/apps/messageheader/)
- **Type:** Technique + online analysis tools
- **Platform:** N/A
- **Installation:** N/A
- **Usage:**
  1. Open the suspicious email and access the full source/headers (Gmail: "Show original"; Outlook: File > Properties, or View > All Headers)
  2. Copy the entire header block
  3. Paste into MXToolbox or Google Messageheader and review the output
  4. Read `Received:` headers bottom-up — the first hop is the originating mail server
  5. Check `Authentication-Results:` for SPF, DKIM, and DMARC pass/fail status
- **Notes:** The originating IP from the lowest `Received:` header is the most useful artifact for threat intel lookups. SPF/DKIM/DMARC failures don't automatically mean malicious, but they warrant closer investigation. `Authentication-Results` is often the fastest way to triage a spoofing attempt.

---

## GoPhish

- **Description:** Open-source phishing simulation framework. Blue Teams and security awareness programs use it to run controlled phishing exercises against employees, measure click rates and credential submission, and assess training effectiveness.
- **Blue Team use:**
  - Create realistic phishing email templates and landing pages for internal exercises
  - Launch simulated campaigns and track who clicked, opened, or submitted credentials
  - Measure baseline user awareness before and after training
  - Identify high-risk user groups or departments
- **Website:** [getgophish.com](https://getgophish.com/) / [GitHub](https://github.com/gophish/gophish)
- **Type:** Phishing simulation framework (web UI)
- **Platform:** Linux, Windows, macOS (distributed as binaries)
- **Installation:** Download the release binary, configure `config.json`, run the executable. Accessible via web UI.
- **Usage:** Set up sending profiles (SMTP), email templates, landing pages, and target user groups. Launch campaigns and monitor results in the dashboard.
- **Alternatives:** KnowBe4, Proofpoint Security Awareness Training, Cofense (all commercial), phishing-frenzy (open source)
- **Notes:** Only use for internal training with explicit organizational authorization. Coordinate with HR and legal before running campaigns. GoPhish is the framework — landing page and email quality determine how realistic and useful the exercise is.

---

## PhishTank

- **Description:** Community-driven database of verified phishing URLs. Users submit suspected phishing sites, the community votes on them, and verified entries are available via search and API.
- **Blue Team use:**
  - Check whether a suspicious URL has already been identified as a phishing site
  - Pull a feed of active phishing URLs for blocking or threat intel purposes
  - Submit newly discovered phishing URLs to help the community
- **Website:** [phishtank.org](https://phishtank.org/)
- **Type:** Community phishing database / threat intel feed
- **Platform:** Web (browser), API
- **Installation:** N/A
- **Usage:** Search a URL on the site to check its status. For automation, register for an API key and query the feed programmatically.
- **Alternatives:** OpenPhish, Google Safe Browse API, commercial threat intel feeds
- **Notes:** Community verification introduces a lag — a new phishing site won't appear until someone submits it and it gets verified. Treat this as one data point alongside VirusTotal and urlscan.io, not a definitive source.

---

## urlscan.io

- **Description:** Online service that browses submitted URLs like a regular user and captures a full record of the interaction: contacted IPs and domains, loaded resources, redirects, page screenshot, and DOM content.
- **Blue Team use:**
  - Safely analyze a suspicious URL without exposing your own IP or browser
  - Trace redirect chains
  - Identify malicious infrastructure contacted by the page
  - Detect brand impersonation via screenshot and domain analysis
  - View page content and technologies without direct interaction
- **Website:** [urlscan.io](https://urlscan.io/)
- **Type:** Online URL scanner / website analysis tool (freemium)
- **Platform:** Web (browser), API
- **Installation:** N/A
- **Usage:** Submit the URL via the web interface or API. Review the report for contacted IPs/domains, redirect chain, page screenshot, and detected indicators.
- **Alternatives:** Browserling (interactive), Any.Run (deeper behavioral analysis), VirusTotal (multi-engine URL check)
- **Notes:** Free scans are public by default — use a paid account or the API with a private flag if confidentiality matters. One of the most useful tools for quickly profiling a suspicious link from an email or message.

---

## VirusTotal

- **Description:** Online service that scans files and URLs against 70+ antivirus engines and URL/domain blocklists simultaneously. Owned by Google. Also provides relationship mapping between files, URLs, domains, and IPs based on community intelligence.
- **Blue Team use:**
  - Check email attachments and URLs against multiple AV engines in one submission
  - Review detection ratios as a quick triage indicator
  - Explore relationships between a file and known malicious infrastructure
  - Access behavioral analysis and sandbox results for submitted files
- **Website:** [virustotal.com](https://www.virustotal.com/)
- **Type:** Online file/URL scanner / threat intel aggregator (freemium)
- **Platform:** Web (browser), API
- **Installation:** N/A
- **Usage:** Upload a file or paste a URL/IP/domain/hash into the search box. Review the detection summary, then check the **Relations** and **Behavior** tabs for additional context.
- **Alternatives:** Hybrid Analysis, MetaDefender Cloud, Joe Sandbox
- **Notes:** A clean result doesn't guarantee safety — zero-day threats and recently created malware often evade detection initially. A high detection ratio across multiple engines is a strong indicator of malice. Avoid uploading files that contain sensitive organizational data — all submissions are accessible to other VirusTotal users. Hash-first lookups (`sha256:...`) let you check if a file is known without uploading it.

---
