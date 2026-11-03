## PHẦN 1. PHÂN TÍCH

### 1. Tiến trình DNS Lookup

DNS (Domain Name System) được ví như cuốn danh bạ Internet, có nhiệm vụ chuyển đổi tên miền thành địa chỉ IP để trình duyệt tìm được máy chủ.

-   Input: Tên miền `phong-dieu-phoi.grenlog.vn`.
-   Output: Địa chỉ IP của máy chủ tương ứng.

**Sơ đồ luồng dữ liệu DNS:**

1.  Nam nhập tên miền vào trình duyệt.
2.  Trình duyệt kiểm tra bộ nhớ đệm DNS (DNS Cache).
3.  Nếu chưa có kết quả, máy tính gửi yêu cầu đến DNS Resolver.
4.  DNS Resolver tìm kiếm thông tin qua Root DNS Server.
5.  Root DNS Server chỉ đến máy chủ DNS quản lý tên miền `.vn`.
6.  DNS tiếp tục tìm máy chủ DNS có thẩm quyền (Authoritative DNS).
7.  Máy chủ DNS kiểm tra tên miền và trả về địa chỉ IP nếu tên miền tồn tại.
8.  Trình duyệt sử dụng IP để kết nối với máy chủ GreenLogistics.

**Nguyên nhân lỗi trong bài:**

Nam nhập sai tên miền thành `phong-dieu-phoi.grenlog.vn` (thiếu chữ e trong GreenLogistics). DNS không tìm thấy bản ghi tương ứng nên trả về lỗi `DNS_PROBE_FINISHED_NXDOMAIN`.

Hậu quả: Trình duyệt không tìm được địa chỉ IP, không thể kết nối đến máy chủ.
### 2.Phân biệt lỗi DNS và lỗi 500
|Tiêu chí|Lỗi DNS | Lỗi 500|
|:----:|:----:|:----:|
|Bản chất| Không tìm thấy tên miền hoặc lỗi IP|Máy chủ gặp lỗi nội bộ|
|Phía xảy ra lỗi|Client/DNS|Sever|
|Có kết nối HTTP không|Chưa thiết lập được|Đã kết nối|
|Nguyên nhân|Sai tên miền, DNS không có bản ghi|Lỗi code, cơ sở dữ liệu hoặc cấu hình Server|
|Kết quả |Không tìm thấy máy chủ|máy chủ trả về lỗi 500|

**Kết luận:** Lỗi DNS của Nam không chứng minh máy chủ công ty bị sập nguồn. Nguyên nhân trực tiếp là Nam nhập sai tên miền khiến DNS không tìm thấy địa chỉ IP.

## PHẦN 2. TRIỂN KHAI

### Quy trình kiểm tra HTTP Status Code

Nam thực hiện các bước sau:

1.  Kiểm tra lại tên miền và sửa thành `phong-dieu-phoi.greenlog.vn`.
2.  Mở trình duyệt và nhấn F12 để mở Developer Tools.
3.  Chọn tab Network.
4.  Tải lại trang bằng phím F5.
5.  Quan sát cột Status để kiểm tra mã trạng thái HTTP.

**Ý nghĩa một số mã trạng thái:**

-   200 OK: Máy chủ xử lý yêu cầu thành công.
-   301/302: Trang web chuyển hướng sang địa chỉ khác.
-   400 Bad Request: Yêu cầu gửi lên không hợp lệ.
-   403 Forbidden: Máy chủ từ chối quyền truy cập.
-   404 Not Found: Không tìm thấy tài nguyên được yêu cầu.
-   500 Internal Server Error: Máy chủ gặp lỗi nội bộ.

**Kết luận:** Nếu trình duyệt nhận được mã HTTP Status Code thì yêu cầu đã đến được một máy chủ HTTP. Nếu vẫn báo lỗi DNS thì yêu cầu chưa đến được máy chủ web thông qua tên miền đó.

## PHẦN 3. KIỂM THỬ

Giả sử Nam sửa đúng tên miền nhưng vẫn nhận lỗi `404 Not Found`.
**Cách khắc phục:**

1.  Kiểm tra lại đường dẫn trang quản trị.
2.  Xác nhận trang có tồn tại trên máy chủ hay không.
3.  Liên hệ bộ phận kỹ thuật để kiểm tra cấu hình và tài nguyên.

**Kết luận:** Lỗi 404 là lỗi không tìm thấy tài nguyên trên Server, không phải lỗi DNS. Điều này chứng tỏ Nam đã vượt qua bước phân giải tên miền và có thể kết nối đến máy chủ HTTP.
