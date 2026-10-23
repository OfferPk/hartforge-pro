/\*\*  
 \* HARTForge Pro \- Web Application Controller  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  const engine \= new HARTEngine();  
  const auditor \= new HARTSecurityAuditor();  
  const synthesizer \= new HARTConfigSynthesizer();

  // Presets  
  const PRESETS \= {  
    cmd1\_req: engine.buildCmd1ReadPv().hexString,  
    cmd1\_resp: engine.buildCmd1Response(101.325, 12).hexString,  
    cmd0\_req: engine.buildCmd0ReadUniqueId(0).hexString,  
    cmd48\_req: engine.buildCmd48ReadAdditionalStatus().hexString,  
    session\_init: engine.buildSessionInit().hexString  
  };

  // Tab switching  
  const tabBtns \= document.querySelectorAll('.tab-btn');  
  const tabPanels \= document.querySelectorAll('.tab-panel');

  tabBtns.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      tabBtns.forEach(b \=\> b.classList.remove('active'));  
      tabPanels.forEach(p \=\> p.classList.remove('active'));  
      btn.classList.add('active');  
      const target \= document.getElementById(btn.getAttribute('data-tab'));  
      if (target) target.classList.add('active');  
    });  
  });

  // Preset selector  
  const presetSelector \= document.getElementById('preset-selector');  
  const hexInput \= document.getElementById('hex-input');

  presetSelector.addEventListener('change', () \=\> {  
    const val \= presetSelector.value;  
    if (PRESETS\[val\]) {  
      hexInput.value \= PRESETS\[val\];  
      runDissection();  
    }  
  });

  // Dissector button  
  document.getElementById('btn-dissect').addEventListener('click', runDissection);  
  document.getElementById('btn-clear-hex').addEventListener('click', () \=\> {  
    hexInput.value \= '';  
    document.getElementById('dissection-output').innerHTML \= '\<p style="color: var(--text-muted);"\>Cleared.\</p\>';  
  });

  function runDissection() {  
    const hex \= hexInput.value.trim();  
    if (\!hex) return;  
    const output \= document.getElementById('dissection-output');  
    try {  
      const bytes \= engine.hexToBytes(hex);  
      const res \= engine.dissect(bytes);

      let html \= \`  
        \<div class="tree-node"\>  
          \<span class="tree-key"\>HART-IP Protocol Header (IEC 62591):\</span\>\<br\>  
          \&nbsp;\&nbsp;Version: \<span class="tree-val"\>\${res.hartIp.version}\</span\> \| MsgType: \<span class="tree-val"\>\${res.hartIp.msgType}\</span\>\<br\>  
          \&nbsp;\&nbsp;Message ID: \<span class="tree-val" style="color:var(--accent-hart); font-weight:700;"\>\${res.hartIp.msgId}\</span\>\<br\>  
          \&nbsp;\&nbsp;Status: \<span class="tree-val"\>\${res.hartIp.status}\</span\> \| Seq: \<span class="tree-val"\>\${res.hartIp.sequenceNumber}\</span\> \| ByteCount: \<span class="tree-val"\>\${res.hartIp.byteCount} B\</span\>  
        \</div\>  
      \`;

      if (res.hartPdu) {  
        html \+= \`  
          \<div class="tree-node"\>  
            \<span class="tree-key"\>HART Token-Passing PDU:\</span\>\<br\>  
            \&nbsp;\&nbsp;Delimiter: \<span class="tree-val"\>\${res.hartPdu.delimiter}\</span\>\<br\>  
            \&nbsp;\&nbsp;Address: \<span class="tree-val" style="color:\#06b6d4; font-weight:600;"\>\${res.hartPdu.address}\</span\>\<br\>  
            \&nbsp;\&nbsp;Command: \<span class="tree-val" style="color:\#10b981; font-weight:700;"\>\${res.hartPdu.command}\</span\>\<br\>  
            \&nbsp;\&nbsp;Data Bytes: \<span class="tree-val"\>\${res.hartPdu.dataByteCount} B\</span\> \| LRC Checksum: \<span class="tree-val" style="color:\${res.hartPdu.lrcValid ? '\#10b981' : '\#f43f5e'}; font-weight:700;"\>\${res.hartPdu.lrc}\</span\>  
          \</div\>  
        \`;

        if (res.hartPdu.responseCodes) {  
          html \+= \`  
            \<div class="tree-node"\>  
              \<span class="tree-key"\>Response Status \&amp; NAMUR Flags:\</span\>\<br\>  
              \&nbsp;\&nbsp;Comm Status: \<span class="tree-val"\>\${res.hartPdu.responseCodes.commStatus}\</span\>\<br\>  
              \&nbsp;\&nbsp;Device Malfunction: \<span class="tree-val" style="color:\${res.hartPdu.responseCodes.deviceStatus.malfunction ? '\#f43f5e' : '\#10b981'}; font-weight:600;"\>\${res.hartPdu.responseCodes.deviceStatus.malfunction ? 'CRITICAL \- MALFUNCTION' : 'Normal'}\</span\>\<br\>  
              \&nbsp;\&nbsp;Cold Start: \<span class="tree-val"\>\${res.hartPdu.responseCodes.deviceStatus.coldStart ? 'Yes (Restarted)' : 'No'}\</span\> \| Config Changed: \<span class="tree-val"\>\${res.hartPdu.responseCodes.deviceStatus.configChanged ? 'Yes' : 'No'}\</span\>  
            \</div\>  
          \`;  
        }

        if (res.hartPdu.measurement) {  
          html \+= \`  
            \<div class="tree-node"\>  
              \<span class="tree-key"\>Decoded Process Measurement:\</span\>\<br\>  
              \&nbsp;\&nbsp;Primary Variable (PV): \<span class="tree-val" style="font-size:1.1rem; color:\#f59e0b; font-weight:800;"\>\${res.hartPdu.measurement.primaryVariable}\</span\>  
            \</div\>  
          \`;  
        }  
      }

      output.innerHTML \= html;  
    } catch (e) {  
      output.innerHTML \= \`\<p style="color: var(--accent-rose); font-weight:600;"\>Dissection Error: \${e.message}\</p\>\`;  
    }  
  }

  // Initial load  
  hexInput.value \= PRESETS.cmd1\_req;  
  runDissection();

  // Addressing mode toggling  
  const addrMode \= document.getElementById('hart-addr-mode');  
  const groupShort \= document.getElementById('group-short-addr');  
  const groupLong \= document.getElementById('group-long-addr');  
  addrMode.addEventListener('change', () \=\> {  
    const isShort \= (addrMode.value \=== 'short');  
    groupShort.style.display \= isShort ? 'block' : 'none';  
    groupLong.style.display \= isShort ? 'none' : 'block';  
  });

  // Build Request Packet  
  document.getElementById('btn-build-req').addEventListener('click', () \=\> {  
    const cmd \= parseInt(document.getElementById('hart-cmd').value, 10);  
    const isShort \= (addrMode.value \=== 'short');  
    const pollAddr \= parseInt(document.getElementById('hart-poll-addr').value, 10\) \|\| 0;  
    const mfgCode \= parseInt(document.getElementById('hart-mfg').value, 16);

    let pkt;  
    if (cmd \=== 0\) {  
      pkt \= engine.buildCmd0ReadUniqueId(pollAddr);  
    } else if (cmd \=== 1\) {  
      pkt \= engine.buildCmd1ReadPv(isShort ? null : \[0x80 \| mfgCode, 0x24, 0x12, 0x34, 0x56\]);  
    } else if (cmd \=== 3\) {  
      pkt \= engine.buildCmd3ReadDynamicVars(isShort ? null : \[0x80 \| mfgCode, 0x24, 0x12, 0x34, 0x56\]);  
    } else if (cmd \=== 48\) {  
      pkt \= engine.buildCmd48ReadAdditionalStatus(isShort ? null : \[0x80 \| mfgCode, 0x24, 0x12, 0x34, 0x56\]);  
    } else {  
      pkt \= engine.buildCmd1ReadPv();  
    }

    document.getElementById('cmd-hex-output').innerText \= pkt.hexString;  
    document.getElementById('cmd-breakdown').innerHTML \= \`  
      \<div\>\<strong\>Packet Length:\</strong\> \${pkt.rawBytes.length} bytes\</div\>  
      \<div\>\<strong\>Target Command:\</strong\> Command \${cmd}\</div\>  
      \<div\>\<strong\>Transport:\</strong\> HART-IP (UDP Port 5094\) Token-Passing Request\</div\>  
    \`;  
  });

  // Synthesize Response Frame  
  document.getElementById('btn-build-resp').addEventListener('click', () \=\> {  
    const pvVal \= parseFloat(document.getElementById('hart-pv-val').value) \|\| 101.325;  
    const unitCode \= parseInt(document.getElementById('hart-unit').value, 10\) \|\| 12;

    const pkt \= engine.buildCmd1Response(pvVal, unitCode);  
    document.getElementById('cmd-hex-output').innerText \= pkt.hexString;  
    document.getElementById('cmd-breakdown').innerHTML \= \`  
      \<div\>\<strong\>Packet Length:\</strong\> \${pkt.rawBytes.length} bytes\</div\>  
      \<div\>\<strong\>Synthesized Measurement:\</strong\> \${pvVal} (Unit Code: \${unitCode})\</div\>  
      \<div\>\<strong\>Response Status:\</strong\> 0x00 0x00 (Success, Normal Health)\</div\>  
    \`;  
  });

  // Session Manager  
  document.getElementById('btn-session-init').addEventListener('click', () \=\> {  
    const timeout \= parseInt(document.getElementById('session-timeout').value, 10\) \|\| 60000;  
    const seq \= parseInt(document.getElementById('session-seq').value, 10\) \|\| 1;  
    const payload \= new Uint8Array(\[0x01, (timeout \>\> 24\) & 0xFF, (timeout \>\> 16\) & 0xFF, (timeout \>\> 8\) & 0xFF, timeout & 0xFF\]);  
    const pkt \= engine.wrapHartIp(payload, engine.MSG\_TYPES.REQUEST, engine.MSG\_IDS.SESSION\_INIT, seq);

    document.getElementById('session-hex-output').innerText \= engine.bytesToHex(pkt);  
    document.getElementById('session-breakdown').innerHTML \= \`  
      \<div\>\<strong\>Service:\</strong\> Session Initiate (MsgId 0x00)\</div\>  
      \<div\>\<strong\>Master Type:\</strong\> Primary Master (0x01)\</div\>  
      \<div\>\<strong\>Inactivity Timeout:\</strong\> \${timeout} ms\</div\>  
    \`;  
  });

  document.getElementById('btn-session-close').addEventListener('click', () \=\> {  
    const seq \= parseInt(document.getElementById('session-seq').value, 10\) \|\| 2;  
    const pkt \= engine.wrapHartIp(new Uint8Array(0), engine.MSG\_TYPES.REQUEST, engine.MSG\_IDS.SESSION\_CLOSE, seq);

    document.getElementById('session-hex-output').innerText \= engine.bytesToHex(pkt);  
    document.getElementById('session-breakdown').innerHTML \= \`  
      \<div\>\<strong\>Service:\</strong\> Session Close (MsgId 0x01)\</div\>  
      \<div\>\<strong\>Status:\</strong\> Graceful socket teardown\</div\>  
    \`;  
  });

  // Export PCAP  
  document.getElementById('btn-export-pcap').addEventListener('click', () \=\> {  
    const hex \= hexInput.value.trim();  
    if (\!hex) return;  
    const bytes \= engine.hexToBytes(hex);  
    const pcapBytes \= engine.exportPcap(\[bytes\]);  
    downloadBlob(pcapBytes, 'hartip-capture.pcap');  
  });

  function downloadBlob(bytes, filename) {  
    const blob \= new Blob(\[bytes\], { type: 'application/vnd.tcpdump.pcap' });  
    const url \= URL.createObjectURL(blob);  
    const a \= document.createElement('a');  
    a.href \= url;  
    a.download \= filename;  
    a.click();  
    URL.revokeObjectURL(url);  
  }

  // Security Auditor  
  function runAudit() {  
    const sampleConfig \= {  
      hasDtlsOrVpn: false,  
      hasCalibrationProtection: false,  
      isWriteProtectJumperSet: false,  
      hasDefaultJoinKey: false,  
      isZoned: true,  
      isNe107Monitored: true,  
      hasL7Ids: true,  
      hasSessionLimits: true,  
      hasSignedEddl: false,  
      usesDefaultPollingZero: true  
    };

    const res \= auditor.auditConfiguration(sampleConfig);  
    document.getElementById('audit-score').innerText \= res.score;  
    document.getElementById('audit-posture').innerText \= \`\${res.posture} POSTURE\`;

    const tbody \= document.getElementById('audit-tbody');  
    tbody.innerHTML \= res.findings.map(f \=\> \`  
      \<tr\>  
        \<td\>\<code\>\${f.id}\</code\>\</td\>  
        \<td\>\<strong\>\${f.title}\</strong\>\</td\>  
        \<td\>\<small\>\${f.standard}\</small\>\</td\>  
        \<td\>\<span class="badge \${f.severity \=== 'CRITICAL' ? 'badge-rose' : f.severity \=== 'HIGH' ? 'badge-hart' : 'badge-cyan'}"\>\${f.severity}\</span\>\</td\>  
        \<td\>\<span class="badge \${f.status \=== 'PASS' ? 'badge-emerald' : 'badge-rose'}"\>\${f.status}\</span\>\</td\>  
        \<td\>\<small\>\${f.remediation}\</small\>\</td\>  
      \</tr\>  
    \`).join('');  
  }

  document.getElementById('btn-run-audit').addEventListener('click', runAudit);  
  runAudit();

  // Config Synthesizer  
  document.getElementById('nftables-output').innerText \= synthesizer.generateNftables();  
  document.getElementById('suricata-output').innerText \= synthesizer.generateSuricataRules();  
});  
