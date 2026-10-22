---
title: |-
  YouTube: CCNA v1.1 200-301 Course - Day 8: IPv4 Addressing (Part 2)
  ---
status: completed
tags:
  - ccna
  - youtube
  - source-note
---
![[Pasted image 20261007161014.png]]
![[Pasted image 20261007161144.png]]
- Show interfaces:
```txt
R1>en
R2#show ip interface brief
```
![[Pasted image 20261007161446.png]]
- `administratively down`: Interface has been disabled with the `shutdown` command. This is the default status of Cisco router interfaces. (Cisco switch interfaces are not `administratively down` by default):
	- Status: L1 status
	- Protocol: L2 status 
- Configure interfaces:
```txt
R1#conf t
R1(config)#interface gigabitethernet 0/0
R1(config-if)#

# or
R1(config)#in g0/0
R1(config-if)#ip address 10.255.255.254 255.0.0.0
R1(config-if)#no shutdown
R1(config-if)#do sh ip i b
R1(config-if)#exit

R1(config)#in g0/1
R1(config-if)#ip address 172.16.255.254 255.255.0.0
R1(config-if)#no shutdown
R1(config-if)#do sh ip i b
R1(config-if)#exit

R1(config)#in g0/2
R1(config-if)#ip address 192.168.0.254 255.255.255.0
R1(config-if)#no shutdown
R1(config-if)#do sh ip i b
R1(config-if)#exit

R1(config)#do sh in g0/0
R1(config)#in g0/0
R1(config-if)#des ## to SW1 ##
R1(config-if)#do sh in g0/0 | inc De
R1(config-if)#do sh in | inc ^(Gi| *De)
```
![[Pasted image 20261007164735.png]]