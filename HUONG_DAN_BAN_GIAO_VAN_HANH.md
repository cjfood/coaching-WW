# ⚡ HƯỚNG DẪN CẬP NHẬT BÁO CÁO COACHING GSBH
* **Thư mục làm việc:** `D:\CJ\11. Adhoc\T8\WW`
* **Link xem báo cáo Online:** https://cjfood.github.io/coaching-WW/

---

### 1️⃣ BƯỚC 1: Bỏ file dữ liệu mới vào thư mục
* Tải file báo cáo công việc từ DMS/SFA (đuôi `.xlsx` hoặc `.xlsb`).
* Copy và dán file vào thư mục: **`raw_data/`**
> *(Lưu ý: Không xóa các file tháng cũ trong thư mục này).*

---

### 2️⃣ BƯỚC 2: Thêm nhân sự mới (Nếu có)
* Mở file: **`DS GSBH.xlsx`**
* Thêm dòng mới: Điền Mã GSBH, Tên GSBH, Miền, Vùng -> Bấm **Save (Ctrl + S)** và đóng file.
> *(Nếu tháng này không đổi GSBH thì BỎ QUA bước này).*

---

### 3️⃣ BƯỚC 3: Chạy cập nhật tự động
* Nhấp đúp chuột vào file: **`Cap_Nhat_Bao_Cao.bat`**
* Hệ thống sẽ tự tính toán KPI và tự đẩy lên GitHub (khoảng 15 giây).
* Khi thấy dòng chữ **`[SUCCESS] Đã đồng bộ thành công lên GitHub!`** -> Bấm phím bất kỳ để tắt.

---

### 4️⃣ BƯỚC 4: Kiểm tra kết quả
* Mở link web: **https://cjfood.github.io/coaching-WW/**
* Nhìn góc trên bên phải xem mục **"Cập nhật lần cuối"** đã nhảy đúng giờ vừa chạy chưa.
> *(Nếu chưa đổi số, bấm phím `Ctrl + F5` trên bàn phím để làm mới web).*
