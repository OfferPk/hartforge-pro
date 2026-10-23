/\*\*  
 \* HARTForge Pro \- IEC 62443 & NAMUR NE107 Process Instrumentation Security Auditor  
 \* Standards: IEC 62443-3-3, IEC 62443-4-2, NAMUR NE107, FieldComm Group Security Guides  
 \* Author: Autonomous Digital Product Factory (Product \#169)  
 \* Version: 1.0.0  
 \*/

class HARTSecurityAuditor {  
  constructor() {  
    this.RULES \= \[  
      {  
        id: 'HART-SEC-01',  
        title: 'Unencrypted HART-IP (UDP/TCP Port 5094\) Exposure',  
        standard: 'IEC 62443-4-2 CR 4.3 (Data Integrity & Confidentiality)',  
        severity: 'HIGH',  
        weight: 15,  
        description: 'Standard HART-IP transmits measurement telemetry and transmitter configuration in cleartext over UDP/TCP 5094 without DTLS/TLS encryption.',  
        remediation: 'Deploy HART-IP v2 with DTLS (RFC 6347\) or isolate process gateways behind IPsec/MACsec industrial conduits.'  
      },  
      {  
        id: 'HART-SEC-02',  
        title: 'Unauthenticated Calibration & Range Writes (Cmd 35 / 45)',  
        standard: 'IEC 62443-3-3 SR 1.1 (Human User Identification & Authentication)',  
        severity: 'CRITICAL',  
        weight: 20,  
        description: 'Common practice commands (Cmd 35 Write Range, Cmd 45 Trim DAC) permit unauthorized hosts to alter transmitter span and zero, risking process overpressure/overfills.',  
        remediation: 'Restrict calibration commands using gateway access lists and lock the physical write-protect jumper on field transmitters.'  
      },  
      {  
        id: 'HART-SEC-03',  
        title: 'Physical Transmitter Write-Protect Jumper State',  
        standard: 'IEC 62443-4-2 CR 1.7 (Physical Protection & Tamper Resistance)',  
        severity: 'HIGH',  
        weight: 15,  
        description: 'Field devices without hardware write-protect jumpers enabled are vulnerable to remote over-the-network reconfiguration.',  
        remediation: 'Physically position the transmitter PCB write-protect switch to ON and audit Response Code bit 7 (In Write Protect Mode).'  
      },  
      {  
        id: 'HART-SEC-04',  
        title: 'WirelessHART Mesh Gateway Joining Security',  
        standard: 'IEC 62443-3-3 SR 3.1 (Communication Integrity) / ISA100',  
        severity: 'CRITICAL',  
        weight: 20,  
        description: 'Using common or default WirelessHART Join Keys (e.g. 0x000000... or factory defaults) allows rogue mesh sensor injection into plant telemetry.',  
        remediation: 'Configure unique, rotated 128-bit AES Join Keys per transmitter and enforce white-listed EUI-64 MAC addresses on the WirelessHART gateway.'  
      },  
      {  
        id: 'HART-SEC-05',  
        title: 'Direct IT-to-OT Process Subnet Conduit Violation',  
        standard: 'IEC 62443-3-2 (Zoning & Conduit Specification)',  
        severity: 'MEDIUM',  
        weight: 10,  
        description: 'Routing Port 5094 directly across corporate networks without an intermediate Level 2/3 industrial DMZ exposes critical loop controls to enterprise ransomware.',  
        remediation: 'Enforce strict Purdue Model zoning; terminate HART-IP connections in the OT DMZ and use read-only OPC UA wrappers for IT historians.'  
      },  
      {  
        id: 'HART-SEC-06',  
        title: 'Suppression of NAMUR NE107 Diagnostic Telemetry',  
        standard: 'NAMUR NE107 / IEC 62443-4-2 CR 6.1 (Audit Log Generation)',  
        severity: 'MEDIUM',  
        weight: 5,  
        description: 'Process telemetry ignores Cmd 48 and Field Device Status bits (Malfunction, Cold Start, Config Changed), blinding operators to hardware drift.',  
        remediation: 'Configure SCADA / asset management systems to poll Command 48 and generate high-priority operator alarms on NE107 status changes.'  
      },  
      {  
        id: 'HART-SEC-07',  
        title: 'Absence of Layer 7 HART-IP Deep Packet Inspection (OT IDS)',  
        standard: 'IEC 62443-4-2 CR 6.2 (Continuous Monitoring & IDS)',  
        severity: 'MEDIUM',  
        weight: 5,  
        description: 'Network perimeter lacks L7 protocol parsers to detect unauthorized write commands or burst spoofing on UDP port 5094.',  
        remediation: 'Deploy Suricata or Zeek sensors with dedicated HART-IP signatures monitoring for Command 35, 42, 45, and 108.'  
      },  
      {  
        id: 'HART-SEC-08',  
        title: 'Session Inactivity & Rate Limiting Enforcement',  
        standard: 'IEC 62443-3-3 SR 7.1 (Denial of Service Protection)',  
        severity: 'LOW',  
        weight: 4,  
        description: 'HART-IP gateways failing to terminate inactive sessions or throttle connection rates are susceptible to denial-of-service socket exhaustion.',  
        remediation: 'Enforce a maximum 60-second session inactivity timeout and apply iptables/nftables connection rate limits on port 5094.'  
      },  
      {  
        id: 'HART-SEC-09',  
        title: 'Device Description (DD/EDDL) Integrity & Signing',  
        standard: 'IEC 62443-4-2 CR 1.2 (Software & Firmware Integrity)',  
        severity: 'LOW',  
        weight: 3,  
        description: 'Loading unsigned or unverified Device Description files (EDDL/FDI) into plant asset management stations risks workstation exploit execution.',  
        remediation: 'Only install FieldComm Group digitally signed EDD/FDI device packages verified against official manufacturer checksums.'  
      },  
      {  
        id: 'HART-SEC-10',  
        title: 'Static Default Polling Address 0 Usage',  
        standard: 'IEC 62443-4-2 CR 1.1 (Device Identification)',  
        severity: 'LOW',  
        weight: 3,  
        description: 'Relying solely on short 1-byte polling address 0 without unique 5-byte long identifier verification increases risk of accidental cross-transmitter misdirection.',  
        remediation: 'Enforce 5-byte Long Frame addressing (Cmd 0 Unique ID) for all critical control loops and multi-drop field installations.'  
      }  
    \];  
  }

  auditConfiguration(config \= {}) {  
    const findings \= \[\];  
    let deduction \= 0;

    // 1\. Encryption  
    if (\!config.hasDtlsOrVpn) {  
      findings.push({ ...this.RULES\[0\], status: 'FAIL', current: 'Cleartext HART-IP over UDP/TCP 5094' });  
      deduction \+= this.RULES\[0\].weight;  
    } else {  
      findings.push({ ...this.RULES\[0\], status: 'PASS', current: 'HART-IP v2 DTLS / Encrypted Conduit Active' });  
    }

    // 2\. Calibration Authorization  
    if (\!config.hasCalibrationProtection) {  
      findings.push({ ...this.RULES\[1\], status: 'FAIL', current: 'Unrestricted Calibration Writes (Cmd 35/45) Allowed' });  
      deduction \+= this.RULES\[1\].weight;  
    } else {  
      findings.push({ ...this.RULES\[1\], status: 'PASS', current: 'Gateway Allowlist & Access Controls Enforced' });  
    }

    // 3\. Hardware Jumper  
    if (\!config.isWriteProtectJumperSet) {  
      findings.push({ ...this.RULES\[2\], status: 'FAIL', current: 'Transmitter Write-Protect Jumper Disabled' });  
      deduction \+= this.RULES\[2\].weight;  
    } else {  
      findings.push({ ...this.RULES\[2\], status: 'PASS', current: 'Hardware Write-Protect Switch Locked ON' });  
    }

    // 4\. WirelessHART Join Key  
    if (config.hasDefaultJoinKey) {  
      findings.push({ ...this.RULES\[3\], status: 'FAIL', current: 'Default / Common Join Key in Use on Mesh' });  
      deduction \+= this.RULES\[3\].weight;  
    } else {  
      findings.push({ ...this.RULES\[3\], status: 'PASS', current: 'Rotated 128-bit AES Join Key Enforced' });  
    }

    // 5\. Zoning  
    if (\!config.isZoned) {  
      findings.push({ ...this.RULES\[4\], status: 'FAIL', current: 'Flat IT/OT Network without Level 2/3 Process DMZ' });  
      deduction \+= this.RULES\[4\].weight;  
    } else {  
      findings.push({ ...this.RULES\[4\], status: 'PASS', current: 'Purdue Model Level 2 Process Zone Isolated' });  
    }

    // 6\. NAMUR NE107  
    if (\!config.isNe107Monitored) {  
      findings.push({ ...this.RULES\[5\], status: 'FAIL', current: 'NE107 Diagnostics & Cmd 48 Not Logged in SCADA' });  
      deduction \+= this.RULES\[5\].weight;  
    } else {  
      findings.push({ ...this.RULES\[5\], status: 'PASS', current: 'Automated NE107 Status Polling & Alarming Active' });  
    }

    // 7\. OT IDS  
    if (\!config.hasL7Ids) {  
      findings.push({ ...this.RULES\[6\], status: 'FAIL', current: 'No Suricata/Zeek HART-IP L7 Signatures Deployed' });  
      deduction \+= this.RULES\[6\].weight;  
    } else {  
      findings.push({ ...this.RULES\[6\], status: 'PASS', current: 'Suricata L7 Rules Inspecting Port 5094 Active' });  
    }

    // 8\. Session Limiting  
    if (\!config.hasSessionLimits) {  
      findings.push({ ...this.RULES\[7\], status: 'FAIL', current: 'No Timeout or Rate Limits on HART-IP Gateway' });  
      deduction \+= this.RULES\[7\].weight;  
    } else {  
      findings.push({ ...this.RULES\[7\], status: 'PASS', current: '60s Timeout & Ingress Rate Limiting Enforced' });  
    }

    // 9\. EDDL Signing  
    if (\!config.hasSignedEddl) {  
      findings.push({ ...this.RULES\[8\], status: 'FAIL', current: 'Unsigned / Unverified EDDL Files in Use' });  
      deduction \+= this.RULES\[8\].weight;  
    } else {  
      findings.push({ ...this.RULES\[8\], status: 'PASS', current: 'Digitally Signed FieldComm Group Packages Only' });  
    }

    // 10\. Long Address  
    if (config.usesDefaultPollingZero) {  
      findings.push({ ...this.RULES\[9\], status: 'FAIL', current: 'Relying on Short Address 0 without Unique ID' });  
      deduction \+= this.RULES\[9\].weight;  
    } else {  
      findings.push({ ...this.RULES\[9\], status: 'PASS', current: '5-Byte Long Unique ID Verified for Loops' });  
    }

    const score \= Math.max(0, 100 \- deduction);  
    let posture \= 'POOR';  
    if (score \>= 90\) posture \= 'EXCELLENT';  
    else if (score \>= 75\) posture \= 'GOOD';  
    else if (score \>= 50\) posture \= 'MODERATE';

    return {  
      score,  
      posture,  
      findings,  
      timestamp: new Date().toISOString()  
    };  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= { HARTSecurityAuditor };  
}  
