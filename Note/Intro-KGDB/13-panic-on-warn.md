# Setting variables trong Kernel Debug

Khi debug kernel bằng GDB, ta có thể thay đổi hành vi của kernel bằng cách gán giá trị cho các biến điều khiển.  
Cách thực hiện trong GDB:

```
set var=<value>
```

Ví dụ:
```
(gdb) set panic_on_oops=1
```

---

## panic_on_oops
`panic_on_oops` quyết định hành vi của kernel khi gặp **Oops** hoặc **BUG**.

- `0`: Kernel **cố gắng tiếp tục chạy** sau khi gặp Oops/BUG. (mặc định)  
- `1`: Kernel sẽ **panic ngay lập tức**.

Ví dụ trong GDB:
```
(gdb) p panic_on_oops
(gdb) set panic_on_oops=1
(gdb) p panic_on_oops
```

### Tại sao cần set = 1?
- Nếu để `0`, kernel sẽ tiếp tục chạy sau khi gặp lỗi, dẫn tới trạng thái **không ổn định**.  
- Trong chế độ debug với GDB, nếu kernel tiếp tục chạy, bạn có thể **không thấy được trạng thái chính xác của lỗi** tại thời điểm xảy ra Oops → khó phân tích.  
- Khi set = `1`, kernel sẽ **panic ngay khi gặp lỗi**, giữ nguyên trạng thái hệ thống để bạn có thể dùng GDB phân tích **nguyên nhân gốc**.

---

## panic_on_warn
`panic_on_warn` quyết định hành vi khi kernel gặp macro `WARN()`.

- `0`: Chỉ in ra cảnh báo (`WARN()`), tiếp tục chạy. (mặc định)  
- `1`: Sau khi in cảnh báo, kernel gọi `panic()`.  

Ví dụ trong GDB:
```
(gdb) p panic_on_warn
(gdb) set panic_on_warn=1
(gdb) p panic_on_warn
```

### Tại sao cần set = 1?
- Nếu để `0`, kernel chỉ in cảnh báo và tiếp tục chạy → bạn **không thể dừng ngay tại lỗi** để debug.  
- Khi set = `1`, kernel sẽ panic ngay khi có cảnh báo, giúp GDB **bắt dừng hệ thống đúng lúc** và phân tích chính xác hơn.  

---

## Kết luận
- Việc set `panic_on_oops=1` và `panic_on_warn=1` rất quan trọng khi debug kernel với GDB.  
- Nó đảm bảo kernel sẽ **panic ngay lập tức khi có lỗi hoặc cảnh báo**, giúp ta có thể phân tích trạng thái chính xác tại thời điểm sự cố.  
- Nếu không set, kernel có thể tiếp tục chạy ở trạng thái hỏng → gây sai lệch kết quả debug.
