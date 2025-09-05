Khởi tạo `agent-proxy` trên host:
```bash
cd agent-proxy
sudo ./agent-proxy 5550^5551 0 /dev/ttyUSB0,115200
```

Mở console Pi trên host qua port 5550:
```bash
telnet localhost 5550
```

Khởi tạo biến môi trường:
```bash
cd /home/toanonestar/yocto/poky/build/tmp/deploy/sdk/
```
![environment](https://toanonestar.github.io/KGDB-note/image-scp/environment.png)

Trên target (Pi):

```bash
echo 'ttyS0,115200' > /sys/module/kgdboc/parameters/kgdboc
echo g > /proc/sysrq-trigger
```

Trên máy host:
```bash
(gdb) target remote localhost:5551
```

Kết quả sau khi kết nối thành công tới Pi:
![taget-remote](https://toanonestar.github.io/KGDB-note/image-scp/target-remote.png)

