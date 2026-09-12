---
title: "YouTube: CCNA v1.1 200-301 Course - Day 4: Intro to the CLI"
status: completed
tags:
  - ccna
  - youtube
  - source-note
---

## 1. What is a CLI? (Giao diện dòng lệnh)

![[Pasted image 20260903152150.png]]

- **CLI (Command-Line Interface)**: Giao diện dựa trên văn bản (text-based) mà kỹ sư mạng sử dụng để tương tác, cấu hình và khắc phục sự cố trên hệ điều hành **Cisco IOS** (Internetwork Operating System).
- **GUI (Graphical User Interface)**: Giao diện đồ họa người dùng với các nút bấm, menu trực quan (ví dụ: Cisco DNA Center, Cisco Configuration Professional).

> [!NOTE]
> **Tại sao CLI vẫn là tiêu chuẩn vàng trong quản trị mạng doanh nghiệp?**
> - **Hiệu quả và tốc độ**: Thao tác phím nhanh hơn nhiều so với việc click qua hàng loạt menu.
> - **Tự động hóa (Automation & Scripting)**: Dễ dàng cấu hình hàng loạt thiết bị bằng script (Python, Ansible, Bash).
> - **Tiết kiệm tài nguyên & Băng thông**: Truy cập qua kết nối từ xa yêu cầu băng thông cực thấp so với truyền tải giao diện đồ họa.
> - **Tính toàn diện**: CLI luôn cung cấp 100% các tính năng cấu hình nâng cao, trong khi GUI thường chỉ hỗ trợ một phần các cấu hình phổ biến.

---

## 2. How to connect to a Cisco device? (Kết nối vật lý qua Console Port)

Quản trị thiết bị mạng được chia làm 2 hình thức:
- **Out-of-Band (OOB) Management**: Quản trị "ngoài băng", kết nối trực tiếp vào thiết bị qua cổng chuyên dụng (**Console port**) mà không phụ thuộc vào trạng thái mạng hay địa chỉ IP của thiết bị. Thường dùng khi cấu hình ban đầu (initial setup) hoặc cứu hộ khi mất mạng.
- **In-Band Management**: Quản trị "trong băng", kết nối từ xa qua mạng thông qua các giao thức như **SSH** (khuyến nghị, bảo mật) hoặc **Telnet** (không mã hóa). Hình thức này đòi hỏi thiết bị đã có địa chỉ IP và mạng đang hoạt động bình thường.

### 2.1. Cổng Console trên thiết bị (Console Ports)
![[Pasted image 20260903152322.png]]

- Các thiết bị Cisco (Router, Switch) thường có cổng Console chuyên biệt, được đánh dấu bằng viền hoặc chữ màu xanh dương nhạt.
- Có 2 loại cổng Console thông dụng:
  - **Cổng RJ-45 Console**: Cổng mạng chuẩn RJ-45 nhưng hoạt động ở chuẩn nối tiếp RS-232.
  - **Cổng Mini-USB / Type-C Console**: Được trang bị trên các dòng thiết bị Cisco hiện đại hơn, cho phép cắm trực tiếp cáp USB từ máy tính.

### 2.2. Cáp Console truyền thống (Rollover Cable)
![[Pasted image 20260903152432.png]]

- Cáp màu xanh dương nhạt (light blue cable) thường đi kèm hộp thiết bị Cisco.
- Đầu một đầu là **RJ-45** (cắm vào cổng Console của Router/Switch), đầu còn lại là **DB-9 (RS-232 serial connector)** để cắm vào cổng nối tiếp (COM port) của PC.
- Cáp này được gọi là **Rollover Cable** vì sơ đồ chân (pinout) của một đầu bị đảo ngược hoàn toàn so với đầu kia (chân 1 nối chân 8, chân 2 nối chân 7...).

### 2.3. Bộ chuyển đổi USB-to-Serial & Cáp Console USB
- Các dòng máy tính và laptop hiện đại không còn cổng COM (DB-9) vật lý.
- Do đó, kỹ sư mạng sử dụng:
  - **Bộ chuyển đổi USB-to-DB9 (USB-to-Serial Adapter)** kết hợp với cáp Rollover truyền thống.
  - Hoặc sử dụng cáp đúc sẵn **USB to RJ-45 Console Cable** (tích hợp chip FTDI / Prolific bên trong đầu USB).
  - Cáp **USB-A sang Mini-B USB** nếu thiết bị hỗ trợ cổng Console USB mini.

### 2.4. Phần mềm giả lập Terminal (Terminal Emulator Software)![[Pasted image 20260903152552.png]]

Để truyền nhận tín hiệu text qua cổng COM giữa PC và thiết bị Cisco, cần cài đặt phần mềm giả lập terminal:
- **PuTTY**: Miễn phí, cực kỳ phổ biến và nhẹ.
- **Tera Term**: Miễn phí, hỗ trợ macro tốt.
- **SecureCRT**: Trả phí, chuyên nghiệp, hỗ trợ quản lý nhiều tab session tiện lợi cho kỹ sư mạng.

### 2.5. Thiết lập thông số kết nối Serial (Serial Port Settings - Chuẩn 9600-8-N-1)
Khi thiết lập kết nối Serial trên phần mềm giả lập (ví dụ PuTTY), bắt buộc phải chọn đúng cổng COM (xem trong Windows Device Manager) và cấu hình đúng 5 thông số mặc định của Cisco:

| Thông số (Parameter) | Giá trị tiêu chuẩn | Ý nghĩa |
| :--- | :--- | :--- |
| **Bits per second (Baud rate)** | **`9600`** | Tốc độ truyền tải tín hiệu (9600 bps) |
| **Data bits** | **`8`** | Mỗi ký tự truyền gồm 8 bit dữ liệu |
| **Parity** | **`None`** | Không sử dụng bit chẵn/lẻ để kiểm tra lỗi |
| **Stop bits** | **`1`** | 1 bit dừng để báo hiệu kết thúc ký tự |
| **Flow control** | **`None`** | Không điều khiển luồng phần cứng |

> [!TIP]
> Nhớ quy tắc ngắn gọn: **9600 - 8 - N - 1 - None**.

---

## 3. Cisco IOS CLI Modes (Hệ thống các chế độ làm việc trong CLI)

Cisco IOS được thiết kế theo cấu trúc phân cấp (hierarchical mode structure) nhằm kiểm soát phân quyền và ngăn chặn thay đổi ngoài ý muốn.

![[Pasted image 20260903152833.png]]

### 3.1. Các cấp độ chế độ chính

| Chế độ (Mode) | Dấu nhắc (Prompt) | Mục đích & Quyền hạn | Lệnh kích hoạt | Lệnh thoát |
| :--- | :--- | :--- | :--- | :--- |
| **User EXEC Mode** | `Router>`<br>`Switch>` | Chế độ cơ bản với quyền hạn thấp nhất. Chỉ cho phép xem thông tin cơ bản, thực hiện lệnh kiểm tra đơn giản (`ping`, `traceroute`). Không thể xem toàn bộ cấu hình hay thay đổi bất kỳ cài đặt nào. | Mặc định khi mới kết nối vào thiết bị | `exit` (ngắt phiên kết nối) |
| **Privileged EXEC Mode**<br>*(Enable Mode)* | `Router#`<br>`Switch#` | Quyền quản trị tối cao (Admin). Cho phép xem toàn bộ file cấu hình (`show running-config`), khởi động lại (`reload`), quản lý file hệ thống, thực hiện debug. Vẫn **chưa** sửa trực tiếp thông số tại mode này. | `enable` (từ User EXEC) | `disable` (về User EXEC)<br>`exit` (đóng phiên) |
| **Global Configuration Mode** | `Router(config)#`<br>`Switch(config)#` | Cho phép thay đổi cấu hình ảnh hưởng đến toàn bộ thiết bị (đặt hostname, bật/tắt dịch vụ toàn cục, cài đặt mật khẩu...). | `configure terminal`<br>*(viết tắt: `conf t`)* | `exit` (về Privileged EXEC)<br>`end` / `Ctrl+Z` |
| **Sub-configuration Modes** | `Router(config-xxx)#` | Cấu hình các thành phần con cụ thể (cổng giao tiếp, đường truyền truy cập, giao thức định tuyến...). | Tùy lệnh (xem bên dưới) | `exit` (lùi 1 cấp)<br>`end` / `Ctrl+Z` (về thẳng `#`) |

### 3.2. Các Sub-configuration Modes thông dụng
- **Interface Configuration Mode**:
  - Lệnh: `interface <tên-cổng>` (ví dụ: `interface GigabitEthernet0/0` hoặc viết tắt `int g0/0`)
  - Prompt: `Router(config-if)#`
  - Dùng để đặt địa chỉ IP, subnet mask, bật cổng (`no shutdown`), điều chỉnh tốc độ, duplex...
- **Line Configuration Mode**:
  - Lệnh: `line console 0` (cổng console vật lý) hoặc `line vty 0 4` (phiên SSH/Telnet ảo)
  - Prompt: `Router(config-line)#`
  - Dùng để cài đặt mật khẩu đăng nhập, thời gian timeout cho phiên kết nối...
- **Routing Configuration Mode**:
  - Lệnh: `router ospf 1`, `router bgp 65000`...
  - Prompt: `Router(config-router)#`
  - Cấu hình các tham số cho giao thức định tuyến động.
- **VLAN Configuration Mode**:
  - Lệnh: `vlan 10`
  - Prompt: `Switch(config-vlan)#`
  - Cấu hình đặt tên và thuộc tính cho VLAN trên Switch.

### 3.3. Điều hướng giữa các chế độ (Navigating Between Modes)

```
  [User EXEC Mode]           Router>
         | 
         |  gõ: enable
         v
  [Privileged EXEC Mode]     Router#  <---------------------------------+
         |                                                              |
         |  gõ: configure terminal                                      |
         v                                                              |
  [Global Config Mode]       Router(config)#                            |
         |                                  \                           |
         |  gõ: interface g0/0               \ gõ: line console 0       |
         v                                    v                         |
  [Sub-config (Interface)]   Router(config-if)#  [Sub-config (Line)]    |
                                      |                   |             |
                                      +---------+---------+             |
                                                |                       |
                                                +--- gõ 'end' ----------+
                                                |    hoặc Ctrl + Z
                                                v
                                 (Gõ 'exit' sẽ chỉ lùi lên đúng 1 cấp)
```

- **`exit`**: Lùi lên **đúng 1 cấp** trong cấu trúc phân cấp (ví dụ: từ `config-if` về `config`, hoặc từ `config` về Privileged EXEC `#`).
- **`end`** hoặc tổ hợp phím **`Ctrl + Z`**: Lập tức thoát khỏi **bất kỳ sub-config hay config mode nào** để quay thẳng về **Privileged EXEC Mode (`#`)**.
- **`disable`**: Rớt quyền từ Privileged EXEC (`#`) về User EXEC (`>`).

---

## 4. CLI Navigation, Shortcuts & Help Features (Trợ giúp & Phím tắt)

### 4.1. Tính năng trợ giúp ngữ cảnh (Context-Sensitive Help: `?`)
Hệ thống Cisco IOS tích hợp tính năng trợ giúp thông minh bằng dấu chấm hỏi `?`:

1. **Hiển thị danh sách lệnh khả dụng ở mode hiện tại**:
   - Gõ `?` trực tiếp ở dấu nhắc lệnh.
2. **Tìm các lệnh bắt đầu bằng ký tự nhất định (không có khoảng trắng)**:
   - Gõ `c?` $\rightarrow$ Cisco IOS sẽ liệt kê tất cả các lệnh bắt đầu bằng chữ cái `c` (ví dụ: `clear`, `clock`, `configure`, `connect`).
3. **Hiển thị tham số/từ khóa tiếp theo (có dấu cách)**:
   - Gõ `clock ?` $\rightarrow$ CLI hiển thị danh sách từ khóa hoặc cú pháp tiếp theo (ví dụ: `set`).
   - Gõ `clock set ?` $\rightarrow$ CLI hướng dẫn định dạng thời gian cần nhập (ví dụ: `hh:mm:ss`).

### 4.2. Tự động hoàn thành lệnh (Tab Completion) & Viết tắt lệnh (Abbreviation)
- **Phím `Tab`**: Tự động điền phần còn lại của lệnh khi bạn đã gõ đủ số ký tự để phân biệt từ khóa đó là duy nhất.
  - Ví dụ: `Router# conf` rồi bấm `Tab` $\rightarrow$ CLI tự động mở rộng thành `Router# configure`.
- **Viết tắt lệnh (Command Abbreviation)**: Không cần bấm Tab hay gõ hết cả từ, thiết bị vẫn hiểu nếu tiền tố gõ vào là duy nhất:
  - `conf t` $\Leftrightarrow$ `configure terminal`
  - `sh run` $\Leftrightarrow$ `show running-config`
  - `int g0/0` $\Leftrightarrow$ `interface GigabitEthernet0/0`
  - `no shut` $\Leftrightarrow$ `no shutdown`
  - `wr` $\Leftrightarrow$ `write`

### 4.3. Các phím tắt quan trọng (Essential Hotkeys)
- **Mũi tên lên (`↑`) / xuống (`↓`)**: Duyệt lại lịch sử các câu lệnh đã thực thi gần đây (Command history).
- **`Ctrl + A`**: Di chuyển con trỏ văn bản về **đầu dòng**.
- **`Ctrl + E`**: Di chuyển con trỏ văn bản về **cuối dòng**.
- **`Ctrl + C`**: Hủy bỏ dòng lệnh hiện tại mà không thực thi, hoặc ngắt lệnh đang chạy như `ping`.
- **`Ctrl + Shift + 6`**: **Ngắt tiến trình mạng bị treo (Aborts ongoing process)**.
  - *Ứng dụng thực tế*: Khi bạn ở Privileged EXEC hoặc User EXEC và gõ nhầm một từ vô nghĩa (ví dụ `conft` thay vì `conf t`), Cisco IOS mặc định hiểu nhầm đó là một tên miền (domain name) và gửi gói tin DNS broadcast để tra cứu IP: `Translating "conft"...domain server (255.255.255.255)...`. CLI sẽ bị đơ trong khoảng 30s. Bấm `Ctrl + Shift + 6` sẽ hủy ngay quá trình này.

---

## 5. Configuration Files: Running-config vs Startup-config

![[Pasted image 20260903153006.png]]

Cisco IOS quản lý cấu hình thông qua 2 file độc lập nằm ở hai loại bộ nhớ khác nhau:

| Tiêu chí | `running-config` | `startup-config` |
| :--- | :--- | :--- |
| **Vị trí lưu trữ** | **RAM (Random Access Memory)** | **NVRAM (Non-Volatile RAM)** |
| **Đặc tính bộ nhớ** | Khả biến (Volatile) – **Mất sạch dữ liệu** khi mất điện hoặc reload thiết bị. | Bền vững (Non-volatile) – **Giữ nguyên dữ liệu** kể cả khi mất điện hay khởi động lại. |
| **Thời điểm áp dụng** | Là cấu hình **đang chạy trực tiếp**. Mọi câu lệnh bạn gõ trong Config Mode sẽ có hiệu lực **ngay lập tức** vào file này. | Được thiết bị nạp vào RAM (thành `running-config`) mỗi khi thiết bị khởi động (boot sequence). |
| **Lệnh kiểm tra** | `show running-config`<br>*(viết tắt: `sh run`)* | `show startup-config`<br>*(viết tắt: `sh start`)* |

### 5.1. Lưu cấu hình (Saving Configuration)
Khi bạn chỉnh sửa cấu hình trong Global Config, cấu hình đó mới chỉ nằm trong **RAM**. Nếu thiết bị đột ngột mất điện, toàn bộ thay đổi sẽ biến mất. Muốn lưu cấu hình vĩnh viễn, bạn phải sao chép từ `running-config` vào `startup-config`:

- **Lệnh chuẩn Cisco (Khuyên dùng trong kỳ thi CCNA)**:
  ```
  Router# copy running-config startup-config
  ```
  *(Có thể viết tắt: `copy run start`)*
- **Lệnh tắt truyền thống (Rất phổ biến ngoài thực tế)**:
  ```
  Router# write
  hoặc
  Router# write memory
  ```
  *(Có thể viết tắt: `wr`)*

### 5.2. Xóa cấu hình để đưa thiết bị về mặc định xuất xưởng (Factory Reset)
Nếu muốn xóa toàn bộ cấu hình cũ:
```
Router# erase startup-config
(hoặc: Router# write erase)
Router# reload
```
> Thiết bị sẽ hỏi xác nhận lưu lại cấu hình running hay không, chọn **`no`**, sau đó thiết bị sẽ khởi động lại với cấu hình trắng tinh.

---

## 6. Lệnh `do` trong Configuration Mode

- **Nguyên tắc**: Các lệnh kiểm tra như `show`, `ping`, `traceroute`, `write` là các lệnh thuộc **Privileged EXEC Mode (`#`)**. Khi bạn đang ở bên trong Configuration Mode (`(config)#` hay `(config-if)#`), bạn không thể gõ trực tiếp `show running-config`.
- **Giải pháp thông thường**: Gõ `exit` hoặc `end` ra `#`, chạy lệnh xong rồi lại gõ `conf t` vào lại. Rất mất thời gian!
- **Giải pháp tối ưu - Dùng tiền tố `do`**:
  - Bằng cách thêm từ khóa `do` trước câu lệnh, bạn có thể thực thi **bất kỳ lệnh EXEC nào ngay từ bên trong Config Mode**:
  ```cisco
  Router(config)# do show running-config
  Router(config)# do write
  Router(config-if)# do ping 192.168.1.1
  Router(config-line)# do show ip interface brief
  ```
- **Lưu ý kỹ thuật**: Trong một số phiên bản Cisco IOS cũ, sau từ khóa `do` sẽ không hỗ trợ phím `Tab` hay dấu `?` trợ giúp.

---

## 7. Password Security & Encryption (Bảo mật & Mã hóa mật khẩu)

### 7.1. Phân biệt `enable password` và `enable secret`
Cả hai câu lệnh này đều dùng để đặt mật khẩu bảo vệ khi chuyển từ User EXEC (`>`) sang Privileged EXEC (`#`):

- **`enable password <mật-khẩu>`**:
  - Lưu mật khẩu dưới dạng **văn bản thuần (clear-text / plaintext)** trong `running-config`.
  - Bất kỳ ai nhìn vào màn hình hoặc gõ `show run` đều đọc được mật khẩu này $\rightarrow$ **Cực kỳ không an toàn**.
- **`enable secret <mật-khẩu>`**:
  - Mã hóa mật khẩu bằng hàm băm bảo mật (MD5 - Type 5 hoặc SHA-256 - Type 8/9).
  - Chuỗi mã hóa trong file cấu hình là một chiều (one-way hash), không thể dịch ngược.
  - **Luôn luôn ưu tiên sử dụng `enable secret`**. Nếu cả hai lệnh cùng được cấu hình, `enable secret` sẽ ghi đè và có hiệu lực.

### 7.2. Lệnh `service password-encryption`
Khi cấu hình mật khẩu cho cổng Console (`line console 0`) hoặc phiên từ xa (`line vty`), mặc định Cisco IOS lưu chúng dưới dạng clear-text.
Để ẩn những mật khẩu này đi:

```cisco
Router(config)# service password-encryption
```

- **Bản chất kỹ thuật của mã hóa Type 7**:
  - Lệnh này mã hóa toàn bộ mật khẩu dạng clear-text hiện có thành **Cisco Type 7**.
  - **Type 7 là một thuật toán mã hóa Vigenère đảo ngược rất yếu (weak reversible cipher)**. Bất kỳ ai cũng có thể copy chuỗi Type 7 này và giải mã ra mật khẩu gốc trong vòng 1 giây bằng các trang web trực tuyến (Cisco password cracker).
  - **Mục đích thực sự của Type 7**: Chỉ nhằm chống việc "nhìn trộm qua vai" (prevent shoulder surfing) khi ai đó vô tình đứng cạnh nhìn vào màn hình cấu hình, chứ **không** bảo vệ được dữ liệu trước các cuộc tấn công bảo mật thực sự.
- **Tắt mã hóa**:
  ```cisco
  Router(config)# no service password-encryption
  ```
  > [!WARNING]
  > Việc gõ `no service password-encryption` sẽ chỉ ngăn chặn việc mã hóa các mật khẩu được tạo *sau này*. Những mật khẩu *đã bị mã hóa Type 7 trước đó vẫn giữ nguyên dạng mã hóa*. Nếu muốn chúng trở lại clear-text, bạn phải vào cấu hình lại mật khẩu đó bằng tay.

---

## 8. Kịch bản lệnh mẫu từng bước (Step-by-step CLI Walkthrough)

Dưới đây là bảng phân tích toàn bộ chuỗi lệnh minh họa từ bài giảng:

```cisco
! =======================================================
! 1. TỪ USER EXEC CHUYỂN SANG PRIVILEGED EXEC
! =======================================================
Router>
Router>e?
enable  exit

Router>enable
Router#

! =======================================================
! 2. DÙNG DẤU '?' VÀ VÀO GLOBAL CONFIGURATION MODE
! =======================================================
Router#?
Router#con?
configure  connect
Router#conf t?
terminal
Router#conf t
Router(config)#

! Thoát thử về Privileged EXEC
Router(config)#exit
Router#

! =======================================================
! 3. XEM VÀ LƯU FILE CẤU HÌNH
! =======================================================
! Xem cấu hình đang chạy trong RAM
Router#show running-config

! Xem cấu hình khởi động trong NVRAM
Router#show startup-config

! Các cách lưu cấu hình từ RAM sang NVRAM:
Router#write
Router#write memory
Router#copy running-config startup-config

! =======================================================
! 4. CẤU HÌNH BẢO MẬT MẬT KHẨU VÀ DÙNG LỆNH 'DO'
! =======================================================
Router#conf t
Router(config)#

! Bật mã hóa Type 7 cho toàn bộ mật khẩu dạng văn bản rõ
Router(config)#service password-encryption

! Đặt mật khẩu mã hóa cấp cao cho Privileged EXEC Mode
Router(config)#enable secret Cisco

! Dùng 'do' để kiểm tra running-config ngay khi đang ở Config Mode
Router(config)#do sh run

! Tắt tính năng tự động mã hóa mật khẩu cho các lệnh sau này
Router(config)#no service password-encryption
```
