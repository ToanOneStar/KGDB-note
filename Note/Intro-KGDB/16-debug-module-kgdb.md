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
