# ARP Poisoning (MiTM) Attack & Detection Lab

**Ettercap + Wireshark | Network Traffic Analysis | SOC-Focused Project**

Note: This documentation was refined using AI-assisted editing to improve clarity and presentation, while maintaining my original analysis and findings.

---

## Executive Summary

This project simulates a **Man-in-the-Middle (MiTM) attack via ARP poisoning** and demonstrates how a SOC analyst can **detect, investigate, and validate** the attack using network traffic analysis in Wireshark.

The lab focuses not just on exploitation, but on:

* Identifying **indicators of compromise (IoCs)**
* Building **detection filters**
* Understanding attacker behavior at the packet level

---

## Objectives

* Simulate ARP poisoning using Ettercap
* Capture live attack traffic in Wireshark
* Identify **anomalous ARP behavior**
* Build a reusable **detection filter**
* Map activity to **SOC triage workflows**

---

## Lab Environment

| Role     | System      | Function                  |
| -------- | ----------- | ------------------------- |
| Attacker | Kali Linux  | Runs Ettercap + Wireshark |
| Victim   | Windows 10  | Target endpoint           |
| Gateway  | Router (xx.xx.xx.1) | Network gateway           |

**Interface:** `eth0`
**Attack Type:** ARP Cache Poisoning (Layer 2)

---

## Attack Setup (Condensed)

### 1. Identify Network Targets

* Enumerate IP ↔ MAC mappings using:

  * `arp -a`
  * `ipconfig` / `ifconfig`

### 2. Enable Traffic Forwarding (Attacker)

```bash
sysctl net.ipv4.ip_forward=1
```

### 3. Launch Attack

* Scan for hosts in Ettercap
* Assign:

  * Target 1 → Victim
  * Target 2 → Gateway
* Start **ARP Poisoning (MiTM)**

---

## Attack Validation

### Victim-Side Indicator

```cmd
arp -a
```

**Observed Behavior:**

* Gateway IP resolves to attacker’s MAC address
* Confirms ARP cache poisoning

---

## 📡 Traffic Interception

While MiTM is active:

* Victim sends traffic (e.g., `ping 8.8.8.8`, web browsing)
* Attacker captures packets via Wireshark

**Key Insight:**
Traffic is transparently relayed through the attacker due to IP forwarding.

---

## Detection Analysis (SOC Perspective)

### Initial Triage Filter Wireshark

```
arp
```

### Indicators of Compromise (IoCs)

| Indicator                   | Description                                  |
| --------------------------- | -------------------------------------------- |
| Unsolicited ARP replies     | ARP responses without corresponding requests |
| Duplicate IP → MAC mappings | Same IP resolves to multiple MACs            |
| ARP Reply Storm             | High frequency of ARP reply packets          |
| Opcode 2 anomalies          | Excessive ARP replies (opcode 2)             |

---

## Deep Packet Analysis

### Suspicious Packet Characteristics:

* **Ethernet II Layer**

  * Source MAC = Attacker
* **ARP Layer**

  * Opcode: `Reply (2)`
  * Claiming to be:

    * Gateway → Victim
    * Victim → Gateway

These packets are **forged identity assertions**

---

## Custom Detection Filter

### Wireshark Filter

```
((arp.src.proto_ipv4 == xx.xx.xx.1) && (arp.opcode == 2)) && !(arp.src.hw_mac == <LEGIT_GATEWAY_MAC>)
```

---

### Detection Logic

This filter isolates:

* ARP replies claiming to be from the **gateway**
* Excludes legitimate MAC address
* Surfaces **spoofed ARP responses**

---

### SOC Workflow Integration

* Save filter as: `ARP_Poison_Detection`
* Add to Wireshark profile for rapid triage
* Can be translated into:

  * IDS/IPS rules
  * SIEM detection queries

---

## Attack Flow (Analyst View)

1. Attacker sends forged ARP replies
2. Victim updates ARP table → trusts attacker as gateway
3. Gateway updates ARP table → trusts attacker as victim
4. Traffic is rerouted through attacker
5. Attacker forwards packets (maintains session integrity)

---

## Mitigation Strategies

| Control                      | Description                             |
| ---------------------------- | --------------------------------------- |
| Dynamic ARP Inspection (DAI) | Validates ARP packets on switches       |
| Static ARP entries           | Prevents spoofing (limited scalability) |
| Network segmentation         | Reduces attack surface                  |
| IDS/IPS monitoring           | Detects anomalous ARP patterns          |
| Encryption (HTTPS, TLS)      | Limits data exposure                    |

---

## SOC Analyst Takeaways

* ARP poisoning is **low complexity, high impact**
* Detection relies on **behavioral anomalies**, not signatures alone
* Wireshark is effective for:

  * **Packet-level validation**
  * **Incident investigation**
* Building filters = **real SOC skill**, not just lab work

---

## Skills Demonstrated

* Network Traffic Analysis (Wireshark)
* Threat Detection & Triage
* Layer 2 Attack Understanding
* Packet-Level Forensics
* Detection Engineering (custom filters)
* Adversarial Thinking

---

## Disclaimer

This project was conducted in a **controlled lab environment** for educational purposes only. No unauthorized systems or networks were targeted.

---

