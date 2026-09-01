---
title: "YouTube: CCNA v1.1 200-301 Course - Day 3: TCP/IP Model"
status: completed
tags:
  - ccna
  - youtube
  - source-note
---
## Protocols and standards
- A protocol is a set of rules defining how data should be communicated between devices over a network.
- A standard is an agreed-upon specification that describes how a protocol or technology should work.
## Layered models
- A model lets us group related jobs into layers. Each layer has a specific role, uses the services of the layer below, and provides services to the layer above.
- Protocols live (mostly) at one layer (IP, TCP, HTTP, etc.). Together they form a stack of protocols that work as a team (the network stack).
![[Pasted image 20260831163312.png]]
![[Pasted image 20260831164044.png]]
![[Pasted image 20260831164805.png]]
### Layer 1: Physical Layer
- This layer sends and receives bits as electrical, optical, or radio signal over the medium. It defines things like cables, connectors, signal levels, and link speeds.
- Example:
	- Copper UTP cables
	- Fiber-optic cables
	- Wi-Fi radios and antennas
	- Network Interface Cards (NICs)
> [!INFO]
> Network engineers don't have to know the low-level details.

![[Pasted image 20260831165720.png]]
### Layer 2: Data Link Layer
- This layer provides hop-to-hop delivery of messages on a local network. A hop is one step along the path between two devices: From router or host, to the next router or host in the path (Switches don't count).
- L2 uses MAC (Media Access Control) addresses to identify interfaces.
- Most commonly used protocols at this layer include:
	- Ethernet (IEEE 802.3)
	- Wi-Fi (IEEE 802.11)
![[Pasted image 20260831232612.png]]

**Bản chất kỹ thuật tại sao Switch không tạo thành một Hop**:
- **Tính trong suốt của Switch (Layer 2 Transparency):**
    - Switch chỉ đọc bảng địa chỉ MAC và chuyển tiếp nguyên vẹn khung (Ethernet Frame) mà không thay đổi hay bóc mở cấu trúc gói tin Layer 3 (IP Packet).
    - Đối với thiết bị gửi (PC1) và thiết bị nhận tiếp theo (Router R1), toàn bộ hệ thống cáp và Switch ở giữa đóng vai trò như một đường truyền nội bộ duy nhất (Single Broadcast/Collision Domain mở rộng).

- **Không làm giảm chỉ số TTL (Time To Live):**
    - Mỗi một **hop Layer 3** (Router) khi xử lý gói tin bắt buộc phải giảm giá trị trường $TTL = TTL - 1$ trong IP Header để chống lặp vòng lặp vô tận (Routing Loop).
    - Switch Layer 2 không đọc và không can thiệp vào IP Header, do đó giá trị $TTL$ được giữ nguyên tuyệt đối khi đi qua SW1 hay SW2.

- **Không thay đổi Header IP hay phân chia Subnet:**
    - Một hop xảy ra khi dữ liệu chuyển từ dải mạng/Subnet này sang dải mạng/Subnet khác.
    - Cả PC1, SW1 và cổng kết nối của R1 đều nằm chung trong cùng **một dải Subnet logic**.
### Layer 3: Network Layer
![[Pasted image 20260831233138.png]]
- This layer provides end-to-end delivery between hosts across multiple networks.
- L3 uses IP addresses to identify hosts in the network.
- Routers operate mainly at this layer, using the message's destination IP address to forward message toward its final destination host.
- Protocols at this layer include:
	- IP (IPv4, IPv6)
	- ICMP (Internet Control Message Protocol)
### Layer 4: Transport Layer
![[Pasted image 20260831233759.png]]
- This layer provides end-to-end communication between application processes (also called "process-to-process" or "service-to-service").
- L4 uses port numbers to identify processes on each host.
- It runs mainly on the communicating hosts; routers normally operate based on IP (L3), not on Transport-layer information.
- Protocols at this layer include:
	- UDP (User Diagram Protocol): simple and efficient.
	- TCP (Transmission Control Protocol): more robust features beyond basic message addressing.
### Layer 5: Application Layer
![[Pasted image 20260831234426.png]]
- This layer is where network communications meet applications. It defines how application processes format, send, and interpret data.
- It is also called L7.
- Protocols at this layer define message formats and rules for specific tasks, such as:
	- Browsing web pages (HTTP/HTTPS)
	- Transfering files (FTP, TFTP)
	- Sending/receiving email (SMTP, POP3, IMAP)
- Network infrastructure devices (routers, switches) don't care about Application-layer details. They just move messages across the network, only the communicating hosts interpret the data.
## Encapsulation & Decapsulation
### 1. Encapsulation
![[Pasted image 20260901115128.png]]
1. The Application layer prepares the data to be sent over the network.
2. As the message moves down the stack, each layer encapsulates the data with a header including the information needed for that layer:
	- Source and destination addresses (port numbers, IP addresses, MAC addresses), etc.
	- L2 also adds a trailer that the receiving device uses to check for transmisstion errors.
3. The Physical layer transmit the bits as signals over the physical medium.
	- The L2 header is transmitted first, and the L2 trailer is transmitted last.
### 2. Decapsulation
![[Pasted image 20260901115916.png]]
1. The receiving device receives the message as a stream of bits at L1.
2. The device examines the information in the L2 header and trailer, and then removes them (decapsulation).
	- The decapsulation process continues up the stack: L3 removes the L3 header, then L4 removes the L4 header, and then the data is delivered to the Application layer.
3. The application processes the data and, if needed, generates a response that goes back down the stack.
## Protocol data units
![[Pasted image 20260901122158.png]]
- At each stage in the Encapsulation/Decapsulation process, there is a name given to the message:
	- The combination of data and a L4 header is called a segment (TCP) or datagram (UDP).
	- The combination of a segment/datagram and a L3 header is called a packet.
	- The combination of a packet and a L2 header/trailer is called a frame. This is what is actually sent over the wire.
- We can use alternative names to describe the message at each stage: protocol data unit (PDU):
	- A segment or datagram is a L4PDU.
	- A packet is a L3PDU.
	- A frame is a L2PDU.
- The content of each PDU are called the payload:
	- A segment or datagram's payload is the application data.
	- A packet's payload is a segment or datagram.
	- A frame's payload is a packet.
## Adjacent-layer interaction
![[Pasted image 20260901123829.png]]
- Each layer provides a service to the layer above it, and is serviced by the layer below it (adjacent-layer interaction): 
	- Layer 4 provides a service to Layer 5 by delivering data to the correct application using port numbers.  
	- Layer 3 provides a service to Layer 4 by delivering segments/datagrams to the correct destination host using IP addresses.  
	- Layer 2 provides a service to Layer 3 by delivering packets to the next hop using MAC addresses.  
	- Layer 1 provides a service to Layer 2 by sending and receiving frames as electrical, optical, or radio signals.
- Each layer communicates with the same layer on other devices (same-layer interaction):
	- The Application layer on one host sends data to the Application layer on the other host.  
	- A segment/datagram is addressed to the Layer 4 port number of the correct application on the destination host.  
	- A packet is addressed to the Layer 3 IP address of the destination host.  
	- A frame is addressed to the Layer 2 MAC address of the next hop.  
	- Signals sent out of a physical port are received by a physical port on the connected device.
- The layer are modular, we can swap protocols at one layer without changing the others.