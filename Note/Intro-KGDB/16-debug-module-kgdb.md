Trên target (Pi):

```bash
echo 'ttyS0,115200' > /sys/module/kgdboc/parameters/kgdboc
echo g > /proc/sysrq-trigger
```

Trên máy host:
```bash
(gdb) target remote localhost:5551
```
