# Kernel Panic và Debug với GDB

## Khi nào xảy ra Panic?
Kernel panic xảy ra khi hệ điều hành phát hiện lỗi nghiêm trọng không thể phục hồi, ví dụ:
- Lỗi truy cập bộ nhớ bất hợp pháp (NULL pointer dereference, invalid memory access).
- Deadlock hoặc lỗi đồng bộ nghiêm trọng.
- Filesystem corruption trong kernel space.
- Hardware error từ CPU/thiết bị.
- Gọi trực tiếp hàm `panic()` trong mã nguồn kernel hoặc module.

Ngoài hàm `panic()`, còn có những lỗi khác trong quá trình chạy kernel (như `BUG()`, `Oops`) cũng có thể dẫn tới panic.

---

## Ảnh hưởng của Kernel Panic
Khi kernel panic xảy ra, toàn bộ hệ điều hành sẽ ngừng hoạt động vì kernel là lõi quản lý mọi thứ.

Ảnh hưởng chính:
1. **Hệ thống bị treo (hang)**: không thể chạy thêm tiến trình nào.
2. **Mất kết nối**: mọi kết nối mạng như SSH sẽ bị ngắt.
3. **Mất dữ liệu chưa lưu**: dữ liệu trong RAM/buffer chưa ghi xuống disk sẽ mất.
4. **Không thể phục hồi tự động** (trừ khi cấu hình `panic=N` để kernel tự reboot sau N giây).

➡️ Panic gây ảnh hưởng toàn cục, không giống lỗi ứng dụng thông thường.

---

## Khi Panic xảy ra bạn có thể làm gì?
Khi panic, kernel sẽ dừng lại và nếu bật KGDB, bạn có thể:
- **Xem bộ nhớ** tại thời điểm panic.
- **Traceback (backtrace)** để biết call stack trước khi lỗi.
- **Xem giá trị thanh ghi CPU**.
- Thực hiện các lệnh GDB khác để tìm nguyên nhân.

---

## Debug Panic với GDB
Khi kernel panic và hệ thống dừng, GDB trên máy host (qua serial/network) có thể kết nối và thực hiện:
- `bt` : hiển thị call stack (backtrace).
- `info registers` : xem trạng thái CPU registers.
- `list` hoặc chế độ `gdbtui` để xem source code tại vị trí panic.
- Kiểm tra các biến, vùng nhớ, breakpoints để tìm nguyên nhân.

---

## Kết luận
- Panic xảy ra khi kernel gặp lỗi không thể phục hồi.
- Ảnh hưởng nghiêm trọng: hệ thống ngừng, mất kết nối, mất dữ liệu.
- KGDB + GDB là công cụ quan trọng để phân tích và khắc phục nguyên nhân panic.