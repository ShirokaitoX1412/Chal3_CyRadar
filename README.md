# Challenge 03 - Internal Network Attack Simulation

## XÂY DỰNG MÔ PHỎNG MỘT CUỘC TẤN CÔNG THỰC TẾ

Thực hiện các bước như một cuộc tấn công thật:

* RECON & ENUMERATION
* XÁC ĐỊNH ĐIỂM TẤN CÔNG
* EXPLOITATION (Reverse Shell)
* PRIVILEGE ESCALATION (nếu có)
* LẤY FLAG

---

## 1. RECON & ENUMERATION

### Mục tiêu:

Tìm mọi thông tin có thể về hệ thống mục tiêu.

### Lệnh Nmap thực hiện:

<img width="1118" height="557" alt="{18ABC7D3-9600-4A70-BE47-71CB66D07061}" src="https://github.com/user-attachments/assets/85befb0d-9491-4778-a776-17d8c760008f" />
<img width="667" height="533" alt="{9845BFBF-667E-4626-9B19-F541F79008C6}" src="https://github.com/user-attachments/assets/3df6c078-cf61-4a1e-b182-14f4c4e61ead" />
<img width="864" height="574" alt="{0AB41F54-E405-4BA2-8356-232BA68FC9C8}" src="https://github.com/user-attachments/assets/4e8d773e-b5f3-458a-8f7f-0f8046be7bb7" />
<img width="703" height="573" alt="{01BC9EE7-2FFB-4B84-9E45-E2644D6738B0}" src="https://github.com/user-attachments/assets/83b0f0e4-5983-4853-902a-13579a049507" />



### Phân tích từng dịch vụ:
 
#### Port 5000 - Werkzeug HTTP

* Framework Python/Flask.
* Truy cập qua trình duyệt: `http://10.3.145.25:5000`
* Hiển thị trang “Nyan Cat Upload” → có tính năng upload file → nghi vấn **Unrestricted File Upload**.
<img width="1070" height="630" alt="{FFC39B06-7BD7-4809-AEA9-BCCB77A7CC9D}" src="https://github.com/user-attachments/assets/445a39e4-d885-4064-8f58-9ef42da6a824" />

Sử dụng Burpsuite bắt request upload lấy path 
<img width="971" height="351" alt="{A9AECD88-6F39-4D11-A51E-E6B906DE1759}" src="https://github.com/user-attachments/assets/71ae6fb9-dbfb-42a6-9e8b-41c9f4b8fe89" />

khai thác path traversal trên endpoint /file?path=... bằng cách sử dụng ký tự ..\\..
<img width="844" height="115" alt="{0CB1E2E4-2624-4437-AFF5-9DA074B702DE}" src="https://github.com/user-attachments/assets/28398058-aea6-42f7-9432-85725aab1802" />

Có endpoint /console trên Flask/Werkzeug
<img width="900" height="457" alt="{27F4FAC1-2D9F-4EA5-8190-CF6ABD7F84BB}" src="https://github.com/user-attachments/assets/ac7897ce-df34-40fa-a696-1a823a25a95c" />

<img width="1086" height="129" alt="{23F798A9-2F58-4E92-B7D5-140E641EA704}" src="https://github.com/user-attachments/assets/b82078e4-ee7f-4149-8955-06b582c9f224" />

Lỗi khi thực thi lệnh python => console đã bị vô hiệu hóa
<img width="1132" height="340" alt="{8A7C8980-3D07-437C-8C10-D8E6B4BC5EAD}" src="https://github.com/user-attachments/assets/2e818572-7c05-49d6-9d45-bd158340584a" />


#### Port 9998 - Jetty

* Jetty là Java web server.
* Cho phép PUT file lên server.
* Kiểm tra bằng curl:

<img width="795" height="803" alt="{60551902-DD23-4228-A90F-893E8241D5BD}" src="https://github.com/user-attachments/assets/c1e520fb-6cea-41a6-9933-25bcc55bb5f9" />


<img width="529" height="27" alt="{350A89D1-420F-40FF-BDF7-BD693475678F}" src="https://github.com/user-attachments/assets/89ab8472-f0e0-4e37-9806-733c8675b28e" />

Ý tưởng 
Tạo một file .jar chứa mã Java độc hại.
Gửi .jar đó lên endpoint /meta.
Khi server xử lý file, nó thực thi mã bên trong → RCE.

Tạo 2 terminal 1 cái lắng nghe từ server 1 cái gửi payload lên apache tika
Bước 1 : Terminal 1 – Khởi động Interactsh client
Khởi động Interactsh để nhận kết nối từ server nếu payload thành công:
<img width="802" height="268" alt="{CDA2458D-D41D-48A5-AA5C-510248ED751C}" src="https://github.com/user-attachments/assets/2f74e6e3-be1f-40a1-8b06-cbedd3085a49" />
Sau khi chạy nhận được subdomain d1r5iihqrgmn6dc0mtmggyixg9hd1miu8.oast.site

Bước 2 Terminal 2  Viết mã Java độc hại
<img width="933" height="197" alt="{D63AC816-5E0F-49B2-9F94-0127F32181D5}" src="https://github.com/user-attachments/assets/b2898ed9-9fc8-4c5e-8844-b2d22c76c4e1" />
Khi .jar được load, nó gửi request về subdomain

Bước 3 – Gửi file .jar đến Tika
<img width="898" height="138" alt="{5449ACF4-E0DB-4B83-97E5-8D2AD53E742B}" src="https://github.com/user-attachments/assets/5d448061-2ee3-48e9-a920-cc93d1e7d6da" />
Sau đó kiểm tra terminal 1 sẽ thấy log 
<img width="960" height="51" alt="{0F02E2A0-0890-4308-8C18-1BD341437564}" src="https://github.com/user-attachments/assets/3d552d40-1ee2-4e3a-aa50-860ab01db4c0" />




#### Port 11025 - SharePoint
* Truy cập: `http://10.3.145.25:11025/_layouts/15/start.aspx#/default.aspx`
* Là một site SharePoint nội bộ.

<img width="913" height="808" alt="{DD46C3AB-A08E-488E-8C5B-448F89AAC4AE}" src="https://github.com/user-attachments/assets/59c6d449-d1e0-4d6d-b7f0-9870713a92c9" />


<img width="1082" height="325" alt="{66627054-9D47-40B8-81E7-56E80BABBD08}" src="https://github.com/user-attachments/assets/2171ca9b-6100-4ea4-be4a-67857f54e11e" />


<img width="1093" height="463" alt="{F1055EB4-BE26-41F8-A4D8-F86774D17C01}" src="https://github.com/user-attachments/assets/279e72cd-de0a-40e0-b922-e43dbb09adb0" />
_vti_inf.html → tiềm năng liên quan đến CV
CVE-2019-0604 – RCE qua ViewState

