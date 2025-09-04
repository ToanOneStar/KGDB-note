# Gỡ lỗi Kernel với GDB (KGDB)

Tài liệu này hướng dẫn cách biên dịch và cấu hình Linux Kernel với hỗ trợ **KGDB**, và sử dụng GDB để kết nối, gỡ lỗi kernel thông qua serial hoặc mạng.

## Tổng quan

Quy trình gỡ lỗi Linux Kernel bằng GDB gồm 3 bước:

1. **Biên dịch kernel có bật hỗ trợ KGDB**  
2. **Cấu hình kernel trên máy đích (target) để chạy ở chế độ debug**  
3. **Dùng GDB trên máy phát triển (development) để kết nối với target qua serial hoặc mạng**

## Cấu hình & Biên dịch Kernel

### 1. Tải mã nguồn kernel

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.4.85.tar.xz
tar xvf linux-5.4.85.tar.xz
cd linux-5.4.85
```

### 2. Cài đặt gói cần thiết

Trên Debian/Ubuntu:

```bash
sudo apt update
sudo apt install build-essential dkms flex bison libssl-dev libelf-dev
```

### 3. Cấu hình kernel

Nạp lại cấu hình sẵn có:

```bash
make oldconfig
```

Mở menu cấu hình:

```bash
make menuconfig
```

Bật các tùy chọn sau:

- `CONFIG_KGDB` : **Kernel hacking → Kernel debugging → KGDB: kernel debugger**  
- `CONFIG_KGDB_SERIAL_CONSOLE` : Bật hỗ trợ console qua serial cho KGDB  
- `CONFIG_DEBUG_INFO` : Biên dịch kernel kèm thông tin debug  

**Tùy chọn thêm (khuyến nghị):**
- `CONFIG_FRAME_POINTER = y`  
  → Cho phép dựng lại stack trace chính xác hơn khi debug.  
- `CONFIG_DEBUG_RODATA = n`  
  → Nên tắt nếu kiến trúc hỗ trợ. Khi bật, kernel sẽ đánh dấu một số vùng nhớ là chỉ đọc → không thể đặt software breakpoint. (Nếu vẫn bật, bạn phải dùng hardware breakpoint trong GDB).

### 4. Biên dịch và cài đặt kernel

```bash
make -j$(nproc)
sudo make modules_install
sudo make install
```

## Triển khai Kernel

Sau khi build xong, copy thư mục kernel build sang **máy phát triển (development machine)** để sử dụng với GDB:

```bash
scp -r ~/linux-5.4.85 user@dev-machine:~/kernel-build
```

Trên **máy đích (target)**, reboot vào kernel vừa cài.

## Bước tiếp theo

- Cấu hình KGDB trên target (serial hoặc network).  
- Trên máy phát triển, dùng `gdb vmlinux` trong thư mục kernel build để kết nối tới target thông qua KGDB.  
