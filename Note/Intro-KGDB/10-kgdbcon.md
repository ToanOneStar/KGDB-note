kgdbcon
--------

`kgdbcon` cho phép bạn xem các thông điệp `printk` ngay bên trong **gdb**, trong khi gdb đang kết nối với máy target.

### Cách kích hoạt

Có hai cách để bật tính năng này:

1. **Bằng tham số dòng lệnh kernel**:

   ```
   kgdbcon
   ```

2. **Bằng sysfs trước khi cấu hình driver I/O**:

   ```bash
   echo 1 > /sys/module/debug_core/parameters/kgdb_use_con
   ```

   > Lưu ý: Nếu bạn thực hiện sau khi đã cấu hình driver I/O cho kgdb, thiết lập này sẽ **không có hiệu lực** cho đến khi I/O được cấu hình lại.

---

### Quan trọng
Bạn **không thể** sử dụng `kgdboc` + `kgdbcon` trên một tty đang được dùng làm **console hệ thống**.  

Ví dụ **sai**:

```
console=ttyS0,115200 kgdboc=ttyS0 kgdbcon
```