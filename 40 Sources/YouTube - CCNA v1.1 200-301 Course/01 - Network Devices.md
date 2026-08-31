---
title: "YouTube: CCNA v1.1 200-301 Course - Day 1: Network Devices"
status: completed
tags:
  - ccna
  - youtube
  - source-note
---
![[network.jpg]]
A computer network is a digital telecommunications network which allows nodes to share resources. There are many types of node. The same device can be a client in some situations, and a server in other situations. Server and client are end hosts or endpoints:
- Server: It is a device that provides functions or services for client.
- Client: It is a device that accesses a service made available by a server.
- Switch: 
	- It serves a different function than a router, but it is also similar in many ways. Typically you don’t connect end hosts like PCs or servers directly to each other. You aggregate the connections to a device called a switch. 
	- Switches have lots of interfaces for you to connect end hosts to. 
	- Switches are used to forward traffic within a LAN (Local Area Network).
![[switch.jpg]]
	- Switches have many network interfaces or ports for end hosts, such as PCs, to connect to (usually 24+).
	- Switches provide connectivity to hosts within the same LAN.
	- Switches do not provide connectivity between LANs or over the internet. To do so, we need another kind of network device. That device is a router.
- Router:
	- We can connect switches to routers, and then connect the routers to the internet.
	- When end hosts in a LAN wants to communicate with end hosts in another LAN, they will send the data to their router, which will then forward it to the other LAN via the internet.
![[router.jpg]]
	- Routers have fewer network interfaces than switches.
	- Routers are used to provide connectivity between LANs.
	- Routers are therefore used to send data over the internet.
- Firewall: 
	- You probably familiar with firewalls, and you most likely have a firewall installed on your computer. That a software firewall, but large networks usually have a hardware firewall, a separate network appliance, which helps protect the network.
	- Firewalls are specialty network security devices that control network traffic entering and exiting your network. 
	- Firewalls can be placed before the router (outside the network) or inside of your network. They protect the end hosts inside.
	- Firewalls can be configured with security rules to determine which network traffic should be allowed and which should be denied.
![[firewall.jpg]]
	- Firewalls monitor and control network traffic based on configured rules. 
	- Firewalls can be placed ‘inside’ the network or ‘outside’ the network.
	- Firewalls are known as “Next-Generation Firewalls” when they include more modern and advanced filtering capabilities.
- Firewall on your computer:
	- Network firewalls are hardware devices that filter traffic between networks.
	- Host-based firewalls are software applications that filter traffic entering and exiting a host machine, like a PC.
## Quiz 1: Your company wants to purchase some network hardware to which they can plug the 30 PCs in your department. Which type of network device is appropriate?
A. A router
B. A firewall
**C. A switch**
D. A server
### Explanation: 
A router, like this Cisco ISR 900 series router, is designed for forwarding traffic between networks, not for connecting lots of end hosts like PCs to. A router will not typically have 30 network interfaces to connect hosts to.

A firewall, like this Cisco ASA 5500-X series firewall, is designed to filter traffic as it enters and exits the local network. It is not designed to connect directly to end hosts, and typically will not have enough network interfaces for 30 hosts.

A server is an end host itself, not a networking device to which you will connect other end hosts.

A switch, like this Cisco Catalyst 9200 series switch, is designed to connect many end hosts in the same LAN together. They include many network interfaces to connect hosts 
## Quiz 2: You received a video file from your friend's Apple iPhone using AirDrop. What was his iPhone functioning as in that transaction?
**A. A server**
B. A client
C. A local area network
### Explanation:
In this case your iPhone, not your friend's iPhone, is functioning as a client. A client accesses a service, it does not provide a service.

An end host like an iPhone does not function as a local area network (LAN) by itself. It can, however, be a part of a local area network.

A server is a device that provides functions or service for clients. In this case, your friend's phone provided the file to your iPhone.
## Quiz 3: What is your computer or smartphone functioning as while you watch this video?
A. A server
B. An end host
**C. A client**
### Explanation:
Your device is receiving a service, not providing one, so it is not functioning as a server.

Although your device is an end host, that does not describe its function. Both servers and clients are end hosts in a network.

Your device is receiving a service from YouTube's servers. Therefore, it is functioning as a client.
## Quiz 4: Your company wants to purchase some network hardware to connect its separate networks together. What kind of network device is appropriate?
A. A firewall
B. A host
C. A LAN
D. A router
### Explanation:
Although a firewall can connect multiple networks together, its real purpose is to monitor and control traffic as it enters and exits the network.

The term 'host' can refer to any type of network node.

LAN stands for Local Area Network. A LAN is not a network device itself.

A router is a device that is designed to connect and forward network traffic between multiple networks.
## Quiz 5: Your company wants to upgrade its old network firewall that has been in use for several years to one that provides more advanced functions. What kind of firewall should they purchase?
A. A host-based firewall
B. A next-level firewall
**C. A next-generation firewall**
D. A top-layer firewall
### Explanation:
A host-based firewall is a piece of software that runs on an end host, like the firewall on your computer. It is not a network firewall.

A next-generation firewall combines traditional firewall features with more advanced filtering functionalities.