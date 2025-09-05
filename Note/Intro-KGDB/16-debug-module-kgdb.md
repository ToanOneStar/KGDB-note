# Debug module trên Raspberry Pi Zero W bằng KGDB
Trong bài này ta sẽ viết một module bằng C và insmod kernel trên Raspberry Pi Zero W, sau đó tiến hành debug trên userspace bằng KGDB.
# 1. Chương trình module
Ta có chương trình module kernel viết bằng C:
```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/debugfs.h>
#include <linux/uaccess.h>

#define DRV_NAME "safe_debug"

static struct dentry *dir, *file;
static int counter = 0;

/**
 * safe_read - Safely read the value of the counter and return it to userspace
 *
 * This function formats the global variable `counter` into a string of the form
 * "counter=<value>\n" and copies the data into the userspace buffer using
 * `simple_read_from_buffer()`. It ensures the read is bounded by the requested
 * length and the internal buffer size.
 *
 * @f:     [in] Pointer to the file structure (unused in this function).
 * @ubuf:  [out] Userspace buffer where the formatted counter string is copied.
 * @len:   [in] Maximum number of bytes requested by userspace.
 * @ppos:  [in,out] Current file offset, updated after the read operation.
 *
 * Return: The number of bytes successfully copied to userspace (>= 0),
 *         or a negative error code if the operation fails (e.g. -EFAULT).
 */
static ssize_t safe_read(struct file *f, char __user *ubuf, size_t len, loff_t *ppos) {
    char buf[64];
    int n;

    n = snprintf(buf, sizeof(buf), "counter=%d\n", counter);
    return simple_read_from_buffer(ubuf, len, ppos, buf, n);
}

/**
 * safe_write - Safely write data from userspace and update the counter
 *
 * This function copies a string from userspace into a temporary kernel buffer,
 * ensures it is null-terminated, and logs the received string using `pr_info()`.
 * After a successful write, the global variable `counter` is incremented.
 *
 * The function also limits the maximum number of bytes copied to prevent buffer
 * overflow, ensuring at most (sizeof(buf) - 1) characters are stored.
 *
 * @f:     [in] Pointer to the file structure (unused in this function).
 * @ubuf:  [in] Userspace buffer containing the string to be written.
 * @len:   [in] Number of bytes requested to be written by userspace.
 * @ppos:  [in,out] Current file offset, not used in this function but updated
 *                  as required by the file operation interface.
 *
 * Return: The number of bytes successfully written (>= 0),
 *         or a negative error code if the operation fails (e.g. -EFAULT).
 */
static ssize_t safe_write(struct file *f, const char __user *ubuf, size_t len, loff_t *ppos) {
    char buf[32];

    if (len >= sizeof(buf))
        len = sizeof(buf) - 1;
    if (copy_from_user(buf, ubuf, len))
        return -EFAULT;
    buf[len] = '\0';

    pr_info(DRV_NAME ": write '%s'\n", buf);
    counter++;

    return len;
}

static const struct file_operations fops = {
    .owner = THIS_MODULE,
    .read  = safe_read,
    .write = safe_write,
    .llseek = no_llseek,
};

/**
 * safe_init - Module initialization function
 *
 * This function is executed when the module is loaded. It creates a directory
 * under debugfs named after the driver (`DRV_NAME`) and registers a file named
 * "trigger" inside that directory. The file is associated with the file
 * operations defined in `fops`.
 *
 * If directory or file creation fails, all previously created debugfs entries
 * are removed, and the function returns `-ENOMEM`.
 *
 * On success, a kernel log message is printed with instructions on how to use
 * the "trigger" file.
 *
 * Return: 0 on success,
 *         -ENOMEM if allocation or debugfs creation fails.
 */
static int __init safe_init(void) {
    dir = debugfs_create_dir(DRV_NAME, NULL);
    if (!dir)
        return -ENOMEM;

    file = debugfs_create_file("trigger", 0666, dir, NULL, &fops);
    if (!file) {
        debugfs_remove_recursive(dir);
        return -ENOMEM;
    }

    pr_info(DRV_NAME ": loaded. echo sth > /sys/kernel/debug/%s/trigger\n", DRV_NAME);
    return 0;
}

/**
 * safe_exit - Module cleanup function
 *
 * This function is executed when the module is unloaded. It removes the debugfs
 * directory and all files previously created by `safe_init()`. After cleanup,
 * a kernel log message is printed to indicate that the module has been unloaded.
 *
 * This ensures no debugfs entries remain after the module is removed.
 */
static void __exit safe_exit(void) {
    debugfs_remove_recursive(dir);
    pr_info(DRV_NAME ": unloaded\n");
}

module_init(safe_init);
module_exit(safe_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("You");
MODULE_DESCRIPTION("Safe debug demo module for KGDB");
```

Ta có file `Makefile`:
```make
obj-m := kgdb-ras.o

all:
	$(MAKE) -C $(KERNEL_SRC) M=$(PWD) modules

clean:
	$(MAKE) -C $(KERNEL_SRC) M=$(PWD) clean

# Thêm mục tiêu modules_install
install: modules_install

modules_install:
	$(MAKE) -C $(KERNEL_SRC) M=$(PWD) modules_install
```

File `kgdb-raspberrypi.bb`:
```bash
SUMMARY = "A simple GPIO kernel module"
LICENSE = "GPL-2.0-only"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://kgdb-ras.c \
           file://Makefile"

S = "${WORKDIR}"

inherit module

EXTRA_OEMAKE:append:class-target = " KERNEL_SRC=${STAGING_KERNEL_DIR}"
KERNEL_MODULE_PROBE = "kgdb-ras"
```

Build module:
```bash
cd yocto/poky
source oe-init-build-env
bitbake kgdb-raspberrypi
```
# 2. Debugging
Run `agent-proxy` trên host:
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


Sau đó chuyển tới thư mục standard build để khởi động GDB:
```bash
cd /home/toanonestar/yocto/poky/build/tmp/work/raspberrypi0_wifi-poky-linux-gnueabi/linux-raspberrypi/1_5.4.72+gitAUTOINC+5d52d9eea9_154de7bbd5-r0/linux-raspberrypi0_wifi-standard-build/
$GDB vmlinux
```
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

