/\*\*  
 \* HARTForge Pro \- Industrial HART-IP (IEC 62591 / FieldComm Group) & WirelessHART Protocol Engine  
 \* Standards: IEC 62591, HART Field Communication Protocol Spec (HCF\_SPEC-054 / 065 / 099\)  
 \* Author: Autonomous Digital Product Factory (Product \#169)  
 \* Version: 1.0.0  
 \*/

class HARTEngine {  
  constructor() {  
    this.HART\_IP\_PORT \= 5094;  
    this.HART\_IP\_VERSION\_1 \= 0x01;  
    this.HART\_IP\_VERSION\_2 \= 0x02;

    this.MSG\_TYPES \= {  
      REQUEST: 0x00,  
      RESPONSE: 0x01,  
      PUBLISH: 0x02,  
      NAK: 0x03  
    };

    this.MSG\_IDS \= {  
      SESSION\_INIT: 0x00,  
      SESSION\_CLOSE: 0x01,  
      KEEP\_ALIVE: 0x02,  
      TOKEN\_PASSING: 0x03,  
      DIRECT\_PDU: 0x04  
    };

    this.DELIMITERS \= {  
      SHORT\_MASTER\_REQ: 0x02,  
      SHORT\_SLAVE\_RESP: 0x06,  
      SHORT\_BURST: 0x01,  
      LONG\_MASTER\_REQ: 0x82,  
      LONG\_SLAVE\_RESP: 0x86,  
      LONG\_BURST: 0x81  
    };

    this.COMMANDS \= {  
      CMD\_00\_READ\_UNIQUE\_ID: 0,  
      CMD\_01\_READ\_PV: 1,  
      CMD\_02\_READ\_CURRENT\_PERCENT: 2,  
      CMD\_03\_READ\_DYNAMIC\_VARS: 3,  
      CMD\_06\_WRITE\_POLLING\_ADDR: 6,  
      CMD\_09\_READ\_DEVICE\_VARS: 9,  
      CMD\_33\_READ\_SLOT\_VARS: 33,  
      CMD\_48\_READ\_ADDITIONAL\_STATUS: 48  
    };

    this.UNITS \= {  
      6: { code: 6, name: 'psi', category: 'Pressure' },  
      7: { code: 7, name: 'bar', category: 'Pressure' },  
      8: { code: 8, name: 'mbar', category: 'Pressure' },  
      12: { code: 12, name: 'kPa', category: 'Pressure' },  
      13: { code: 13, name: 'MPa', category: 'Pressure' },  
      24: { code: 24, name: 'L/min', category: 'Flow' },  
      32: { code: 32, name: '°C', category: 'Temperature' },  
      33: { code: 33, name: '°F', category: 'Temperature' },  
      34: { code: 34, name: 'K', category: 'Temperature' },  
      57: { code: 57, name: '%', category: 'Percentage' },  
      65: { code: 65, name: 'mA', category: 'Current' }  
    };

    this.MANUFACTURERS \= {  
      0x11: 'Endress+Hauser',  
      0x16: 'Emerson / Rosemount',  
      0x22: 'Honeywell',  
      0x26: 'Siemens',  
      0x32: 'Yokogawa',  
      0x1A: 'ABB'  
    };  
  }

  // \--- Utility Methods \---  
  calcLrc(bytes) {  
    let lrc \= 0;  
    for (let i \= 0; i \< bytes.length; i++) {  
      lrc ^= bytes\[i\];  
    }  
    return lrc & 0xFF;  
  }

  floatToBytes(val) {  
    const buf \= new ArrayBuffer(4);  
    const view \= new DataView(buf);  
    view.setFloat32(0, val, false); // Big-Endian  
    return new Uint8Array(buf);  
  }

  bytesToFloat(b0, b1, b2, b3) {  
    const buf \= new ArrayBuffer(4);  
    const view \= new DataView(buf);  
    view.setUint8(0, b0);  
    view.setUint8(1, b1);  
    view.setUint8(2, b2);  
    view.setUint8(3, b3);  
    return view.getFloat32(0, false);  
  }

  hexToBytes(hexStr) {  
    const clean \= hexStr.replace(/\[\\s\\-\_:\\n\\r\]/g, '');  
    if (clean.length % 2 \!== 0\) throw new Error('Invalid hex string length');  
    const bytes \= new Uint8Array(clean.length / 2);  
    for (let i \= 0; i \< clean.length; i \+= 2\) {  
      bytes\[i / 2\] \= parseInt(clean.substr(i, 2), 16);  
    }  
    return bytes;  
  }

  bytesToHex(bytes) {  
    return Array.from(bytes).map(b \=\> b.toString(16).padStart(2, '0')).join(' ');  
  }

  // \--- HART Token-Passing PDU Builder \---  
  buildTokenPassingPdu(isLong, isMaster, address, command, dataBytes \= new Uint8Array(0)) {  
    let delimiter;  
    let addrBytes;

    if (\!isLong) {  
      delimiter \= isMaster ? this.DELIMITERS.SHORT\_MASTER\_REQ : this.DELIMITERS.SHORT\_SLAVE\_RESP;  
      const addrVal \= (address & 0x3F) \| (isMaster ? 0x80 : 0x00);  
      addrBytes \= \[addrVal\];  
    } else {  
      delimiter \= isMaster ? this.DELIMITERS.LONG\_MASTER\_REQ : this.DELIMITERS.LONG\_SLAVE\_RESP;  
      // Address is 5 bytes: \[Master/Burst/MfgId, DevType, DevId2, DevId1, DevId0\]  
      if (Array.isArray(address) \|\| address instanceof Uint8Array) {  
        addrBytes \= Array.from(address);  
        if (isMaster) addrBytes\[0\] \|= 0x80;  
      } else {  
        // Synthesize standard long address for mfg 0x16 (Emerson), devType 0x24, devId 0x123456  
        addrBytes \= \[0x80 \| 0x16, 0x24, 0x12, 0x34, 0x56\];  
      }  
    }

    const byteCount \= dataBytes.length;  
    const pduCore \= \[delimiter, ...addrBytes, command, byteCount, ...dataBytes\];  
    const lrc \= this.calcLrc(pduCore);  
    pduCore.push(lrc);

    return new Uint8Array(pduCore);  
  }

  // \--- HART-IP Packet Wrapper (8-Byte Header \+ PDU) \---  
  wrapHartIp(pduBytes, msgType \= 0x00, msgId \= 0x03, seqNum \= 1, status \= 0x00, version \= 1\) {  
    const totalLen \= 8 \+ pduBytes.length;  
    const header \= \[  
      version,  
      msgType,  
      msgId,  
      status,  
      (seqNum \>\> 8\) & 0xFF,  
      seqNum & 0xFF,  
      (totalLen \>\> 8\) & 0xFF,  
      totalLen & 0xFF  
    \];

    const packet \= new Uint8Array(totalLen);  
    packet.set(header, 0);  
    packet.set(pduBytes, 8);  
    return packet;  
  }

  // \--- Preconfigured HART Command Requests \---

  buildCmd0ReadUniqueId(pollingAddr \= 0, seq \= 1\) {  
    const pdu \= this.buildTokenPassingPdu(false, true, pollingAddr, this.COMMANDS.CMD\_00\_READ\_UNIQUE\_ID);  
    const pkt \= this.wrapHartIp(pdu, this.MSG\_TYPES.REQUEST, this.MSG\_IDS.TOKEN\_PASSING, seq);  
    return { rawBytes: pkt, hexString: this.bytesToHex(pkt), pduBytes: pdu };  
  }

  buildCmd1ReadPv(longAddr \= null, seq \= 2\) {  
    const pdu \= this.buildTokenPassingPdu(true, true, longAddr, this.COMMANDS.CMD\_01\_READ\_PV);  
    const pkt \= this.wrapHartIp(pdu, this.MSG\_TYPES.REQUEST, this.MSG\_IDS.TOKEN\_PASSING, seq);  
    return { rawBytes: pkt, hexString: this.bytesToHex(pkt), pduBytes: pdu };  
  }

  buildCmd3ReadDynamicVars(longAddr \= null, seq \= 3\) {  
    const pdu \= this.buildTokenPassingPdu(true, true, longAddr, this.COMMANDS.CMD\_03\_READ\_DYNAMIC\_VARS);  
    const pkt \= this.wrapHartIp(pdu, this.MSG\_TYPES.REQUEST, this.MSG\_IDS.TOKEN\_PASSING, seq);  
    return { rawBytes: pkt, hexString: this.bytesToHex(pkt), pduBytes: pdu };  
  }

  buildCmd48ReadAdditionalStatus(longAddr \= null, seq \= 4\) {  
    const pdu \= this.buildTokenPassingPdu(true, true, longAddr, this.COMMANDS.CMD\_48\_READ\_ADDITIONAL\_STATUS);  
    const pkt \= this.wrapHartIp(pdu, this.MSG\_TYPES.REQUEST, this.MSG\_IDS.TOKEN\_PASSING, seq);  
    return { rawBytes: pkt, hexString: this.bytesToHex(pkt), pduBytes: pdu };  
  }

  buildCmd1Response(pvValue \= 101.325, unitCode \= 12, seq \= 2\) { // Default 101.325 kPa  
    const floatBytes \= this.floatToBytes(pvValue);  
    const respData \= new Uint8Array(\[  
      0x00, // Comm status / Response code 0 (Success)  
      0x00, // Field device status  
      unitCode,  
      ...floatBytes  
    \]);  
    const pdu \= this.buildTokenPassingPdu(true, false, null, this.COMMANDS.CMD\_01\_READ\_PV, respData);  
    const pkt \= this.wrapHartIp(pdu, this.MSG\_TYPES.RESPONSE, this.MSG\_IDS.TOKEN\_PASSING, seq);  
    return { rawBytes: pkt, hexString: this.bytesToHex(pkt), pduBytes: pdu };  
  }

  buildSessionInit(seq \= 1\) {  
    // HART-IP Session Initiate payload: Master Type (1B), Inactivity Timeout in ms (4B)  
    const payload \= new Uint8Array(\[0x01, 0x00, 0x00, 0xEA, 0x60\]); // 60,000 ms timeout  
    const pkt \= this.wrapHartIp(payload, this.MSG\_TYPES.REQUEST, this.MSG\_IDS.SESSION\_INIT, seq);  
    return { rawBytes: pkt, hexString: this.bytesToHex(pkt) };  
  }

  // \--- Real-Time Dissector \---  
  dissect(bytes) {  
    if (bytes.length \< 8\) {  
      throw new Error('Frame too short for HART-IP Header (min 8 bytes)');  
    }

    const version \= bytes\[0\];  
    const msgType \= bytes\[1\];  
    const msgId \= bytes\[2\];  
    const status \= bytes\[3\];  
    const seq \= (bytes\[4\] \<\< 8\) \| bytes\[5\];  
    const byteCount \= (bytes\[6\] \<\< 8\) \| bytes\[7\];

    const msgTypeNames \= { 0: 'Request', 1: 'Response', 2: 'Publish / Burst', 3: 'NAK' };  
    const msgIdNames \= {  
      0: 'Session Initiate',  
      1: 'Session Close',  
      2: 'Keep-Alive',  
      3: 'Token-Passing PDU',  
      4: 'Direct PDU'  
    };

    const res \= {  
      hartIp: {  
        version,  
        msgType: \`\${msgTypeNames\[msgType\] \|\| 'Unknown'} (0x\${msgType.toString(16).padStart(2, '0')})\`,  
        msgId: \`\${msgIdNames\[msgId\] \|\| 'Unknown'} (0x\${msgId.toString(16).padStart(2, '0')})\`,  
        status: status \=== 0 ? 'Success (0x00)' : \`Error (0x\${status.toString(16).padStart(2, '0')})\`,  
        sequenceNumber: seq,  
        byteCount  
      }  
    };

    if (msgId \=== this.MSG\_IDS.TOKEN\_PASSING && bytes.length \> 8\) {  
      const pduBytes \= bytes.slice(8);  
      const delim \= pduBytes\[0\];  
      const isLong \= (delim & 0x80) \!== 0;  
      const addrLen \= isLong ? 5 : 1;

      if (pduBytes.length \< 1 \+ addrLen \+ 2\) {  
        throw new Error('PDU truncated: insufficient bytes for address and command');  
      }

      const addrSlice \= pduBytes.slice(1, 1 \+ addrLen);  
      let addressStr \= '';  
      let mfgName \= 'Generic';

      if (\!isLong) {  
        const pollAddr \= addrSlice\[0\] & 0x3F;  
        addressStr \= \`Polling Address: \${pollAddr} (Short)\`;  
      } else {  
        const mfgId \= addrSlice\[0\] & 0x3F;  
        mfgName \= this.MANUFACTURERS\[mfgId\] \|\| \`Mfg 0x\${mfgId.toString(16)}\`;  
        const devType \= addrSlice\[1\];  
        const devId \= (addrSlice\[2\] \<\< 16\) \| (addrSlice\[3\] \<\< 8\) \| addrSlice\[4\];  
        addressStr \= \`\${mfgName} (DevType 0x\${devType.toString(16)}, ID: \${devId})\`;  
      }

      const cmd \= pduBytes\[1 \+ addrLen\];  
      const pduByteCount \= pduBytes\[1 \+ addrLen \+ 1\];  
      const dataStart \= 1 \+ addrLen \+ 2;  
      const dataEnd \= dataStart \+ pduByteCount;  
      const dataPayload \= pduBytes.slice(dataStart, dataEnd);  
      const receivedLrc \= pduBytes\[dataEnd\];

      const expectedLrc \= this.calcLrc(pduBytes.slice(0, dataEnd));  
      const lrcValid \= (receivedLrc \=== expectedLrc);

      const cmdNames \= {  
        0: 'Read Unique Identifier',  
        1: 'Read Primary Variable (PV)',  
        2: 'Read Current & % Range',  
        3: 'Read Dynamic Variables',  
        6: 'Write Polling Address',  
        9: 'Read Device Variables with Status',  
        33: 'Read Slot Variables',  
        48: 'Read Additional Device Status (NAMUR NE107)'  
      };

      res.hartPdu \= {  
        delimiter: \`0x\${delim.toString(16).padStart(2, '0')} (\${isLong ? 'Long' : 'Short'} Frame)\`,  
        address: addressStr,  
        command: \`Cmd \${cmd} \- \${cmdNames\[cmd\] \|\| 'Vendor Specific'}\`,  
        dataByteCount: pduByteCount,  
        dataHex: this.bytesToHex(dataPayload),  
        lrc: \`0x\${receivedLrc?.toString(16).padStart(2, '0')} (\${lrcValid ? 'VALID' : 'CHECKSUM ERROR'})\`,  
        lrcValid  
      };

      // Interpret Response Data for Standard Commands  
      if (dataPayload.length \>= 2\) {  
        const firstResp \= dataPayload\[0\];  
        const devStatus \= dataPayload\[1\];

        res.hartPdu.responseCodes \= {  
          commStatus: firstResp \=== 0 ? 'Success (0)' : \`Error Code \${firstResp}\`,  
          deviceStatus: {  
            malfunction: Boolean(devStatus & 0x80),  
            configChanged: Boolean(devStatus & 0x40),  
            coldStart: Boolean(devStatus & 0x20),  
            moreStatusAvailable: Boolean(devStatus & 0x10),  
            currentFixed: Boolean(devStatus & 0x08),  
            currentSaturated: Boolean(devStatus & 0x04),  
            nonPvOutOfLimits: Boolean(devStatus & 0x02),  
            pvOutOfLimits: Boolean(devStatus & 0x01)  
          }  
        };

        // Cmd 1 (Read PV): Unit (1B) \+ Float (4B)  
        if (cmd \=== 1 && dataPayload.length \>= 7\) {  
          const unitCode \= dataPayload\[2\];  
          const pvVal \= this.bytesToFloat(dataPayload\[3\], dataPayload\[4\], dataPayload\[5\], dataPayload\[6\]);  
          const unitObj \= this.UNITS\[unitCode\] \|\| { name: \`Unit \${unitCode}\` };  
          res.hartPdu.measurement \= {  
            primaryVariable: \`\${pvVal.toFixed(3)} \${unitObj.name}\`,  
            rawValue: pvVal,  
            unit: unitObj.name  
          };  
        }  
      }  
    }

    return res;  
  }

  // \--- Wireshark Libpcap 2.4 Exporter (UDP Port 5094\) \---  
  exportPcap(packets) {  
    const PCAP\_MAGIC \= 0xA1B2C3D4;  
    const PCAP\_SNAPLEN \= 65535;  
    const LINKTYPE\_ETHERNET \= 1;

    let totalPcapSize \= 24; // Global header  
    for (const pkt of packets) {  
      const frameLen \= 14 \+ 20 \+ 8 \+ pkt.length; // Eth \+ IP \+ UDP \+ HART-IP  
      totalPcapSize \+= 16 \+ frameLen; // Packet header \+ frame  
    }

    const buffer \= new ArrayBuffer(totalPcapSize);  
    const view \= new DataView(buffer);  
    const bytes \= new Uint8Array(buffer);

    // Global Header (24 bytes Little-Endian)  
    view.setUint32(0, PCAP\_MAGIC, true);  
    view.setUint16(4, 2, true); // Major 2  
    view.setUint16(6, 4, true); // Minor 4  
    view.setInt32(8, 0, true);  
    view.setUint32(12, 0, true);  
    view.setUint32(16, PCAP\_SNAPLEN, true);  
    view.setUint32(20, LINKTYPE\_ETHERNET, true);

    let offset \= 24;  
    const baseTime \= Math.floor(Date.now() / 1000);

    packets.forEach((pkt, idx) \=\> {  
      const frameLen \= 14 \+ 20 \+ 8 \+ pkt.length;

      // Packet Header (16 bytes)  
      view.setUint32(offset, baseTime \+ idx, true);  
      view.setUint32(offset \+ 4, idx \* 50000, true); // Microseconds  
      view.setUint32(offset \+ 8, frameLen, true);  
      view.setUint32(offset \+ 12, frameLen, true);  
      offset \+= 16;

      // Ethernet Header (14 bytes)  
      const eth \= \[  
        0x00, 0x1A, 0x2B, 0x3C, 0x4D, 0x5E, // Dst MAC  
        0x00, 0x11, 0x22, 0x33, 0x44, 0x55, // Src MAC  
        0x08, 0x00                          // EtherType IPv4  
      \];  
      bytes.set(eth, offset);  
      offset \+= 14;

      // IPv4 Header (20 bytes)  
      const ipLen \= 20 \+ 8 \+ pkt.length;  
      const ip \= \[  
        0x45, 0x00,  
        (ipLen \>\> 8\) & 0xFF, ipLen & 0xFF,  
        0x12, 0x34, 0x40, 0x00,  
        0x40, 0x11, 0x00, 0x00, // TTL=64, Proto=UDP (17 / 0x11), Checksum  
        192, 168, 1, 100,       // Src IP (SCADA Host)  
        192, 168, 1, 20         // Dst IP (HART-IP Gateway)  
      \];  
      bytes.set(ip, offset);  
      offset \+= 20;

      // UDP Header (8 bytes)  
      const udpLen \= 8 \+ pkt.length;  
      const udp \= \[  
        (5094 \>\> 8\) & 0xFF, 5094 & 0xFF, // Src Port  
        (5094 \>\> 8\) & 0xFF, 5094 & 0xFF, // Dst Port  
        (udpLen \>\> 8\) & 0xFF, udpLen & 0xFF,  
        0x00, 0x00                       // Checksum  
      \];  
      bytes.set(udp, offset);  
      offset \+= 8;

      // HART-IP Payload  
      bytes.set(pkt, offset);  
      offset \+= pkt.length;  
    });

    return bytes;  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= { HARTEngine };  
}  
