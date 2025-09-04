# KGDB on Raspberry Pi Zero W

Hướng dẫn setup và sử dụng KGDB để debug kernel trên Raspberry Pi Zero W bằng Yocto Project.

---

## 1. Build Yocto image với KGDB kernel config

```bash
bitbake -c menuconfig virtual/kernel
````

Bật các tùy chọn sau trong kernel config:

* `CONFIG_DEBUG_INFO`
* `CONFIG_KGDB`
* `CONFIG_GDB_SCRIPTS`

Thêm gói `gdbserver` vào image:

```conf
IMAGE_INSTALL_append = " gdbserver"
```

Sau đó build image:

```bash
bitbake core-image-minimal
```

---

## 2. Build và cài SDK (dùng cho gdb)

Build toolchain SDK:

```bash
bitbake meta-toolchain
```

Sau khi hoàn tất, vào thư mục `tmp/deploy/sdk` và chạy script installer:

```bash
./poky-glibc-x86_64-meta-toolchain-arm1176jzfshf-vfp-raspberrypi0-wifi-toolchain-3.1.33.sh
```

Source môi trường cross:

```bash
source /opt/poky/3.1.33/environment-setup-arm1176jzfshf-vfp-poky-linux-gnueabi
```

Kiểm tra:

```bash
echo $GDB
```

---

## 3. Flash image lên thẻ nhớ Raspberry Pi Zero W

Sử dụng `dd` hoặc `bmaptool` để ghi image đã build vào thẻ nhớ SD, sau đó boot Pi.

---

## 4. Kết nối serial port

Cắm dây USB-to-TTL (UART) vào Raspberry Pi Zero W, đầu còn lại cắm vào host/VM.
Trên host, xác định cổng `/dev/ttyUSB0`.

Chú ý kiểm tra đã kết nối thành công hay chưa bằng lệnh ```sudo dmesg```:

![kgdb-ras](https://toanonestar.github.io/KGDB-note/image-scp/kgdb-ras.png)

Nếu bị lỗi disconnect thì cần gỡ bở gói Braille chiếm dụng cổng USB0:
```bash
sudo apt remove brltty
```
Kết nối minicom:
```bash
sudo minicom -D /dev/ttyUSB0 -b 115200
```

---
## 5. Build và chạy agent-proxy

Clone và build `agent-proxy`. Sau đó chạy:

```bash
sudo ./agent-proxy 5550^5551 0 /dev/ttyUSB0,115200
```

Truy cập console Raspberry Pi qua telnet

```bash
telnet localhost 5550
```
![5550](https://toanonestar.github.io/KGDB-note/image-scp/5550.png)


---

## 6. Lấy workdir của kernel trong Yocto build

```bash
bitbake -e virtual/kernel | grep ^WORKDIR=
```
Sau đó chuyển tới thư mục standard build và sử dụng ```$GDB vmlinux```:

![kgdb-ras2](https://toanonestar.github.io/KGDB-note/image-scp/kgdb-ras2.png)

---

## 7. Cấu hình KGDB trên Raspberry Pi

Trên target (Pi):

```bash
echo 'ttyS0,115200' > /sys/module/kgdboc/parameters/kgdboc
echo g > /proc/sysrq-trigger
```

---

## 8. Kết nối GDB từ host

Mở GDB với vmlinux:

```bash
$GDB tmp/work/raspberrypi0_wifi-poky-linux-gnueabi/linux-raspberrypi/*/linux-raspberrypi0_wifi-standard-build/vmlinux
```

Trong gdb để list source code thì ta sử dụng ```set substitute-path``` với đường dẫn thường nằm trong ```/home/toanonestar/yocto/poky/build/tmp/work-shared/raspberrypi0-wifi/kernel-source``` :

```gdb
(gdb) set substitute-path /usr/src/kernel /home/toanonestar/yocto/poky/build/tmp/work-shared/raspberrypi0-wifi/kernel-source
(gdb) target remote localhost:5551
```
![kgdb-ras3](https://toanonestar.github.io/KGDB-note/image-scp/kgdb-ras3.png)
---

## Notes

* `agent-proxy` giúp chia serial thành 2 cổng TCP: một cho console, một cho gdb.
* Đảm bảo Pi boot được với image có KGDB enabled.
* Debug kernel khác với user-space: bạn cần symbols từ `vmlinux` (không bị strip).
