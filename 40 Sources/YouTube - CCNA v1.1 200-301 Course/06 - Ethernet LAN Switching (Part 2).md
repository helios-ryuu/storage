---
title: "YouTube: CCNA v1.1 200-301 Course - Day 6: Ethernet LAN Switching (Part 2)"
status: completed
tags:
  - ccna
  - youtube
  - source-note
---
## Ethernet Frame
- The Preamble + SFD is usually not considered part of the Ethernet header.
- The size of the Ethernet header + trailer is 18 bytes (6 + 6 + 2 + 4).
- The minimum size for an Ethernet frame (Header + Payload + Trailer) is 64 bytes (Frame's payload is packet). 64 - 18 = 46 bytes minimun for payload.
- If the payload is less than 46 bytes, padding bytes are added (these bytes are 0).
- The Ethernet Type code for an ARP packet is 0x0806
## Ethernet LAN Switching
![[Pasted image 20261003110957.png]]
- FFFF.FFFF.FFFF is boardcast MAC address.
![[Pasted image 20261003113941.png]]
![[Pasted image 20261003121021.png]]
![[Pasted image 20261003121356.png]]
![[Pasted image 20261003121602.png]]
![[Pasted image 20261003125021.png]]
- `clear mac address-table dynamic address <mac-address>`
- `clear mac address-table dynamic interface <interface-id>`
- `show arp`