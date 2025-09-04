# agent-proxy

## Giới thiệu
Khi debug kernel bằng **KGDB** qua serial, thường bạn cần cả **console** (để xem log, đăng nhập shell) và **GDB** (để điều khiển kernel debugger).  
Nếu target chỉ có **một cổng serial** (ví dụ `/dev/ttyUSB0`), bạn sẽ gặp xung đột: chỉ một ứng dụng có thể mở serial tại một thời điểm.  

**Giải pháp**: `agent-proxy` là một công cụ nhỏ, đóng vai trò như **multiplexer** (bộ chia/ghép kênh) để nhiều client có thể cùng truy cập vào một serial device thông qua các cổng TCP.

---

## Cơ chế hoạt động

```
     -----------------
     | Target (UART) |
     -----------------
             |
             v
     -----------------
     |  agent-proxy  |
     -----------------
         /    |    \
        /     |     \
  TCP:5550 TCP:5551 TCP:5552 ...
   Console     GDB     Logger
```

- `agent-proxy` mở **1 kết nối duy nhất** tới serial (ví dụ `/dev/ttyUSB0,115200`).
- Sau đó, nó mở ra nhiều TCP port. Mỗi client có thể kết nối vào 1 port:
  - Console (qua telnet, minicom, …)  
  - GDB (qua `target remote`)  
  - Logger khác (tùy ý)

### Nguyên tắc truyền dữ liệu
- **Serial → Client**: mọi byte từ serial được **broadcast** tới tất cả client TCP đang kết nối.  
- **Client → Serial**: dữ liệu từ các client TCP được **multiplex** nối tiếp nhau xuống serial.  
- Vì serial vốn là **byte stream tuần tự**, không có xung đột bộ nhớ.  
- Nếu nhiều client cùng gửi dữ liệu, thứ tự byte phụ thuộc vào lúc agent-proxy nhận được. Trong thực tế, chỉ GDB gửi lệnh còn console chủ yếu đọc log, nên không gây lỗi.

---

## Xây dựng agent-proxy

```bash
git clone http://git.kernel.org/pub/scm/utils/kernel/kgdb/agent-proxy.git
cd agent-proxy
make
```

---

## Cách chạy

Ví dụ, chạy với một USB-serial device:

```bash
./agent-proxy 5550^5551 0 /dev/ttyUSB0,115200
```

- `5550^5551`: mở 2 cổng TCP (5550 và 5551).  
- `0`: bind trên `0.0.0.0` (localhost).  
- `/dev/ttyUSB0,115200`: thiết bị serial + tốc độ baud.

> ⚠️ Hai port TCP này **không có nhãn trước** là “console” hay “GDB”.  
> Bạn tự quyết định port nào dùng cho ứng dụng nào. Thông thường:
> - Port đầu tiên (5550) → console  
> - Port thứ hai (5551) → GDB  

---

## Sử dụng

### Console
```bash
telnet localhost 5550
```
Bạn sẽ thấy log kernel hoặc login shell.

### KGDB với GDB
```bash
gdb ./vmlinux
(gdb) target remote localhost:5551
```
- `vmlinux` là file kernel bạn build ra có chứa debug symbols.  
- Trên target, kernel phải được boot với tham số cmdline phù hợp:
  ```
  kgdboc=ttyS0,115200 kgdbwait
  ```

---

## Nhiều port hơn?
`agent-proxy` không giới hạn chỉ 2 port.  
Bạn có thể mở nhiều hơn, ví dụ:

```bash
./agent-proxy 5550^5551^5552 0 /dev/ttyUSB0,115200
```

- 5550: console  
- 5551: gdb  
- 5552: thêm một client logger  

Miễn là số port hợp lệ (1–65535, chưa bị chiếm).

---

## Tóm tắt
- `agent-proxy` cho phép **nhiều client** cùng truy cập một serial device.  
- Dữ liệu từ serial được broadcast tới tất cả client.  
- Dữ liệu từ client được multiplex nối tiếp xuống serial.  
- Thường dùng để kết hợp **console** và **GDB** khi debug kernel bằng KGDB mà chỉ có một cổng serial.  

## Kiểm tra port có đang rảnh không

Trước khi chọn số port để chạy với `agent-proxy`, bạn cần kiểm tra xem port đó có **đang bị chiếm dụng** (LISTEN bởi process nào khác) hay không.

Một số cách phổ biến:

---

### 1. Trên Linux (host hoặc target)

1. **Dùng `ss` (khuyên dùng)**

```bash
ss -ltnp
```

* `-l` = listening sockets
* `-t` = TCP
* `-n` = hiển thị số port thay vì tên dịch vụ
* `-p` = hiển thị process đang giữ port

Ví dụ output:

```
LISTEN 0      128       0.0.0.0:22      0.0.0.0:*     users:(("sshd",pid=712,fd=3))
LISTEN 0      128       127.0.0.1:5550  0.0.0.0:*     users:(("agent-proxy",pid=1234,fd=4))
```

→ Port **22** đang được sshd chiếm, port **5550** đang do agent-proxy chiếm.

---

2. **Dùng `netstat` (nếu cài)**

```bash
netstat -tuln | grep 5550
```

Nếu không có output → port 5550 đang rảnh.

---

3. **Dùng `lsof`**

```bash
lsof -i:5550
```

Nếu không có dòng nào → port 5550 chưa bị chiếm.

---

### 2. Script nhanh để tìm port rảnh

Ví dụ tìm port rảnh trong khoảng `5000–6000`:

```bash
for p in $(seq 5000 6000); do
    if ! ss -ltn | awk '{print $4}' | grep -q ":$p\$"; then
        echo "Port $p free"
        break
    fi
done
```

→ In ra port rảnh đầu tiên.

