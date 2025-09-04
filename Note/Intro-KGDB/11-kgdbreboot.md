kgdbreboot
-----------

`kgdbreboot` là tham số kernel cho phép kiểm soát hành vi của KGDB khi hệ thống **reboot**.

## Các giá trị của `kgdbreboot`

- **0 (mặc định)**  
  Gửi thông báo "detach" tới gdb. GDB sẽ thoát phiên debug và in ra thông báo:  
  ```
  [Inferior 1 (Remote target) exited with code 01]
  ```

- **1**  
  GDB vẫn giữ kết nối và cung cấp cho bạn tùy chọn để debug trong quá trình reboot.

- **-1**  
  Không gửi thông báo nào tới gdb khi hệ thống reboot.

## Cách sử dụng

### 1. Thêm vào kernel command line
Trong cấu hình bootloader (ví dụ GRUB), thêm:  
```
kgdbreboot=<giá trị>
```

Ví dụ:
```
kgdbreboot=1
```

### 2. Thay đổi tại runtime bằng sysfs
Bạn có thể thay đổi giá trị mà không cần reboot kernel bằng cách ghi vào sysfs:

```bash
# Tắt thông báo gửi tới gdb
echo -1 > /sys/module/debug_core/parameters/kgdbreboot

# Trạng thái mặc định (detach gdb khi reboot)
echo 0 > /sys/module/debug_core/parameters/kgdbreboot

# Cho phép gdb tiếp tục debug khi reboot
echo 1 > /sys/module/debug_core/parameters/kgdbreboot
```

## Lưu ý
- Nếu bạn muốn debug khi hệ thống khởi động lại, hãy sử dụng `kgdbreboot=1`.  
- Nếu không muốn gdb nhận bất kỳ thông báo nào khi reboot, hãy dùng `kgdbreboot=-1`.  