\# HARTForge Pro (v1.0.0) — Technical Architecture

\#\# 1\. System Overview  
\*\*HARTForge Pro\*\* is an industrial-grade protocol engineering studio, Process Variable (PV) & dynamic variables codec, NAMUR NE107 diagnostics analyzer, and IEC 62443 cybersecurity auditor for \*\*HART-IP (IEC 62591 / FieldComm Group)\*\* and \*\*WirelessHART\*\* process automation systems.

HART (Highway Addressable Remote Transducer) is the world's most pervasive process field communication standard (\>40 million active devices across oil & gas, chemicals, pharmaceuticals, power generation, and water treatment). HARTForge Pro addresses the modern ethernet transition to \*\*HART-IP over UDP/TCP Port 5094\*\*, providing zero-dependency wire frame synthesis, bit-level dissection, Libpcap 2.4 capture generation, and automated industrial defense synthesis (Linux \`nftables\` process conduits and Suricata Layer 7 signatures).

\---

\#\# 2\. Protocol Encapsulation & Packet Architecture

\`\`\`  
\+---------------------------------------------------------------+  
\| Layer 7b: HART Token-Passing PDU (Application Commands)      \|  
\| Delim \| Address (1B/5B) \| Cmd (0,1,3,48) \| Byte Count \| Data \| LRC \|  
\+---------------------------------------------------------------+  
\| Layer 7a: HART-IP Protocol Header (8 Bytes)                   \|  
\| Version (1B) \| MsgType (1B) \| MsgID (1B) \| Status (1B)       \|  
\| Sequence Number (2B BE) \| Total Byte Count (2B BE)            \|  
\+---------------------------------------------------------------+  
\| Layer 4: UDP or TCP (Port 5094\)                               \|  
\+---------------------------------------------------------------+  
\| Layer 3: IPv4 / IPv6 Process Automation Subnet                \|  
\+---------------------------------------------------------------+  
\| Layer 2: Ethernet / 802.3 Physical Conduit                   \|  
\+---------------------------------------------------------------+  
\`\`\`

\#\#\# 2.1 HART-IP Header Specification (IEC 62591\)  
The standard HART-IP header comprises 8 contiguous bytes:  
\- \*\*Byte 0 — Version\*\*: Fixed protocol revision (\`0x01\` for HART-IP v1, \`0x02\` for HART-IP v2).  
\- \*\*Byte 1 — Message Type\*\*:  
  \- \`0x00\`: Request (Client to Server / Gateway).  
  \- \`0x01\`: Response (Server / Gateway to Client).  
  \- \`0x02\`: Publish (Asynchronous burst telemetry).  
  \- \`0x03\`: NAK (Negative Acknowledgement).  
\- \*\*Byte 2 — Message ID / Service\*\*:  
  \- \`0x00\`: Session Initiate (Establish keep-alive and client master role).  
  \- \`0x01\`: Session Close (Orderly session teardown).  
  \- \`0x02\`: Keep-Alive (Heartbeat maintaining UDP session state).  
  \- \`0x03\`: Token-Passing PDU (Encapsulates native HART command frames).  
  \- \`0x04\`: Direct PDU.  
\- \*\*Byte 3 — Status Code\*\*: \`0x00\` denotes Success; non-zero denotes error conditions.  
\- \*\*Bytes 4..5 — Transaction Sequence Number\*\*: 16-bit Big-Endian incrementing counter.  
\- \*\*Bytes 6..7 — Byte Count\*\*: 16-bit Big-Endian total packet length (8-byte header \+ payload).

\---

\#\# 3\. HART Token-Passing PDU Architecture

\#\#\# 3.1 Frame Delimiters & Addressing  
1\. \*\*Short Frame (Legacy / Polling)\*\*:  
   \- Delimiter \`0x02\` (Master Request) or \`0x06\` (Slave Response).  
   \- 1-byte address: Polling address \`0\` to \`15\` (\`0x80\` bit set for primary master).  
2\. \*\*Long Frame (HART 5/6/7 Unique Identifier)\*\*:  
   \- Delimiter \`0x82\` (Master Request) or \`0x86\` (Slave Response).  
   \- 5-byte address:  
     \- Byte 0: Master bit (\`0x80\`), Burst bit (\`0x40\`), Manufacturer Code lower 6 bits.  
     \- Byte 1: Expanded Device Type.  
     \- Bytes 2..4: 24-bit unique device serial identifier.

\#\#\# 3.2 Longitudinal Redundancy Check (LRC)  
The HART PDU concludes with a 1-byte Longitudinal Redundancy Check (LRC):  
\$\$\\text{LRC} \= \\text{Delimiter} \\oplus \\text{Address}\_0 \\dots \\oplus \\text{Address}\_n \\oplus \\text{Command} \\oplus \\text{ByteCount} \\oplus \\text{Data}\_0 \\dots \\oplus \\text{Data}\_k\$\$  
Evaluating the XOR sum over the entire PDU (including the LRC) yields \`0x00\`.

\#\#\# 3.3 Core Supported Commands  
\- \*\*Command 0 (\`0x00\`)\*\*: Read Unique Identifier (Manufacturer, Device Type, Revision, Device ID).  
\- \*\*Command 1 (\`0x01\`)\*\*: Read Primary Variable (PV) — returns Engineering Unit Code (1B) \+ IEEE 754 32-bit Float (4B).  
\- \*\*Command 2 (\`0x02\`)\*\*: Read Loop Current and Percent of Range.  
\- \*\*Command 3 (\`0x03\`)\*\*: Read Dynamic Variables (PV, SV, TV, QV).  
\- \*\*Command 48 (\`0x30\`)\*\*: Read Additional Device Status (NAMUR NE107 diagnostics: Failure, Check Function, Out of Spec, Maintenance).

\---

\#\# 4\. Security & Compliance Architecture (IEC 62443 & NAMUR NE107)

1\. \*\*Zone & Conduit Isolation (IEC 62443-3-2)\*\*:  
   \- Terminating HART-IP (UDP/TCP 5094\) within Purdue Model Level 2/3 Process DMZs.  
   \- Restricting engineering management commands strictly to authorized Asset Management Stations (AMS).  
2\. \*\*Calibration & Span Tampering Defense (FR 1 / FR 2)\*\*:  
   \- Monitoring for unauthorized Command 35 (Write Range Values) and Command 45 (Trim DAC Current).  
   \- Validating physical transmitter write-protect jumper state.  
3\. \*\*WirelessHART Gateway Security\*\*:  
   \- Auditing joining keys (detecting default or weak 128-bit AES keys).  
4\. \*\*Intrusion Detection & Deep Packet Inspection\*\*:  
   \- Synthesizing Suricata Layer 7 signatures and Linux \`nftables\` process firewalls.

\---

\#\# 5\. Software Architecture

\`\`\`  
product\_169/  
├── source-code/  
│   ├── index.html                  \# 5-Tab Industrial Process Studio Web App  
│   ├── hart-forge.css              \# Refinery amber & deep process glassmorphism styling  
│   ├── app.js                      \# Application controller & reactive UI bindings  
│   ├── hart-protocol-engine.js     \# HART-IP & Token-Passing PDU codecs, LRC, Libpcap engine  
│   ├── hart-security-auditor.js    \# 10-point IEC 62443 & NAMUR NE107 audit matrix  
│   └── hart-config-synthesizer.js  \# nftables & Suricata ruleset generator  
├── build/  
│   ├── hartforge-cli.py            \# Standalone Python 3 CLI tool (chmod \+x)  
│   └── hartforge-pro-v1.0.0.zip    \# Release package archive  
├── documentation/  
│   ├── ARCHITECTURE.md             \# Technical specification & protocol deep-dive  
│   ├── USER\_GUIDE.md               \# Operator instructions & CLI reference  
│   ├── MONETIZATION\_AND\_MARKETING\_PLAYBOOK.md \# Pricing, ICPs, and GTM strategy  
│   └── PROGRESS\_REPORT.md          \# Verification log & milestone report  
├── screenshots/  
│   ├── hart-studio-preview.svg     \# UI preview vector graphic  
│   └── hart-architecture-diagram.svg \# Protocol stack architecture diagram  
└── README.md                       \# Comprehensive product documentation  
\`\`\`  
