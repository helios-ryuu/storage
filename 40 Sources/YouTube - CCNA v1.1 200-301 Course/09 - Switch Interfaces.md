---
title: |-
  YouTube: CCNA v1.1 200-301 Course - Day 9: Switch Interfaces
  ---
status: in-progress
tags:
  - ccna
  - youtube
  - source-note
---
## show interface status
![[Pasted image 20261010102032.png]]
- The port field lists each interface.
- The name field is the description of the interface.
- The status field
- The VLAN field can be used to divide LANs into smaller LANs. The default VLAN is 1 (The interface connected to the other switch is a trunk).
- The duplex field. a-full means it automatically negotiated a duplex of auto with the neighboring device.
- The speed field is also auto by default.
- The type field
## configuring interface speed and duplex
![[Pasted image 20261010131821.png]]
```txt
SW1(config)#interface range f0/5 - 12
SW1(config-if-range)#des ## not in use ##
SW1(config-if-range)#shutdown
SW1(config-if-range)#exit

SW1(config)#interface range f0/5 - 6, f0/9 - 12
SW1(config-if-range)#no shut
SW1(config-if-range)#
```
## Full/Half Duplex
![[Pasted image 20261010132439.png]]
![[Pasted image 20261010132550.png]]
![[Pasted image 20261010132714.png]]
![[Pasted image 20261010132806.png]]
![[Pasted image 20261010133012.png]]