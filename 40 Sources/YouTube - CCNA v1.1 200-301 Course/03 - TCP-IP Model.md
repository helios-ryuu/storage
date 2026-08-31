---
title: "YouTube: CCNA v1.1 200-301 Course - Day 3: TCP/IP Model"
status: in-progress
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
