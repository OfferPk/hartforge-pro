\# HARTForge Pro (v1.0.0) — Operator & Engineer User Guide

\#\# 1\. Quick Start Guide

\#\#\# 1.1 Web Application Studio  
Open \`source-code/index.html\` in any modern web browser (Chrome, Edge, Firefox, Safari). Zero runtime dependencies or backend servers are required.  
\- \*\*Tab 1: Packet Dissector\*\*: Dissect raw HART-IP and Token-Passing PDU frames with instant bit-level field breakdown and 1-click PCAP export.  
\- \*\*Tab 2: PV & Command Studio\*\*: Synthesize Command 0, 1, 2, 3, or 48 requests and responses with live IEEE 754 floating-point conversion and LRC checksums.  
\- \*\*Tab 3: HART-IP Session Manager\*\*: Model and test Session Initiate (\`0x00\`), Keep-Alive (\`0x02\`), and Session Close (\`0x01\`) lifecycle states.  
\- \*\*Tab 4: IEC 62443 & NE107 Auditor\*\*: Evaluate field transmitter and gateway security postures against 10 critical process automation controls.  
\- \*\*Tab 5: Process Firewall & IDS Rules\*\*: Generate hardened Linux \`nftables\` conduit filters and Suricata Layer 7 intrusion detection signatures.

\#\#\# 1.2 Standalone Python CLI (\`hartforge-cli.py\`)  
Run directly from terminal using Python 3.8+. Zero external pip dependencies required.

\`\`\`bash  
\# Make executable  
chmod \+x build/hartforge-cli.py

\# 1\. Build a Command 1 Read Primary Variable (PV) Request  
./build/hartforge-cli.py cmd1-req

\# 2\. Build a Command 1 PV Response (Value \= 101.325 kPa, Unit Code \= 12\)  
./build/hartforge-cli.py cmd1-resp \--value 101.325 \--unit 12

\# 3\. Build a Command 0 Read Unique Identifier Request  
./build/hartforge-cli.py cmd0 \--addr 0

\# 4\. Build a Command 48 NAMUR NE107 Diagnostics Request  
./build/hartforge-cli.py cmd48

\# 5\. Build a HART-IP Session Initiate Packet (60s Inactivity Timeout)  
./build/hartforge-cli.py session-init \--timeout 60000

\# 6\. Dissect raw HART-IP frame hex  
./build/hartforge-cli.py dissect "0100030000020018869624123456010700002041cb3333e8"

\# 7\. Run automated IEC 62443 / NE107 security audit  
./build/hartforge-cli.py audit

\# 8\. Synthesize nftables conduit rules and Suricata signatures  
./build/hartforge-cli.py synthesize-firewall \--gw-ip 192.168.10.20

\# 9\. Export sample HART-IP sequence to Wireshark-compatible PCAP  
./build/hartforge-cli.py export-pcap \--out hartip\_sample.pcap  
\`\`\`

\---

\#\# 2\. CLI Command Reference

\#\#\# \`cmd1-resp\`  
Synthesizes a HART-IP response packet carrying simulated transmitter measurements.  
\- \`--value\`: Float value of the primary variable (e.g. \`101.325\`, \`25.4\`, \`4.0\`). Default: \`101.325\`.  
\- \`--unit\`: HART Engineering Unit Code integer:  
  \- \`12\`: kPa (Kilopascals)  
  \- \`6\`: psi (Pounds per square inch)  
  \- \`7\`: bar  
  \- \`32\`: °C (Degrees Celsius)  
  \- \`33\`: °F (Degrees Fahrenheit)  
  \- \`24\`: L/min (Liters per minute)  
  \- \`65\`: mA (Milliamperes)

\#\#\# \`dissect\`  
Parses raw hex bytes into structured JSON representation, validating:  
\- HART-IP 8-byte header (Version, MsgType, MsgID, Status, Sequence Number, Byte Count).  
\- Delimiter and 5-byte Long Address (Manufacturer, Device Type, Device ID).  
\- Response Status and NAMUR NE107 flags (Device Malfunction, Configuration Changed, Cold Start).  
\- Decoded IEEE 754 32-bit floating-point measurements.  
\- Longitudinal Redundancy Check (LRC) checksum validity.

\#\#\# \`export-pcap\`  
Generates a Libpcap 2.4 capture file containing synthetic Ethernet, IPv4, UDP (Port 5094), and HART-IP frames ready to open in Wireshark.

\---

\#\# 3\. Engineering Reference & FAQs

\*\*Q: What is the difference between HART-IP v1 and v2?\*\*  
A: HART-IP v1 operates standard unencrypted UDP/TCP communication. HART-IP v2 introduces native DTLS (Datagram Transport Layer Security) for encrypted telemetry and certificate-based mutual authentication.

\*\*Q: How does HARTForge Pro handle LRC Checksums?\*\*  
A: The Longitudinal Redundancy Check is calculated via XOR across all bytes from the delimiter through the last payload data byte. HARTForge Pro computes and validates LRCs automatically.  
