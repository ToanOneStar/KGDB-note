# Khởi động Target với Hỗ trợ KGDB

Sau khi kernel đã được biên dịch với **KGDB support**, bạn cần bật nó khi khởi động.  
Theo mặc định, nếu kernel có KGDB nhưng **không** được bật bằng tham số dòng lệnh, KGDB sẽ **không hoạt động**.

---

## Cách bật KGDB khi khởi động

KGDB được bật thông qua **kernel command line** (dòng lệnh khởi động kernel).  
Điều này được cấu hình trong **GRUB**.

Thêm các tham số sau vào dòng lệnh khởi động của kernel trong GRUB:

```text
kgdbwait kgdboc=ttyS0,115200
```

---

## Giải thích các tham số

- **kgdboc** (KGDB over console)  
  Đây là driver I/O của KGDB. Nó nhận 2 đối số:  
  - `ttyS0` → chỉ định sử dụng cổng serial `ttyS0` để giao tiếp với GDB.  
  - `115200` → tốc độ baud rate cho cổng serial.  

- **kgdbwait**  
  Yêu cầu kernel **dừng lại rất sớm trong quá trình boot** tại một breakpoint đặc biệt,  
  chờ đến khi GDB từ máy phát triển kết nối vào target qua KGDB.  

---

## Quy trình khởi động với KGDB

1. Cài đặt kernel có bật KGDB support trên máy target.  
2. Sửa cấu hình GRUB để thêm tham số khởi động:  

   Mở file `/etc/default/grub` và chỉnh dòng:

   ```bash
   GRUB_CMDLINE_LINUX="kgdbwait kgdboc=ttyS0,115200"
   ```

3. Cập nhật GRUB:

   ```bash
   sudo update-grub
   ```

4. Khởi động lại máy target.  
   Kernel sẽ dừng sớm trong quá trình boot và chờ kết nối GDB từ máy phát triển.  

---

## Kết nối từ máy phát triển

Trên máy phát triển (development machine), dùng `gdb` với file `vmlinux` đã build:

```bash
gdb vmlinux
```

Rồi kết nối tới target qua serial:

```gdb
target remote /dev/ttyS0
```

---

## Kết luận

Với cấu hình trên, target sẽ dừng ngay khi khởi động và chờ GDB kết nối, cho phép bạn gỡ lỗi kernel từ rất sớm trong quá trình boot.  