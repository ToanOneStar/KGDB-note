# Using Linux Kernel GDB Scripts

Linux kernel cung cấp một tập hợp **GDB helper scripts** giúp đơn giản hóa việc debug kernel.  
Trong cây source Linux, bạn sẽ thấy file `vmlinux-gdb.py`. Các script này cung cấp nhiều lệnh tiện dụng (`lx-*`) để phân tích kernel trong GDB.

---

## 1. Bật hỗ trợ GDB Scripts trong Kernel

Trước hết, cần bật option sau trong kernel config:

```
CONFIG_GDB_SCRIPTS=y
```

Sau khi build kernel thành công, trong thư mục build sẽ có:

- `vmlinux` (file symbol)
- `scripts/gdb/linux/*.py` (các helper scripts)

---

## 2. Cấu hình GDB để load script

Mặc định, GDB có cơ chế bảo mật **auto-load safe path**. Nếu bạn chạy:

```bash
gdb vmlinux
```

Bạn có thể gặp cảnh báo kiểu:

```
warning: File "…/scripts/gdb/vmlinux-gdb.py" auto-loading has been declined by your `auto-load safe-path'
```

### Cách fix
Tạo file `~/.config/gdb/gdbinit` (hoặc `~/.gdbinit`) và thêm một trong hai dòng sau:

- **Chỉ cho phép kernel scripts**:
  ```gdb
  add-auto-load-safe-path /home/toanonestar/linux-6.6.102/scripts/gdb/vmlinux-gdb.py
  ```

- **Hoặc mở toàn bộ (ít an toàn nhưng nhanh gọn)**:
  ```gdb
  set auto-load safe-path /
  ```

---

## 3. Xử lý lỗi `No module named linux.constants`

Khi chạy GDB có thể gặp lỗi:

```
ModuleNotFoundError: No module named 'linux.constants'
```

Điều này xảy ra vì file `constants.py` chưa được sinh ra.  
Cách khắc phục:

```bash
make scripts_gdb
```

Lệnh này sẽ tạo ra `scripts/gdb/linux/constants.py` từ `constants.py.in`.

---

## 4. Kiểm tra script hoạt động

Chạy lại:

```bash
gdb vmlinux
```

Trong gdb:

```gdb
apropos lx
```

Nếu thấy danh sách lệnh `lx-*` → script đã được load thành công.

---

## 5. Một số lệnh hữu ích

- Xem kernel command line:
  ```gdb
  (gdb) lx-cmdline
  ```
- Liệt kê tiến trình:
  ```gdb
  (gdb) lx-ps
  ```
- Liệt kê modules đã load:
  ```gdb
  (gdb) lx-lsmod
  ```
- Xem phiên bản kernel:
  ```gdb
  (gdb) lx-version
  ```
- Xem trợ giúp từng lệnh:
  ```gdb
  (gdb) help lx-ps
  ```

---

## 6. Troubleshooting (FAQ)

### ❌ Gặp warning: auto-loading has been declined
- Nguyên nhân: GDB block auto-load scripts vì lý do bảo mật.
- Fix: thêm `add-auto-load-safe-path` hoặc `set auto-load safe-path /` vào `~/.gdbinit`.

### ❌ Lỗi `ModuleNotFoundError: No module named 'linux.constants'`
- Nguyên nhân: chưa sinh file constants.py.
- Fix: chạy `make scripts_gdb` trong thư mục kernel source.

### ❌ Trên máy dev không có `vmlinux-gdb.py`
- Nguyên nhân: chỉ copy `vmlinux` từ target sang, thiếu scripts.
- Fix: copy cả thư mục `scripts/gdb/` kèm theo `vmlinux`.

### ❌ Dùng `apropos lx` không thấy lệnh nào
- Nguyên nhân: script chưa load được vào GDB.
- Fix: kiểm tra lại `~/.gdbinit`, đường dẫn phải trỏ đúng file `vmlinux-gdb.py`.

---

✅ Với các bước trên, bạn đã có thể sử dụng đầy đủ **Linux GDB helper scripts** để debug kernel.