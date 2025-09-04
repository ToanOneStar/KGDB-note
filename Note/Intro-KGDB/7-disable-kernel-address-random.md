# Tắt KASLR với tùy chọn `nokaslr`

## KASLR là gì?

**KASLR** (Kernel Address Space Layout Randomization)  
là một kỹ thuật bảo mật được hệ điều hành triển khai nhằm **ngăn chặn kẻ tấn công đoán được địa chỉ của các hàm và vùng nhớ trong kernel**.  

Cơ chế này thay đổi vị trí nạp kernel trong bộ nhớ mỗi lần khởi động, giúp giảm nguy cơ tấn công khai thác lỗ hổng dựa trên địa chỉ cố định.

---

## Ảnh hưởng khi dùng KASLR trong quá trình gỡ lỗi

- Lệnh `backtrace` trong GDB sẽ hiển thị `??` thay vì tên hàm chính xác.  
- Không thể đặt breakpoint một cách chính xác do địa chỉ bị thay đổi ngẫu nhiên mỗi lần boot.

---

## Giải pháp: Tắt KASLR

Để thuận tiện cho việc **debug kernel**, ta cần **tắt KASLR**.  

Cách thực hiện: Thêm tham số `nokaslr` vào dòng lệnh kernel (kernel command line parameters).  

### Ví dụ (GRUB)

Mở file cấu hình GRUB:

```bash
sudo nano /etc/default/grub
```

Sửa dòng:

```bash
GRUB_CMDLINE_LINUX="nokaslr"
```

Cập nhật GRUB:

```bash
sudo update-grub
```

Khởi động lại hệ thống, kernel sẽ chạy mà không có KASLR.

---

## Kết luận

- **KASLR**: bảo mật tốt hơn nhưng gây khó khăn khi debug kernel.  
- **nokaslr**: vô hiệu hóa KASLR → giúp hiển thị đầy đủ backtrace và đặt breakpoint chính xác trong GDB.  