# 1. Tại sao KGDB cần 2 máy?

* Debug kernel khác với debug user-space program, vì khi bạn dừng kernel thì **cả hệ thống target ngừng hoạt động**.
* Do đó cần **hai máy**:

  * **Host (Development machine)**: nơi bạn chạy `gdb`, dùng `vmlinux` (có symbol).
  * **Target (Debuggee machine)**: máy chạy kernel thật sự, có KGDB stub để nhận lệnh từ host.

---

# 2. Kết nối Host ↔ Target

Có hai cách kết nối chính:

## 2.1. Serial port (phổ biến nhất)

* KGDB hỗ trợ giao tiếp qua **UART/serial port**.
* Trong máy thật: cắm cáp **RS-232/USB-to-Serial** giữa host và target.
* Trong máy ảo (VirtualBox, VMware, QEMU): ta **giả lập cổng serial** và nối hai máy ảo với nhau.

## 2.2. Network (eth)

* Một số patch cho phép KGDB chạy qua Ethernet (`kgdboe` – KGDB over Ethernet).
* Nhưng không luôn có sẵn trong mainline, nên thường dùng serial cho chắc chắn.

---

# 3. Cấu hình khi dùng VirtualBox

Bạn có thể dùng **hai VM** (một làm host, một làm target).

## 3.1. Network Settings

* **Bridged Adapter**: cho phép VM tham gia cùng mạng với host → dễ dàng giao tiếp.
* **Promiscuous Mode**: cho phép VM “nghe” tất cả traffic trên mạng (cần nếu debug qua Ethernet).

## 3.2. Serial Pipe Settings

* VirtualBox hỗ trợ “Virtual Serial Port” → ta cấu hình như sau:

  * **Target VM**: enable serial port, chọn “Host Pipe”.
  * **Host VM**: cũng enable serial port, chọn “Host Pipe”, cùng tên pipe với Target.
* Như vậy, pipe này mô phỏng kết nối serial cable giữa hai máy.

Ví dụ (Target VM settings):

```
Enable Serial Port: COM1
Port Mode: Host Pipe
Path: /tmp/vboxserial
```

Host VM cũng cấu hình tương tự để kết nối vào `/tmp/vboxserial`.

---

# 4. Hình dung sơ đồ

```
          ┌───────────────┐                     ┌───────────────┐
          │    Host VM    │                     │   Target VM   │
          │  (Development)│                     │   (Debuggee)  │
          │               │                     │               │
          │ gdb + vmlinux │ <==== Serial Pipe ===> │ Linux Kernel │
          │               │                     │ + KGDB stub   │
          └───────────────┘                     └───────────────┘
```

# 5. Ý nghĩa

* **Serial port** ở đây đóng vai trò như “đường dây hotline” giữa gdb (host) và kernel (target).
* Khi bạn gõ `breakpoint`, `step` trong gdb → lệnh sẽ truyền qua serial → KGDB trong kernel xử lý.

---

👉 Tóm gọn:

* Cần **hai máy (hoặc hai VM)**.
* Kết nối qua **serial port (Host Pipe)** hoặc **Ethernet (bridged adapter + promiscuous mode)**.
* Host chạy `gdb` với `vmlinux`.
* Target chạy kernel có bật KGDB.