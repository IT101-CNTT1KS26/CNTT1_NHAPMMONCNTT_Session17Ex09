# GIẢI PHÁP KIẾN TRÚC ĐA BROWSER PROFILE VÀ QUẢN LÝ DƯỢC THƯ – TRẠM Y TẾ PHƯỜNG CAREPULSE

## PHẦN 1: BÁO CÁO PHÂN TÍCH KIẾN TRÚC ĐA PROFILE

### 1. Phân tích nguyên nhân sâu xa dẫn đến hiện tượng "văng phiên đăng nhập" (Session Conflict)

* **Cơ chế lưu trữ của trình duyệt mặc định:** Tất cả các tab, cửa sổ và phiên làm việc của Google Chrome khi chạy trên cùng một Profile mặc định (`Default Profile`) đều dùng chung duy nhất một **Thư mục dữ liệu người dùng (User Data Directory)**.
* **Xung đột Cookie & Session Token:** Khi Bác sĩ trực đang đăng nhập hệ thống CarePulse, tệp `Cookies` và bộ nhớ `Local Storage` lưu trữ mã định danh phiên làm việc (Session Token/JWT) của bác sĩ trên miền hệ thống. Khi sinh viên thực tập mở tab mới đăng nhập vào GitHub hoặc Cổng học tập PTIT, hoặc khi hệ thống ghi đè các định danh phiên làm việc đa tài khoản trên cùng một không gian lưu trữ chung, trình duyệt hoặc máy chủ sẽ hiểu là có sự thay đổi trạng thái xác thực hoặc bị tranh chấp tài nguyên phiên. Hậu quả là token của bác sĩ bị vô hiệu hóa hoặc bị ghi đè, dẫn đến hiện tượng **văng phiên đăng nhập (Session Logout)** đột ngột giữa ca khám bệnh.
* **Lộ lọt dữ liệu & Quyền riêng tư:** Do dùng chung thư mục Profile, lịch sử duyệt web (`History`), bộ nhớ đệm (`Cache`), từ khóa tìm kiếm và mật khẩu tự động điền (`Auto-fill Passwords`) bị trộn lẫn hoàn toàn, gây ra tình trạng mất an toàn thông tin cá nhân và y tế.

### 2. Thiết kế Mô hình 3 Profile chuyên dụng cho máy tính trạm khám CarePulse

Để triệt tiêu hoàn toàn xung đột phiên và bảo mật thông tin, máy tính dùng chung tại trạm y tế được phân tách thành 3 Profile độc lập:

| Tên Profile | Mục đích sử dụng | Màu Theme nhận diện thị giác | Dữ liệu được lưu trữ độc lập |
| --- | --- | --- | --- |
| **1. CarePulse Work** | Dành riêng cho Bác sĩ trực và Điều dưỡng viên để xử lý nghiệp vụ khám chữa bệnh, kê đơn và tra cứu hệ thống y tế. | **Xanh dương đậm (Professional Blue)** | Độc lập hoàn toàn về Cookies hệ thống y tế CarePulse, Lịch sử chuyên môn, Bookmarks y khoa, Extension hỗ trợ y tế. |
| **2. Personal & Study** | Dành cho nhân sự tra cứu tài liệu cá nhân, học tập trực tuyến (Cổng PTIT, GitHub cá nhân) trong giờ nghỉ hoặc giao ca. | **Xanh lá / Vàng (Friendly Green/Amber)** | Độc lập về tài khoản Google cá nhân, Lịch sử duyệt web cá nhân, Dữ liệu học tập. |
| **3. Sandbox / Test** | Dành cho sinh viên thực tập hoặc kiểm tra nhanh các trang web lạ, thử nghiệm tính năng mới mà không ảnh hưởng hệ thống chính. | **Đỏ / Xám (Caution Red/Gray)** | Dữ liệu tạm thời, tự động xóa sạch cache sau khi đóng cửa sổ để ngăn chặn rác hệ thống. |

---

## PHẦN 2: THIẾT KẾ HỆ THỐNG BOOKMARK & QUY TRÌNH AN TOÀN

### 3. Cây thư mục Bookmark chuẩn (Ctrl + Shift + O)

Tại Profile **CarePulse Work**, thanh Bookmark được cấu trúc khoa học thành 3 thư mục chính:

* ** `01_CarePulse_Clinic**` *(Thư mục nghiệp vụ cốt lõi)*
1. Trang chủ Quản lý Khám bệnh CarePulse (`[https://carepulse.local/dashboard](https://carepulse.local/dashboard)`)
2. Tra cứu Hồ sơ Bệnh án Điện tử EMR (`[https://carepulse.local/emr](https://carepulse.local/emr)`)
3. Hệ thống Kê đơn Thuốc Quốc gia (`[https://hosokethuoc.gov.vn](https://hosokethuoc.gov.vn)`)


* ** `02_Pharma_Portal**` *(Thư mục tra cứu dược phẩm)*
1. Tra cứu Dược thư Quốc gia Việt Nam (`[https://duothu.org.vn](https://duothu.org.vn)`)
2. Danh mục Thuốc Bảo hiểm Y tế (`[https://bhyt.gov.vn/dan-muc-thuoc](https://bhyt.gov.vn/dan-muc-thuoc)`)
3. Cổng thông tin Quản lý Dược Quốc gia (`[https://drugdept.dav.gov.vn](https://drugdept.dav.gov.vn)`)


* ** `03_Medical_Tools**` *(Thư mục công cụ hỗ trợ lâm sàng)*
1. Tra cứu Tương tác Thuốc (Drugs.com / Micromedex)
2. Công cụ tính chỉ số BMI / Liều lượng theo diện tích cơ thể (BSA)
3. Google Dịch Y khoa (`[https://translate.google.com](https://translate.google.com)`)



### 4. Mẹo tối ưu không gian "Favicon-Only" cho 4 trang web dùng hàng ngày

Để thanh công cụ gọn gàng, chứa được hơn 30 liên kết mà không bị tràn màn hình:

* **Cách thực hiện:** Nhấp chuột phải vào từng liên kết trên thanh Bookmark của 4 trang quan trọng (CarePulse, Cổng BHYT, Dược thư Quốc gia, Google Dịch), chọn **Chỉnh sửa (Edit)**, sau đó **xóa sạch phần Tên (Name)** và chỉ để lại đường dẫn URL trống tên.
* **Kết quả:** Thanh công cụ chỉ hiển thị các biểu tượng biểu trưng nhỏ gọn (**Favicon**). Tiết kiệm diện tích hiển thị gấp 3 lần, giúp bác sĩ nhận diện trực quan bằng mắt và click chọn ngay lập tức trong vòng chưa tới 1 giây mà không phải cuộn chuột tìm kiếm.

---

## PHẦN 3: XỬ LÝ BẪY AN NINH VÀ DUYỆT WEB TẠM THỜI

### 5. Xử lý Bẫy an ninh: Khi đoàn thanh tra hoặc khách mượn máy tính kiểm tra trong 30 phút

* **Chế độ cần kích hoạt:** Sử dụng **Guest Profile (Khách)** hoặc **Incognito (Ẩn danh)**. Tuy nhiên, ưu tiên tuyệt đối sử dụng **Guest Profile** của trình duyệt Chrome cho khách/đoàn thanh tra mượn máy.
* **Giải thích lý do:**
* Khi mở phiên **Guest Profile**, trình duyệt tạo ra một không gian cách ly hoàn toàn, không cho phép khách truy cập nhìn thấy lịch sử, mật khẩu hay tài khoản đăng nhập của các Profile nhân viên y tế khác (`Work` hay `Personal`).
* Khi khách đóng cửa sổ Guest, toàn bộ lịch sử duyệt web, cookie và dữ liệu phiên làm việc của họ sẽ **tự động bị xóa sạch**, không để lại bất kỳ dấu vết rác hay rò rỉ thông tin nào trên máy tính phòng khám.


* **Cảnh báo nguy cơ nghiêm trọng nếu bật Cloud Sync tài khoản cá nhân:**
* *Nguy hiểm:* Nếu nhân viên vô tình đăng nhập và bật tính năng **Đồng bộ hóa đám mây (Cloud Sync)** của tài khoản Google cá nhân trên máy tính phòng khám dùng chung, toàn bộ mật khẩu, lịch sử tìm kiếm cá nhân, thẻ tín dụng lưu trữ trên đám mây sẽ bị kéo về máy trạm này.
* Ngược lại, lịch sử duyệt web khám chữa bệnh nhạy cảm của bệnh nhân trên trạm y tế sẽ bị đồng bộ ngược lên tài khoản cá nhân của nhân sự, dẫn đến vi phạm nghiêm trọng luật bảo mật thông tin y tế (HIPAA / Quy định bảo mật dữ liệu cá nhân).
