# OSI Layer 1 – Physical (SOC Notes)

Part of my SOC learning log: Security+ (SY0-701) prep and blue-team fundamentals.

## What it does
Moves raw **bits** (1s and 0s) as physical signals:
- Electricity over copper cable
- Light over fibre optic
- Radio waves over Wi-Fi

**Unit:** bits  
**Address:** none. Layer 1 devices read no addresses, so they cannot filter traffic.

## Hardware
| Device | Layer | Note |
|---|---|---|
| Cable (copper, fibre) | 1 | Carries the signal |
| Repeater | 1 | Regenerates the signal to go farther |
| Hub | 1 | Repeats every signal out of every port |
| NIC | 1 and 2 | Sends signals (L1) and holds the MAC address (L2) |

Linux interface names: `eth0` (wired), `wlo1` (wireless).

## Attacks and defences
| Attack | What happens | Defence |
|---|---|---|
| Hub sniffing | Hub sends all traffic to every port | Replace hubs with switches |
| Wiretapping | Attacker physically taps a cable to copy traffic | Encryption (TLS), fibre, locked cable runs |
| Rogue device | Unauthorised device plugged into an open jack | Disable unused jacks, 802.1X / NAC, port security |
| Cable cut / damage | Link goes down | Redundant links on separate paths, locked wiring closets |
| Wi-Fi jamming | Radio noise blocks the signal | Wired backup, wireless monitoring |
| Signal leakage | Wireless signals escape a secure room | Faraday cage |

## Security concepts linked to Layer 1
- **Air gap:** a system with no physical network connection at all.
- **Faraday cage:** blocks radio signals in or out.
- Most Layer 1 incidents hit **availability** in the CIA triad. Tapping hits **confidentiality**.

## SOC / NOC view
- "Link down" on a switch port → check Layer 1 first: cable, port, NIC.
- Unknown device appearing on a switch port → possible rogue hardware.
- Troubleshoot bottom-up: Layer 1 before anything else.

## What I got wrong (and fixed)
- Mixed up hub and switch under exam pressure. Clue: *"can see everyone's traffic"* = hub.
- Picked port security for a cable tap. If the attacker is already on the wire, only **encryption** protects confidentiality.

## Sources
- Practical Networking – Networking Fundamentals, Lesson 2a
- TryHackMe – Networking Concepts
- Professor Messer – Security+ SY0-701

Next: Layer 2 – Data Link.
