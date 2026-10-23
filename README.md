\# HARTForge Pro (v1.0.0)  
\#\#\# Industrial HART-IP (IEC 62591 / FieldComm Group) & WirelessHART Protocol Studio, Process Variable (PV) Codec, NAMUR NE107 Diagnostics, & IEC 62443 Security Auditor

\---

\#\# 🏭 Overview  
\*\*HARTForge Pro\*\* is a high-performance, zero-dependency protocol engineering platform and cybersecurity auditing suite built specifically for \*\*HART-IP (IEC 62591 / FieldComm Group)\*\* and \*\*WirelessHART\*\* process automation systems.

HART is the world's most ubiquitous process field communication technology, deployed across more than 40 million instruments (pressure transmitters, magnetic flowmeters, RTD temperature sensors, valve positioners) in energy refineries, chemical plants, offshore platforms, pharmaceutical cleanrooms, and municipal water utilities.

Communicating over UDP and TCP port 5094, HARTForge Pro bridges field instrumentation engineering with modern industrial OT cybersecurity operations:  
1\. \*\*Frame Constructor & Codec\*\*: Generate valid HART-IP packets and Token-Passing PDUs for Command 0 (Unique ID), Command 1 (Read PV), Command 2 (Loop Current), Command 3 (Dynamic Variables), and Command 48 (NAMUR NE107 Diagnostics).  
2\. \*\*Deep Wire Dissector\*\*: Parse raw hex captures into structured HART-IP 8-byte headers, Short/Long addresses, response status codes, decoded IEEE 754 floating-point measurements, and validated Longitudinal Redundancy Check (LRC) checksums.  
3\. \*\*Session Lifecycle Manager\*\*: Model and test HART-IP Session Initiate (\`0x00\`), Keep-Alive (\`0x02\`), and Session Close (\`0x01\`) socket states.  
4\. \*\*Automated IEC 62443 & NAMUR NE107 Security Auditor\*\*: Evaluate transmitter installations and gateway conduits against 10 critical process cybersecurity controls.  
5\. \*\*Defense Synthesizer\*\*: Output ready-to-deploy Linux \`nftables\` process conduit rules and Suricata Layer 7 intrusion detection signatures.  
6\. \*\*Wireshark PCAP Exporter\*\*: Export generated packet sequences directly into standard Libpcap 2.4 files with synthetic Ethernet/IPv4/UDP headers (Port 5094\) for offline inspection.

\---

\#\# 🚀 Key Features

\- \*\*Zero External Dependencies\*\*: Operates 100% offline in modern web browsers (pure HTML5/Vanilla JS/CSS) or via standalone Python 3 CLI.  
\- \*\*Air-Gapped Plant Operational Safety\*\*: Built for completely isolated OT networks with zero cloud or internet prerequisites.  
\- \*\*Purdue Model Alignment\*\*: Tailored for Level 2/3 Process Control Network integration and OT DMZ gateway auditing.  
\- \*\*Dual-Interface Flexibility\*\*: Includes an intuitive 5-tab dark refinery glassmorphism web studio and an automated headless CLI for CI/CD or field diagnostics.

\---

\#\# 📦 Directory Structure

\`\`\`  
product\_169/  
├── source-code/  
│   ├── index.html                  \# 5-Tab Industrial Process Studio  
│   ├── hart-forge.css              \# Dark refinery glassmorphism design  
│   ├── app.js                      \# Application controller & UI bindings  
│   ├── hart-protocol-engine.js     \# Core framing, PDU codecs, LRC, Libpcap engine  
│   ├── hart-security-auditor.js    \# 10-point IEC 62443 & NE107 audit engine  
│   └── hart-config-synthesizer.js  \# nftables & Suricata ruleset generator  
├── build/  
│   ├── hartforge-cli.py            \# Standalone Python 3 CLI tool (chmod \+x)  
│   └── hartforge-pro-v1.0.0.zip    \# Complete release distribution archive  
├── documentation/  
│   ├── ARCHITECTURE.md             \# Technical specification & protocol deep-dive  
│   ├── USER\_GUIDE.md               \# Operator manual & CLI command guide  
│   ├── MONETIZATION\_AND\_MARKETING\_PLAYBOOK.md \# Pricing models, ICPs, and GTM strategy  
│   └── PROGRESS\_REPORT.md          \# Verification log & milestone report  
├── screenshots/  
│   ├── hart-studio-preview.svg     \# UI preview vector graphic  
│   └── hart-architecture-diagram.svg \# Protocol stack architecture diagram  
└── README.md                       \# Master product documentation  
\`\`\`

\---

\#\# 🛠️ CLI Usage Examples

\`\`\`bash  
\# Make CLI executable  
chmod \+x build/hartforge-cli.py

\# 1\. Build Command 1 Read Primary Variable (PV) Request  
./build/hartforge-cli.py cmd1-req

\# 2\. Build Command 1 PV Response (Value \= 101.325 kPa, Unit Code \= 12\)  
./build/hartforge-cli.py cmd1-resp \--value 101.325 \--unit 12

\# 3\. Build Command 0 Read Unique Identifier Request  
./build/hartforge-cli.py cmd0 \--addr 0

\# 4\. Build Command 48 NAMUR NE107 Diagnostics Request  
./build/hartforge-cli.py cmd48

\# 5\. Build HART-IP Session Initiate Packet (60s Inactivity Timeout)  
./build/hartforge-cli.py session-init \--timeout 60000

\# 6\. Dissect raw HART-IP frame hex  
./build/hartforge-cli.py dissect "0100030000020018869624123456010700002041cb3333e8"

\# 7\. Run automated IEC 62443 / NE107 security audit  
./build/hartforge-cli.py audit

\# 8\. Synthesize nftables conduit rules & Suricata signatures  
./build/hartforge-cli.py synthesize-firewall \--gw-ip 192.168.10.20

\# 9\. Export HART-IP sequence to PCAP file  
./build/hartforge-cli.py export-pcap \--out hartip\_sample.pcap  
\`\`\`

\---

\#\# 🧪 Automated Testing  
Run the complete unit test suite validating all protocol components:  
\`\`\`bash  
python3 test\_hartforge.py \-v  
\`\`\`  
All 10 test suites cover HART-IP 8-byte headers, Token-Passing PDU delimiters, LRC XOR checksums, Command 0/1/48 codecs, IEEE 754 float conversion, Session Initiate framing, PCAP generation, and IEC 62443 auditing.

\---

\#\# 📄 License & Attribution  
Developed by \*\*Autonomous Digital Product Factory Agent\*\* under Commercial Industrial Dual-License (Community Open-Core / Commercial Enterprise).  
