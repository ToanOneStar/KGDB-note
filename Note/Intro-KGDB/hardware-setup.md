Test on 2 Virtual machines

Dùng **VMware Workstation** và mở phần **Serial Port settings** .
Để cho **hai VM (Host VM & Target VM)** giao tiếp với nhau qua serial (dùng cho KGDB), bạn cần cấu hình “Named Pipe” ở cả hai máy.

---

## 1. Cách setup Serial Port giữa 2 máy ảo VMware

Giả sử:

* **VM1 = Development (chạy gdb)**
* **VM2 = Target (chạy kernel có KGDB)**

### 1.1. Thêm Serial Port cho cả hai VM

* Vào **VM settings → Add → Serial Port** nếu chưa có.
* Chọn **Use named pipe**.

### 1.2. Cấu hình Named Pipe giống nhau

Ở **VM1 (Development)**:

* Chọn **Use named pipe**.
* Điền tên pipe, ví dụ:

  ```
  \\.\pipe\kgdb
  ```
* Chọn **This end is the server**.
* Chọn **The other end is a virtual machine**.

![dev](https://toanonestar.github.io/KGDB-note/image-scp/dev.png)

Ở **VM2 (Target)**:

* Chọn **Use named pipe**.
* Điền **đúng cùng tên pipe**:

  ```
  \\.\pipe\kgdb
  ```
* Chọn **This end is the client**.
* Chọn **The other end is a virtual machine**.

![target](https://toanonestar.github.io/KGDB-note/image-scp/target.png)

📌 Lưu ý: Một máy là **server**, máy kia là **client**, nhưng cả hai dùng cùng đường pipe.

### Bước 3. Bật tùy chọn (không bắt buộc)

* Bạn có thể tick **Yield CPU on poll** để tránh chiếm CPU khi poll serial.

---

## 🔹 Hình dung kết nối

```
VM1 (Development, gdb)    <==== Serial over Named Pipe ====>
VM2 (Target, kernel+KGDB)
```

Cổng COM trong VM bây giờ mô phỏng như dây serial thật, nối trực tiếp hai máy lại với nhau.

---

👉 Sau khi setup xong, trong kernel (VM2) bạn bật KGDB với boot param, ví dụ:

```
kgdboc=ttyS0,115200
```

Còn trong VM1, bạn mở gdb với `vmlinux` và connect:

```
(gdb) target remote /dev/ttyS0
```

