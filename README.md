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
* `whatweb`, `curl`
* `netcat`, `telnet`
* `smbclient`, `ldapsearch`
* `searchsploit`

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
<img width="1022" height="804" alt="{7EDE327D-5B45-4C47-AECC-5850A4B127B2}" src="https://github.com/user-attachments/assets/e8cc7579-f7d0-46e7-a652-d599dc23ee78" />


#### Port 9998 - Jetty

* Jetty là Java web server.
* Cho phép PUT file lên server.
* Kiểm tra bằng curl:

<img width="795" height="803" alt="{60551902-DD23-4228-A90F-893E8241D5BD}" src="https://github.com/user-attachments/assets/c1e520fb-6cea-41a6-9933-25bcc55bb5f9" />


* Phản hồi có phương thức `PUT`, nghĩa là có thể upload file độc hại:

```http
Allow: GET,HEAD,POST,PUT,DELETE,OPTIONS
```

#### Port 11025 - SharePoint

* Header:

```
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
```

* Truy cập: `http://10.3.145.25:11025/_layouts/15/start.aspx#/default.aspx`
* Là một site SharePoint nội bộ.

<img width="913" height="808" alt="{DD46C3AB-A08E-488E-8C5B-448F89AAC4AE}" src="https://github.com/user-attachments/assets/59c6d449-d1e0-4d6d-b7f0-9870713a92c9" />


#### LDAP/AD/Kerberos (Port 389, 3268, 88, 464)

* Xác định domain: `xteam.local`
* LDAP từ chối anonymous bind:

```bash
ldapsearch -x -h 10.3.145.25 -s base
# => result: Insufficient access
```

* Tuy nhiên có thể khai thác khi có thông tin user sau này.

#### MSSQL (port 54681)

* Banner cho thấy chạy MSSQL Server 2014 RTM
* Có thể dùng `sqsh`, `impacket-mssqlclient`, hoặc PowerShell để truy cập nếu có credential

#### SMB (Port 139/445)

* Thử liệt kê share:

```bash
smbclient -L \\10.3.145.25 -N
```

* Kết quả: bị từ chối anonymous

```bash
Anonymous login successful
NT_STATUS_ACCESS_DENIED listing \\*
```

### Nhận định:

* Các cổng đáng chú ý cho exploit: 5000 (Werkzeug upload), 9998 (Jetty PUT), 11025 (SharePoint).
* SMB và LDAP cần credential → có thể thu thập sau.
* Nhiều dịch vụ nội bộ phản ánh đây là một máy trong domain (xteam.local), có thể dẫn đến leo thang đặc quyền sau khi có shell.

→ Tiếp tục xác định điểm tấn công từ các web service có khả năng dễ khai thác nhất.
