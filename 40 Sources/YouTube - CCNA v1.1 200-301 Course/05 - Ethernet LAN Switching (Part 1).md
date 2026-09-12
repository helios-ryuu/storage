---
title: "YouTube: CCNA v1.1 200-301 Course - Day 5: Ethernet LAN Switching (Part 1)"
status: in-progress
tags:
  - ccna
  - youtube
  - source-note
---
## OSI Model - Physical Layer
- Physical Layer (L1) defines physical characteristics of the medium used to transfer data between devices. For example:
	- Voltage levels
	- Maximun transmission distances
	- Physical connectors
	- Cable specifications
	- ...
- Digital bits are converted into electrical (for wired connections) or radio (for wireless connections) signals.
## OSI Model - Data Link Layer
- Data Layer (L2) provides node-to-node connectivity and data transfer. For example:
	- PC to Switch
	- Switch to Router
	- Router to Router
- It defines how data is formatted for transmission over a physical medium: For example: Copper UTP cables
- It detects and corrects Physical Layer errors.
- It uses L2 addressing, separate from L3 addressing.
- Switches operate at L2.
## Local Area Networks (LANs)
- It a network contained in a relatively small area.
- Router are used to connect separate LANs.
## Ethernet Frame
- There are 5 fields in the header:
	- Preamble|SFD... (Start Frame Delimiter): This is used for synchronization and to allow the deviced to be prepared to receive the data of the frame.
	- ...Destination...: Declares L2 address to which the frame is being sent.
	- ...Source...: Declares L2 address of the device which sent the frame.
	- ...Type: This indicates the L3 protocol used in the encapsulated packet, which is almost always Internet Protocol (Or IPv4/IPv6). Sometimes this is the field indicates the length of the encapsulated data, depending on the version of the internet.
- There is 1 fields in the trailer:
	- FCS (Frame Check Sequence):  It's used by the receiving device to detect any errors that might have occured in transmission.
- Total size of header and trailer = 26 bytes
### Preamble & SFD
- Preamble:
	- Length: 7 bytes (56 bits)
	- Alternating 1's and 0's
	- 10101010 * 7
	- Allows devices to synchronize their receiver clocks
- SFD:
	- Length: 1 byte (8 bits)
	- 10101011
	- Marks the end of the preamble, and the beginning of the rest of the frame.
### Destination & Source
- Indicate the devices sending and receiving the frame
- Consist of the destination and source 'MAC (Media Access Control) address'
- Length = 6 bytes (48 bits) address of the physical device
### Type / Length
- Length: 2 bytes (16 bits)
- A value of 1500 or less in this field indicates the LENGTH of the encapsulated packet (in bytes)
- A value of 1536 or greater in this field indicates the TYPE of the encapsulated packet (usually IPv4 or IPv6), and the length is determined via other methods:
	- IPv4 = 0x0800 = 2048 = 00001000 00000000
	- IPv6 = 0x86DD = 34525 = 10000110 11011101
### FCS
- Length = 4 bytes (32 bits)
- Detects corrupted data by running a CRC (Cyclic Redundancy Check) algorithm over the received data
## MAC Address
- A.K.A. Burned-In Address (BIA)
- Is globally unique
- The first 3 bytes are the OUI (Organizationally Unique Indentifer), which is assigned to the company making the device
- The last 3 bytes are unique to the device itself
- Written as 12 hexadecimal characters
- Unicast frame: a frame destined for a single target. There are other kinds of frame like broadcast frame. After the switch received the frame, it look out the source MAC address field of the frame and then used that information to learn where the sender is