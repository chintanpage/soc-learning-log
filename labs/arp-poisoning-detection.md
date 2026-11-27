# Lab: Detecting ARP Poisoning with Splunk

**Date:** 2026-11-26
**Analyst:** Chintan
**Goal:** Run an ARP poisoning (man-in-the-middle) attack in an isolated lab, detect it three ways, build a SIEM detection, and document it like a real incident.

> All testing was done inside my own isolated VMware lab. Indicators are defanged.

## Lab setup

| Role | Host | IP |
|---|---|---|
| Attacker | Kali | 192.168.159[.]130 |
| Victim | Windows 11 | 192.168.159[.]129 |
| SIEM | Ubuntu (Splunk + Zeek + Suricata + arpwatch) | 192.168.159[.]128 |
| Gateway | Router | 192.168.159[.]2 |

## What is ARP poisoning
ARP maps an IP address to a MAC address on the local network. In ARP poisoning, the attacker sends forged ARP replies claiming the gateway's IP belongs to the attacker's MAC. Victim traffic to the gateway then flows through the attacker first — a man-in-the-middle. The **IP never changes; only the MAC mapping changes.** It is a Layer 2 attack.

## The attack
From Kali:
```
sudo arpspoof -i eth0 -t 192.168.159.129 -r 192.168.159.2
```

## Detection 1 — victim ARP table (arp -a)

| IP | Before attack | After attack | Changed? |
|---|---|---|---|
| 192.168.159[.]2 (gateway) | 00:50:56:f2:16:5e | **00:0c:29:54:ea:8f** | YES — now points to Kali |
| 192.168.159[.]130 (Kali) | not listed | 00:0c:29:54:ea:8f | New — attacker appears |

Two signs: the gateway's MAC changed, and one MAC (`00:0c:29:54:ea:8f`) now answers for two IPs. The duplicate MAC is the attacker.

## Detection 2 — network sensor (arpwatch)
arpwatch on the SIEM logged the change to syslog:
```
arpwatch: flip flop 192.168.159.2 00:0c:29:54:ea:8f (00:50:56:f2:16:5e)
arpwatch: ethernet mismatch 192.168.159.2 00:0c:29:54:ea:8f (00:50:56:f2:16:5e)
```
"flip flop" = the MAC for that IP keeps switching between the real and the fake — a strong sign of active poisoning.

## Detection 3 — SIEM (Splunk)
Pipeline built: **arpwatch -> syslog -> Splunk**. Splunk monitors `/var/log/syslog`.

Detection search:
```
index=main sourcetype=syslog arpwatch ("flip flop" OR "ethernet mismatch")
| rex "arpwatch: (?<alert_type>flip flop|ethernet mismatch|changed ethernet address) (?<ip>\d+\.\d+\.\d+\.\d+) (?<fake_mac>[0-9a-f:]+) \((?<real_mac>[0-9a-f:]+)\)"
| table _time alert_type ip fake_mac real_mac
| sort - _time
```

A single-value dashboard tile on this search turns **red** when any ARP poisoning events appear, giving an at-a-glance alert while on duty.

## Key lesson
At first the attack did not appear in Splunk, because Zeek was only logging `conn` and `dns`. **A SIEM only detects what it collects.** Adding arpwatch as a log source made the attack visible. No log source = no detection.

## Incident ticket
```
INCIDENT: ARP Poisoning / MITM
Detected: SOC dashboard ARP Poisoning tile red and climbing
Spoofed IP: 192.168.159[.]2 (gateway)
Attacker MAC: 00:0c:29:54:ea:8f
Evidence: arpwatch flip-flop between real gateway MAC 00:50:56:f2:16:5e
          and attacker MAC, in index=main sourcetype=syslog arpwatch
Impact: man-in-the-middle; attacker can intercept gateway traffic
Action: isolate attacker MAC (disable switch port), find owner, enable DAI
Escalated: L2
```

## Defences
- **Dynamic ARP Inspection (DAI)** on managed switches — drops forged ARP replies.
- **Static ARP entries** for critical devices like the gateway.
- Network monitoring (arpwatch / IDS) to alert on MAC changes.

## Tools used
Kali (arpspoof), Windows 11 (arp -a), Ubuntu SIEM (arpwatch, Splunk), VMware lab.
