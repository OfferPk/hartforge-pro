\<\!DOCTYPE html\>  
\<html lang="en"\>  
\<head\>  
  \<meta charset="UTF-8"\>  
  \<meta name="viewport" content="width=device-width, initial-scale=1.0"\>  
  \<title\>HARTForge Pro \| Industrial HART-IP \&amp; WirelessHART Protocol Studio\</title\>  
  \<link rel="stylesheet" href="hart-forge.css"\>  
  \<link rel="preconnect" href="https\://fonts.googleapis.com"\>  
  \<link rel="preconnect" href="https\://fonts.gstatic.com" crossorigin\>  
  \<link href="https\://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800\&family=JetBrains+Mono:wght@400;500;600;700\&display=swap" rel="stylesheet"\>  
\</head\>  
\<body\>  
  \<div class="container"\>  
    \<header\>  
      \<div class="brand"\>  
        \<div class="logo-badge"\>HART\</div\>  
        \<div class="brand-info"\>  
          \<h1\>HARTForge Pro\</h1\>  
          \<p\>Industrial HART-IP (IEC 62591 / FieldComm Group) \&amp; WirelessHART Protocol Studio, PV Codec \&amp; IEC 62443 Auditor\</p\>  
        \</div\>  
      \</div\>  
      \<div class="header-status"\>  
        \<span class="badge badge-hart"\>HART-IP v1/v2\</span\>  
        \<span class="badge badge-emerald"\>UDP/TCP 5094\</span\>  
        \<span class="badge badge-cyan"\>NAMUR NE107\</span\>  
      \</div\>  
    \</header\>

    \<div class="nav-tabs"\>  
      \<button class="tab-btn active" data-tab="tab-dissector"\>Packet Dissector\</button\>  
      \<button class="tab-btn" data-tab="tab-command-studio"\>PV \&amp; Command Studio (Cmd 0/1/3)\</button\>  
      \<button class="tab-btn" data-tab="tab-session-mgr"\>HART-IP Session Manager\</button\>  
      \<button class="tab-btn" data-tab="tab-auditor"\>IEC 62443 \&amp; NE107 Auditor\</button\>  
      \<button class="tab-btn" data-tab="tab-config"\>Process Firewall \&amp; IDS Rules\</button\>  
    \</div\>

    \<\!-- TAB 1: DISSECTOR \--\>  
    \<div id="tab-dissector" class="tab-panel active"\>  
      \<div class="grid-2"\>  
        \<div class="card"\>  
          \<div class="card-title"\>  
            \<span\>Raw Packet Input (HART-IP \+ Token-Passing PDU)\</span\>  
            \<select id="preset-selector" style="width: auto; padding: 4px 8px; font-size: 0.8rem;"\>  
              \<option value="cmd1\_req"\>Preset: Cmd 1 Read PV Request\</option\>  
              \<option value="cmd1\_resp"\>Preset: Cmd 1 PV Response (101.325 kPa)\</option\>  
              \<option value="cmd0\_req"\>Preset: Cmd 0 Read Unique ID\</option\>  
              \<option value="cmd48\_req"\>Preset: Cmd 48 NE107 Diagnostics Request\</option\>  
              \<option value="session\_init"\>Preset: HART-IP Session Initiate\</option\>  
            \</select\>  
          \</div\>  
          \<div class="form-group"\>  
            \<label\>Raw HART-IP Frame Hex Stream\</label\>  
            \<textarea id="hex-input" class="mono" rows="8" placeholder="Enter hex stream (e.g. 01 00 03 00 00 01 00 13 82 96 24 12 34 56 01 00 29)..."\>\</textarea\>  
          \</div\>  
          \<div class="btn-group"\>  
            \<button id="btn-dissect" class="btn"\>Dissect Frame\</button\>  
            \<button id="btn-export-pcap" class="btn btn-secondary"\>Export to PCAP\</button\>  
            \<button id="btn-clear-hex" class="btn btn-secondary"\>Clear\</button\>  
          \</div\>  
        \</div\>

        \<div class="card"\>  
          \<div class="card-title"\>Protocol Dissection Tree\</div\>  
          \<div id="dissection-output" class="tree-view"\>  
            \<p style="color: var(--text-muted);"\>Select a preset or enter raw hex and click "Dissect Frame".\</p\>  
          \</div\>  
        \</div\>  
      \</div\>  
    \</div\>

    \<\!-- TAB 2: COMMAND STUDIO \--\>  
    \<div id="tab-command-studio" class="tab-panel"\>  
      \<div class="grid-2"\>  
        \<div class="card"\>  
          \<div class="card-title"\>HART Variable \&amp; Command Synthesizer\</div\>  
          \<div class="form-group"\>  
            \<label\>Target Command\</label\>  
            \<select id="hart-cmd"\>  
              \<option value="1"\>Cmd 1 \- Read Primary Variable (PV)\</option\>  
              \<option value="0"\>Cmd 0 \- Read Unique Identifier\</option\>  
              \<option value="2"\>Cmd 2 \- Read Current \&amp; % Range\</option\>  
              \<option value="3"\>Cmd 3 \- Read Dynamic Variables (PV, SV, TV, QV)\</option\>  
              \<option value="48"\>Cmd 48 \- Read Additional Device Status (NAMUR NE107)\</option\>  
            \</select\>  
          \</div\>  
          \<div class="form-group"\>  
            \<label\>Frame Type / Addressing Mode\</label\>  
            \<select id="hart-addr-mode"\>  
              \<option value="long"\>Long Frame (5-Byte Unique Device ID \- HART 5/6/7)\</option\>  
              \<option value="short"\>Short Frame (1-Byte Polling Address 0-15)\</option\>  
            \</select\>  
          \</div\>  
          \<div class="form-group" id="group-short-addr" style="display: none;"\>  
            \<label\>Short Polling Address (0 to 63)\</label\>  
            \<input type="number" id="hart-poll-addr" value="0" min="0" max="63"\>  
          \</div\>  
          \<div class="form-group" id="group-long-addr"\>  
            \<label\>Manufacturer \&amp; Device Identity\</label\>  
            \<select id="hart-mfg"\>  
              \<option value="0x16"\>0x16 \- Emerson / Rosemount\</option\>  
              \<option value="0x11"\>0x11 \- Endress+Hauser\</option\>  
              \<option value="0x32"\>0x32 \- Yokogawa\</option\>  
              \<option value="0x26"\>0x26 \- Siemens\</option\>  
              \<option value="0x22"\>0x22 \- Honeywell\</option\>  
            \</select\>  
          \</div\>  
          \<div class="form-group"\>  
            \<label\>Simulated Measurement PV (for Response Synthesis)\</label\>  
            \<div style="display:flex; gap:10px;"\>  
              \<input type="number" step="0.001" id="hart-pv-val" value="101.325" style="flex:1;"\>  
              \<select id="hart-unit" style="width:140px;"\>  
                \<option value="12"\>kPa (Pressure)\</option\>  
                \<option value="6"\>psi (Pressure)\</option\>  
                \<option value="7"\>bar (Pressure)\</option\>  
                \<option value="32"\>°C (Temp)\</option\>  
                \<option value="33"\>°F (Temp)\</option\>  
                \<option value="24"\>L/min (Flow)\</option\>  
                \<option value="65"\>mA (Current)\</option\>  
              \</select\>  
            \</div\>  
          \</div\>  
          \<div class="btn-group"\>  
            \<button id="btn-build-req" class="btn"\>Build Request Packet\</button\>  
            \<button id="btn-build-resp" class="btn btn-secondary"\>Synthesize Response Frame\</button\>  
          \</div\>  
        \</div\>

        \<div class="card"\>  
          \<div class="card-title"\>Synthesized Wire Representation\</div\>  
          \<label\>Wire Sequence (HART-IP Header \+ Token-Passing PDU)\</label\>  
          \<div id="cmd-hex-output" class="hex-view"\>Click "Build Request Packet" to generate.\</div\>  
          \<div style="margin-top: 16px;"\>  
            \<label\>PDU Architecture \&amp; Checksum (LRC)\</label\>  
            \<div id="cmd-breakdown" class="tree-view" style="max-height: 160px;"\>-\</div\>  
          \</div\>  
        \</div\>  
      \</div\>  
    \</div\>

    \<\!-- TAB 3: SESSION MANAGER \--\>  
    \<div id="tab-session-mgr" class="tab-panel"\>  
      \<div class="grid-2"\>  
        \<div class="card"\>  
          \<div class="card-title"\>HART-IP Client Session Lifecycle\</div\>  
          \<p style="color: var(--text-muted); font-size: 0.85rem; margin-bottom: 16px;"\>  
            HART-IP establishes logical management sessions over UDP or TCP port 5094\.  
          \</p\>  
          \<div class="form-group"\>  
            \<label\>Session Inactivity Timeout (ms)\</label\>  
            \<input type="number" id="session-timeout" value="60000" min="1000"\>  
          \</div\>  
          \<div class="form-group"\>  
            \<label\>Transaction Sequence Number\</label\>  
            \<input type="number" id="session-seq" value="1" min="1"\>  
          \</div\>  
          \<div class="btn-group"\>  
            \<button class="btn" id="btn-session-init"\>Build Session Initiate (0x00)\</button\>  
            \<button class="btn btn-secondary" id="btn-session-close"\>Build Session Close (0x01)\</button\>  
          \</div\>  
        \</div\>

        \<div class="card"\>  
          \<div class="card-title"\>Session Frame Stream\</div\>  
          \<div id="session-hex-output" class="hex-view"\>-\</div\>  
          \<div style="margin-top: 16px;"\>  
            \<div id="session-breakdown" class="tree-view" style="max-height: 180px;"\>-\</div\>  
          \</div\>  
        \</div\>  
      \</div\>  
    \</div\>

    \<\!-- TAB 4: AUDITOR \--\>  
    \<div id="tab-auditor" class="tab-panel"\>  
      \<div class="card"\>  
        \<div class="card-title"\>  
          \<span\>IEC 62443 \&amp; NAMUR NE107 Cybersecurity Audit\</span\>  
          \<button id="btn-run-audit" class="btn" style="padding: 6px 14px;"\>Run Security Audit\</button\>  
        \</div\>  
        \<div class="score-display"\>  
          \<div\>  
            \<div class="score-num" id="audit-score"\>72\</div\>  
            \<div style="font-size: 0.8rem; color: var(--text-muted);"\>COMPLIANCE SCORE\</div\>  
          \</div\>  
          \<div\>  
            \<div style="font-size: 1.2rem; font-weight: 700;" id="audit-posture"\>MODERATE POSTURE\</div\>  
            \<p style="color: var(--text-muted); font-size: 0.85rem;" id="audit-summary"\>7 of 10 process cybersecurity controls currently pass IEC 62443 criteria.\</p\>  
          \</div\>  
        \</div\>

        \<div style="overflow-x: auto;"\>  
          \<table class="audit-table"\>  
            \<thead\>  
              \<tr\>  
                \<th\>Rule ID\</th\>  
                \<th\>Security Control Title\</th\>  
                \<th\>Standard\</th\>  
                \<th\>Severity\</th\>  
                \<th\>Status\</th\>  
                \<th\>Remediation Strategy\</th\>  
              \</tr\>  
            \</thead\>  
            \<tbody id="audit-tbody"\>  
              \<\!-- Rendered via JS \--\>  
            \</tbody\>  
          \</table\>  
        \</div\>  
      \</div\>  
    \</div\>

    \<\!-- TAB 5: CONFIG SYNTHESIZER \--\>  
    \<div id="tab-config" class="tab-panel"\>  
      \<div class="grid-2"\>  
        \<div class="card"\>  
          \<div class="card-title"\>Linux nftables Process Conduit Firewall\</div\>  
          \<p style="color: var(--text-muted); font-size: 0.85rem; margin-bottom: 12px;"\>Purdue Level 2/3 Process conduit protecting HART-IP gateways (UDP/TCP 5094).\</p\>  
          \<div id="nftables-output" class="hex-view" style="max-height: 380px;"\>\</div\>  
        \</div\>  
        \<div class="card"\>  
          \<div class="card-title"\>Suricata L7 Industrial IDS Signatures\</div\>  
          \<p style="color: var(--text-muted); font-size: 0.85rem; margin-bottom: 12px;"\>Signatures detecting Cmd 35 re-span, Cmd 45 DAC trim, and Cmd 42 resets.\</p\>  
          \<div id="suricata-output" class="hex-view" style="max-height: 380px;"\>\</div\>  
        \</div\>  
      \</div\>  
    \</div\>

  \</div\>

  \<script src="hart-protocol-engine.js"\>\</script\>  
  \<script src="hart-security-auditor.js"\>\</script\>  
  \<script src="hart-config-synthesizer.js"\>\</script\>  
  \<script src="app.js"\>\</script\>  
\</body\>  
\</html\>  
