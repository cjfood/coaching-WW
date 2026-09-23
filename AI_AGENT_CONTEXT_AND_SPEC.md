# 🚀 TÀI LIỆU DỰ ÁN & HƯỚNG DẪN DÀNH CHO AI AGENT (PROJECT CONTEXT & SPECIFICATION)
> **Dự án:** Hệ Thống Báo Cáo & Dashboard Đánh Giá Coaching / Work-With (WW) Giám Sát Bán Hàng (GSBH)  
> **Khách hàng / Đơn vị:** CJ Foods Việt Nam  
> **Mục tiêu tài liệu:** Cung cấp toàn bộ bối cảnh nghiệp vụ, kiến trúc kỹ thuật, các giải pháp kỹ thuật đặc thù (DRM Workaround, Smart Cache, Auto Sync GitHub) để bất kỳ AI Agent nào khi tiếp nhận dự án đều có thể hiểu 100% và tiếp tục phát triển mà không làm gãy hệ thống.

---

## 📌 PHẦN 1: BỐI CẢNH NGHIỆP VỤ FMCG & ROUTE-TO-MARKET (RTM)

### 1.1. Mô hình Phân phối & Vận hành
* **Đối tượng theo dõi:** Đội ngũ **GSBH (Giám sát bán hàng / Sales Supervisor - SS / SUP)** phụ trách kênh truyền thống (GT) và kênh hiện đại (MT) của CJ Foods trên toàn quốc.
* **Master Data:** Quản lý danh sách chuẩn **47 GSBH Master** phân chia theo 3 Miền chính (Miền Bắc, Miền Trung, Miền Nam) và các Vùng kinh doanh (Mekong, HCM, Đông Nam Bộ, v.v.).
* **Hoạt động cốt lõi (Coaching / Work-With):** 
  * GSBH có nhiệm vụ đồng hành (Work-With) trên tuyến bán hàng (Route / MCP) cùng Nhân viên bán hàng (Sales Rep - SR).
  * Mục tiêu: Đào tạo kỹ năng 7 bước bán hàng tại điểm bán, kiểm tra trưng bày (Merchandising), tỷ lệ thành công đơn hàng (Strike Rate), kiểm tra độ bao phủ (Coverage) và lắng nghe phản hồi từ Nhà phân phối (NPP) / Điểm bán (Outlet).
* **Tiêu chuẩn KPI:**
  * Số ngày đi thị trường chuẩn: Tối thiểu 16 - 18 ngày/tháng thực tế ngoài thị trường.
  * Tỷ lệ tuân thủ lịch kế hoạch vs thực tế.
  * Phân loại công việc: Coaching thực địa, Họp nội bộ, Tuyển dụng/Đào tạo, Giải quyết vấn đề NPP/Đơn hàng.

---

## 🏗️ PHẦN 2: KIẾN TRÚC TỔNG THỂ CỦA DỰ ÁN

```
+---------------------------------------------------------------------------------+
|                                1. NGUỒN DỮ LIỆU                                |
|  - DMS / SFA Export: raw_data/*.xlsb, *.xlsx (Mã hóa DRMONE của CJ Foods)      |
|  - Danh sách Master: DS GSBH.xlsx (Backup: ds_gsbh_master.json)                 |
+---------------------------------------+-----------------------------------------+
                                        |
                                        v
+---------------------------------------------------------------------------------+
|                       2. ENGINE XỬ LÝ (coaching_report.py)                       |
|  [A] Smart Cache (.cache/): So khớp hash file thô -> Bỏ qua file cũ (0.01s)    |
|  [B] Excel COM Reader: win32com mở file DRMONE, tự unblock & thoát ProtectedView |
|  [C] Data Cleaning & Standardization: Chuẩn hóa Unicode, ngày tháng, mã NV      |
|  [D] Business KPI Engine: Tính toán số ngày đi tuyến, tỷ lệ tuân thủ, lịch sử   |
|  [E] Export: coaching_data.js (JSON gắn vào window.COACHING_DATA)               |
+---------------------------------------+-----------------------------------------+
                                        |
                                        v
+---------------------------------------------------------------------------------+
|                       3. DASHBOARD FRONTEND (index.html)                        |
|  - Single Page Application (SPA), chạy trực tiếp không cần web server           |
|  - Framework / Library: TailwindCSS (CDN), Chart.js, FontAwesome, ExcelJS       |
|  - Cache-Busting: Dynamic script injection '?v=' + timestamp tránh browser cache|
|  - Bộ lọc đa chiều: Tháng (T7, T8, T9...), Miền, Vùng, GSBH                     |
+---------------------------------------+-----------------------------------------+
                                        |
                                        v
+---------------------------------------------------------------------------------+
|                   4. ĐỒNG BỘ TỰ ĐỘNG GITHUB (push_to_github.py)                 |
|  - GitHub REST API v3 (Branch 'main', Repo 'cjfood/coaching-WW')                |
|  - KHÔNG cần cài Git CLI trên máy tính người dùng                               |
|  - Git Blob SHA Diffing: Chỉ tải file có nội dung thay đổi                      |
|  - Tự động bắt lỗi 401 Unauthorized và cho phép nhập token mới tại chỗ          |
|  - Host trực tuyến: https://cjfood.github.io/coaching-WW/                      |
+---------------------------------------------------------------------------------+
```

---

## ⚠️ PHẦN 3: CÁC GIẢI PHÁP KỸ THUẬT ĐẶC THÙ (CRITICAL WORKAROUNDS)
> **LƯU Ý CỰC KỲ QUAN TRỌNG DÀNH CHO AI AGENT TIẾP QUẢN:**  
> Không được tùy tiện thay thế các cơ chế dưới đây bằng các thư viện Python thông thường vì sẽ làm gãy hệ thống ngay lập tức trên máy người dùng!

### 3.1. Mã hóa DRMONE của Tập đoàn CJ (`\x9b DRMONE`)
* **Hiện tượng:** Tất cả file Excel (`.xlsx`, `.xlsb`) xuất từ hệ thống nội bộ CJ Foods đều bị bảo vệ bởi mã hóa bản quyền DRMONE. Header file chứa các byte `\x9b DRMONE`.
* **Hậu quả nếu dùng pandas/openpyxl/pyxlsb:** Thư viện Python thuần túy sẽ báo lỗi `zipfile.BadZipFile: File is not a zip file`.
* **Giải pháp chuẩn:** Bắt buộc phải đọc thông qua **Microsoft Excel COM Automation** (`win32com.client.Dispatch('Excel.Application')` hoặc Kingsoft WPS `ET.Application`). Excel của máy đã đăng nhập tài khoản doanh nghiệp CJ sẽ tự động giải mã file này.

### 3.2. Windows Mark-of-the-Web & Protected View
* **Hiện tượng:** File người dùng tải về qua Zalo / Teams / Outlook bị Windows gắn luồng Alternate Data Stream `:Zone.Identifier`. Khi mở bằng COM, Excel rơi vào chế độ *Protected View* và báo lỗi `Open method of Workbooks class failed`.
* **Giải pháp:** 
  1. Trong script batch và Python, tự động gọi `Get-ChildItem -Recurse | Unblock-File` và `os.remove(p + ':Zone.Identifier')`.
  2. Bắt exception và kích hoạt `pv.Edit()` từ `excel.ProtectedViewWindows`.

### 3.3. Xử lý treo tiến trình COM (COM Zombie Processes)
* **Giải pháp:** Không khởi tạo/đóng Excel COM lặp đi lặp lại trong vòng lặp từng file. Thay vào đó, gom toàn bộ file cần đọc vào một instance Excel duy nhất (`excel_batch`), sau khi đọc xong toàn bộ mới gọi `excel_batch.Quit()`.

### 3.4. Cơ chế Smart Cache (Tăng tốc độ từ 45s xuống 1s)
* **Vấn đề:** Dữ liệu có nhiều tháng lịch sử (T7: 14.000 dòng, T8: 11.000 dòng, T9: 3.000 dòng...). Nếu mỗi lần cập nhật đều mở Excel COM đọc lại tất cả thì rất chậm và dễ xung đột.
* **Giải pháp:**
  * Mỗi file trong `raw_data/` được tạo mã hash: `md5(f'{file_name}_{size}_{mtime}')`.
  * Dữ liệu đã xử lý được lưu trong `.cache/raw_{md5}.pkl`.
  * Nếu file cũ không bị sửa đổi, hệ thống nạp tức thì từ file `.pkl` (0.01 giây/file).
  * Chỉ khi có file mới hoặc file bị sửa đổi nội dung, Excel COM mới được khởi động.

### 3.5. Xử lý Trình duyệt bị kẹt Cache cũ (Browser Cache-Busting)
* **Vấn đề:** Khi nhân viên cập nhật dữ liệu mới lên GitHub, người xem mở web vẫn thấy số liệu cũ do trình duyệt tự động lưu cache file `coaching_data.js`.
* **Giải pháp:** Trong `index.html`, sử dụng đoạn mã nạp động theo timestamp:
  ```html
  <script>
    document.write('<script src="coaching_data.js?v=' + new Date().getTime() + '"><\/script>');
  </script>
  ```
  Nhờ đó, mỗi lần F5 hoặc mở web, trình duyệt luôn bắt buộc phải tải bản dữ liệu mới nhất.

### 3.6. Đồng bộ GitHub không cần Git CLI (`push_to_github.py`)
* **Vấn đề:** Máy nhân viên văn phòng thường không cài đặt Git, không biết dùng dòng lệnh git bash.
* **Giải pháp:** Viết script Python giao tiếp trực tiếp với **GitHub REST API v3**.
  * Sử dụng Personal Access Token (PAT) lưu tại `github_token.txt`.
  * Tính mã Git Blob SHA (`blob {len}\0{data}`) của từng file local và so sánh với SHA trên GitHub.
  * Chỉ đẩy các file thực sự có thay đổi nội dung (giảm băng thông và tránh xung đột).
  * Nếu gặp lỗi `401 Unauthorized` (do Token hết hạn), script sẽ in hướng dẫn rõ ràng và cho phép người dùng dán token mới trực tiếp ngay trong cửa sổ terminal.

---

## 📂 PHẦN 4: DANH MỤC FILE & VAI TRÒ CHI TIẾT

| Tên File / Thư mục | Định dạng | Vai trò & Trách nhiệm |
| :--- | :--- | :--- |
| **`coaching_report.py`** | Python | **Trọng tâm ETL:** Đọc file thô từ `raw_data/`, áp dụng Smart Cache, chuẩn hóa dữ liệu, tính toán KPI theo GSBH/Vùng/Miền và xuất file `coaching_data.js` + `Bao_Cao_Coaching_GSBH.xlsx`. |
| **`push_to_github.py`** | Python | **Đồng bộ GitHub:** Giao tiếp GitHub API, kiểm tra SHA và upload các file cập nhật lên repository `cjfood/coaching-WW`. |
| **`index.html`** | HTML/JS | **Dashboard giao diện:** Hiển thị trực quan dữ liệu từ `coaching_data.js`, bộ lọc đa chiều (Tháng, Miền, Vùng, GSBH), biểu đồ Chart.js, bảng chi tiết. |
| **`coaching_data.js`** | JS/JSON | Dữ liệu đầu ra do `coaching_report.py` sinh ra, chứa toàn bộ thống kê và data thô cho web. |
| **`DS GSBH.xlsx`** | Excel | Danh sách Master 47 GSBH chuẩn của CJ Foods. |
| **`ds_gsbh_master.json`** | JSON | Bản backup của danh sách GSBH Master (để bypass DRM nếu cần). |
| **`raw_data/`** | Thư mục | Chứa các file báo cáo công việc thô xuất từ DMS/SFA (T7, T8, T9...). |
| **`.cache/`** | Thư mục | Chứa các file pickle `.pkl` lưu trữ tạm dữ liệu thô đã giải mã. |
| **`github_token.txt`** | Text | Chứa mã GitHub Personal Access Token (PAT) có quyền `repo`. |
| **`requirements.txt`** | Text | Danh sách thư viện Python (`pandas`, `openpyxl`, `pyxlsb`, `pywin32`, `xlsxwriter`). |
| **`Cap_Nhat_Bao_Cao.bat`**| Windows Batch| File 1-click cho nhân viên chạy cập nhật dữ liệu định kỳ. |
| **`Cai_Dat_Tren_May_Moi.bat`**| Windows Batch| File 1-click thiết lập môi trường ảo `.venv` và cài thư viện trên máy mới. |
| **`HUONG_DAN_BAN_GIAO_VAN_HANH.md`**| Markdown | Hướng dẫn 4 bước cực ngắn gọn cho nhân viên vận hành hàng ngày. |
| **`COACHING_GSBH_TRON_GOI.zip`**| Zip | Gói nén toàn bộ dự án để chuyển giao giữa các máy tính. |

---

## 🛠️ PHẦN 5: QUY TRÌNH THIẾT LẬP TRÊN MÁY MỚI (SETUP GUIDE)

Khi chuyển thư mục dự án sang một máy tính mới (hoặc sau khi cài lại Windows):

1. **Cài đặt Python:** Tải Python 3.10 - 3.14 từ [python.org](https://www.python.org/downloads/) (Nhớ tích chọn: **`Add Python to PATH`**).
2. **Cài đặt Microsoft Excel:** Đảm bảo máy tính đã cài Microsoft Office Excel và đã đăng nhập tài khoản công ty để đọc được file nội bộ.
3. **Chạy thiết lập môi trường:** Nhấp đúp chuột vào file **`Cai_Dat_Tren_May_Moi.bat`**. File này sẽ tự động:
   * Mở khóa file (Unblock-File).
   * Tạo môi trường ảo `.venv`.
   * Cài đặt toàn bộ thư viện trong `requirements.txt`.
4. **Vận hành bình thường:** Nhấp đúp chuột vào **`Cap_Nhat_Bao_Cao.bat`** để kiểm tra và cập nhật báo cáo.

---

## 🎯 PHẦN 6: HƯỚNG DẪN DÀNH CHO AI AGENT KHI TIẾP QUẢN & PHÁT TRIỂN TIẾP
Nếu bạn là một AI Agent mới tiếp nhận dự án này, hãy tuân thủ các nguyên tắc sau:
1. **Không sửa đổi cấu trúc đầu ra `coaching_data.js` một cách tùy tiện:** Biến `window.COACHING_DATA` có cấu trúc gồm `meta`, `summary_by_month`, `gsbh_summary`, `raw_coaching`. Nếu thêm trường mới, hãy đảm bảo tương thích ngược để `index.html` không bị lỗi hiển thị.
2. **Luôn giữ gìn cơ chế Smart Cache:** Bất kỳ thay đổi nào trong `coaching_report.py` liên quan đến việc đọc file thô đều phải đi qua logic kiểm tra hash `.cache/` để đảm bảo thời gian chạy dưới 5 giây.
3. **Kiểm tra Token GitHub nếu gặp lỗi 401:** Nếu lệnh push báo 401, hãy nhắc người dùng kiểm tra file `github_token.txt` hoặc tạo token mới tại `https://github.com/settings/tokens/new` với quyền `repo` và chọn `No expiration`.
4. **Tự động đóng gói lại sau khi sửa:** Khi có thay đổi trong mã nguồn, hãy cập nhật lại file nén `COACHING_GSBH_TRON_GOI.zip` để người dùng có thể gửi ngay cho nhân viên.
