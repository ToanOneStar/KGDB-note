# 1. Vấn đề khi debug kernel

* Với **user-space program** thì bạn có thể dùng `gdb` rất dễ dàng: đặt breakpoint, step, xem call stack, inspect memory…
* Nhưng **Linux kernel** lại không chạy trong user-space, nó chạy ở **ring 0 (kernel mode)**. Nghĩa là:

  * Không thể attach gdb trực tiếp vào kernel như một process bình thường.
  * Kernel lại điều khiển cả hệ thống, nên nếu debug sai có thể treo luôn máy.

👉 Bài toán đặt ra: *“Làm thế nào để dùng gdb (hoặc cách tương tự) để debug trực tiếp Linux kernel đang chạy?”*

---

# 2. Giải pháp: KGDB

* **KGDB (Kernel GNU Debugger)** là một **GDB server** được nhúng sẵn vào Linux kernel.
* Nó cho phép bạn dùng một máy khác (host) chạy `gdb` để điều khiển kernel trên target qua kết nối **serial** hoặc **network**.

Cấu trúc:

```
Host (dev machine)                       Target (debuggee)
---------------------------------------------------------------
gdb  <---- serial/ethernet ---->   KGDB stub (inside kernel)
vmlinux (có debug symbols)         Linux kernel đang chạy
```

---

# 3. KGDB giúp gì?

Với KGDB, bạn có thể thao tác **giống như debug một chương trình C bình thường**, nhưng ở cấp **kernel**:

* Đặt breakpoints trong code kernel (kể cả trong **system call handler** hay **interrupt handler**).
* Single-step qua từng dòng code trong kernel.
* Dừng/resume kernel execution.
* Xem và chỉnh sửa biến, thanh ghi, hoặc vùng nhớ kernel.
* Xem call stack của kernel thread.

---

# 4. Lợi ích thực tế

KGDB giải quyết vấn đề lớn nhất khi phát triển và gỡ lỗi kernel:

* **Driver development**: debug khi viết driver mới.
* **Kernel module debugging**: khi module bị crash, panic.
* **Nghiên cứu kernel internals**: hiểu cách kernel hoạt động.
* **Debug kernel panic**: thay vì chỉ xem backtrace, có thể attach gdb và step ngay trong kernel.

---

# 5. Yêu cầu và hạn chế

* Cần **2 máy** (hoặc ít nhất là target có cổng serial/ethernet nối ra).
* Host cần `vmlinux` (file kernel chưa strip symbol).
* Không thể dùng trên production system, vì kernel có thể bị dừng hẳn để debug.
* Hạn chế nếu debug thông qua network (chỉ có trong một số patch, không phải luôn mainline).



👉 Tóm gọn:
**KGDB là “cầu nối” cho phép bạn dùng gdb (ở host) để điều khiển và debug trực tiếp kernel (ở target). Nó giải quyết vấn đề là gdb vốn chỉ debug được user-space, nhưng không đụng tới kernel.**


# 6. vmlinux là gì?

* Khi bạn **build kernel Linux** từ source, kết quả build ra nhiều file. Trong đó:

  * **`vmlinux`** là **file ELF executable** chứa toàn bộ kernel với **debug symbols** (symbol table, function name, variable name…).
  * Nó **chưa được nén** và **chưa được strip**.
  * Dùng trong quá trình **debug** (ví dụ với `gdb`, `kgdb`, `perf`, `ftrace`, `objdump`...).

---

## 6.1. Khác biệt với các file kernel khác

* **`vmlinux`**:

  * Có đủ symbol và thông tin debug.
  * Thường không boot trực tiếp được trên phần lớn hardware.
  * Nằm trong thư mục build sau khi chạy `make`.

* **`zImage` / `bzImage`**:

  * Là **vmlinux đã được nén + kèm bootloader nhỏ** để có thể nạp vào RAM và chạy trên máy thật.
  * Đây mới là file kernel dùng để boot máy.

* **`Image`**:

  * Ở một số kiến trúc (ARM, RISC-V), `Image` là file nhị phân kernel chưa nén (có thể boot trực tiếp nếu bootloader hỗ trợ).

---

## 6.2. Vai trò của vmlinux

* Không dùng để boot kernel trong máy thật (trừ trường hợp đặc biệt).
* **Dùng để debug**:

  * Khi bạn chạy `gdb vmlinux`, gdb sẽ biết được symbol và mapping function name ↔ address.
  * KGDB, perf, ftrace, crash tool… cũng dựa vào vmlinux để phân tích.

Ví dụ debug với gdb:

```bash
gdb vmlinux
(gdb) b start_kernel
(gdb) target remote /dev/ttyS0
```

Ở đây `vmlinux` giúp gdb hiểu `start_kernel` nằm ở địa chỉ nào trong memory.

---

👉 Tóm gọn:
**`vmlinux` là bản kernel ELF chưa nén, có đầy đủ symbol, dùng chủ yếu cho mục đích debug và phân tích.**
Còn file thực tế để boot thường là `bzImage` hoặc `zImage`.

# 7. So sánh 4 “thực thể” thường thấy khi build kernel: `vmlinux`, `zImage`, `bzImage`, và `System.map`.

---

## 7.1. **`vmlinux`**

* **Loại file**: ELF Executable (chạy được trên máy có ELF loader).
* **Đặc điểm**:

  * Chứa đầy đủ **debug symbols** (function name, variable name, địa chỉ…).
  * Chưa được nén, chưa thêm bootloader stub.
* **Vai trò**:

  * Không boot trực tiếp trên đa số hệ thống.
  * Dùng để **debug** (với `gdb`, `kgdb`, `perf`, `crash`, `ftrace`...).

---

## 7.2. **`zImage`**

* **Loại file**: kernel image đã được **nén** bằng thuật toán cũ (gzip).
* **Đặc điểm**:

  * Dùng cho máy có bộ nhớ giới hạn.
  * Nạp bởi bootloader → giải nén → nạp `vmlinux` vào RAM.
* **Vai trò**:

  * Có thể boot trực tiếp (trên ARM thường thấy `zImage`).

---

## 7.3. **`bzImage`**

* **Loại file**: kernel image **big zImage** (nén, nhưng hỗ trợ kernel lớn hơn).
* **Đặc điểm**:

  * Tương tự `zImage`, nhưng dùng cho kernel lớn vượt giới hạn cũ (\~512 KB).
  * Phổ biến trên kiến trúc x86.
* **Vai trò**:

  * File thực tế mà bootloader (GRUB, Syslinux, LILO) dùng để boot Linux.

---

## 7.4. **`System.map`**

* **Loại file**: plain text, mapping từ **symbol name → memory address**.
* **Đặc điểm**:

  * Không phải binary.
  * Sinh ra trong quá trình build kernel.
* **Vai trò**:

  * Dùng để debug hoặc phân tích crash log (vd: “Oops” hoặc “kernel panic” báo địa chỉ PC).
  * Công cụ như `ksymoops`, `perf`, `crash` dùng `System.map` để dịch địa chỉ → tên hàm.

---

## 7.5. Tóm gọn mối quan hệ

```
vmlinux (ELF, đầy đủ symbol)
   │
   ├─> strip + compress → zImage (kernel nhỏ, nén gzip)
   │
   └─> strip + compress → bzImage (kernel lớn, thường dùng trên x86)
   
System.map (bảng symbol name → address, trích từ vmlinux)
```

* Bạn **boot kernel** bằng `bzImage` hoặc `zImage`.
* Bạn **debug kernel** bằng `vmlinux` (kèm `System.map`).

---

👉 Một cách dễ hiểu:

* **`vmlinux`** = “phiên bản đầy đủ” để debug.
* **`bzImage/zImage`** = “phiên bản rút gọn, nén” để boot.
* **`System.map`** = “bản danh bạ” chứa địa chỉ của tất cả symbol trong kernel.


