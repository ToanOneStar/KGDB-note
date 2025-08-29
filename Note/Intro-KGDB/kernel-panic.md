# Kernel Panic & Debugging with GDB

## Khi nào kernel panic xảy ra?
Kernel panic xảy ra khi nhân Linux gặp lỗi nghiêm trọng không thể phục hồi:
- Bug trong code kernel hoặc driver (NULL pointer, invalid memory access).
- Deadlock hoặc lỗi đồng bộ.
- Filesystem corruption (không mount được rootfs).
- Hardware error từ CPU/thiết bị.
- Kernel chủ động gọi `panic()` khi phát hiện tình huống không thể xử lý.

## Hàm panic() và các trường hợp khác
- Hàm `panic(fmt, ...)` được gọi trực tiếp trong kernel để ngừng hệ thống.
- Ngoài `panic()`, còn có:
  - `BUG()` hoặc `BUG_ON()` → gây panic nếu điều kiện xảy ra.
  - Oops hoặc trap nghiêm trọng mà kernel không thể recover.
  - Lỗi lớn trong scheduler hoặc memory management.

## Khi Panic xảy ra, bạn có thể làm gì?
Khi panic xuất hiện, kernel tự động vào chế độ debugger. Tại đây bạn có thể:
- Xem và phân tích bộ nhớ.
- Thực hiện traceback (backtrace).
- Kiểm tra các thanh ghi.
- Chạy các lệnh gdb khác để hiểu rõ trạng thái hệ thống.

## Debug panic bằng GDB
Nếu KGDB đã bật (`kgdbwait kgdboc=ttyS0,115200`):
1. Kết nối từ host:
   ```bash
   gdb vmlinux
   target remote /dev/ttyS0
   ```
2. Xem stack trace:
   ```gdb
   bt
   ```
3. Xem source code tại vị trí panic:
   ```gdb
   list *$pc
   ```
4. Kiểm tra thanh ghi:
   ```gdb
   info registers
   ```
5. Dump stack memory hoặc biến:
   ```gdb
   x/32x $sp
   p variable_name
   ```
6. Đặt breakpoint vào panic cho các lần sau:
   ```gdb
   b panic
   continue
   ```

## Ý nghĩa
Việc phân tích panic bằng GDB giúp hiểu rõ nguyên nhân gốc rễ, rút ngắn thời gian fix bug trong kernel và driver.