gdbtui
------

Trong GDB, lệnh `list` có thể dùng để hiển thị mã nguồn, nhưng chỉ hiển thị các dòng lệnh xung quanh vị trí đang được thực thi. Điều này gây hạn chế khi bạn muốn quan sát mã nguồn, thanh ghi, và các thông tin khác cùng lúc.

### Giải pháp: gdbtui

**gdbtui** (GDB Text User Interface) là một giao diện dạng văn bản (terminal UI) của GDB, sử dụng thư viện `ncurses` để hiển thị nhiều thông tin cùng lúc trong các cửa sổ văn bản riêng biệt:

- Cửa sổ mã nguồn (source file)
- Cửa sổ hiển thị mã assembly
- Cửa sổ thanh ghi (registers)
- Cửa sổ lệnh GDB

Điều này giúp lập trình viên dễ dàng theo dõi quá trình debug hơn so với việc chỉ dùng lệnh `list`.

### Cách sử dụng
Khởi động GDB với tùy chọn `-tui`:
```bash
gdb --tui vmlinux
```

Trong quá trình debug, bạn có thể bật/tắt chế độ TUI bằng:
```bash
Ctrl + x a
```
Ta có giao diện như sau:
![gdbtui](https://toanonestar.github.io/KGDB-note/image-scp/gdbtui.png)

