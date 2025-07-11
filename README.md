# Challenge 03 - Writeup (Penetration Test Simulation)

## RECON & ENUMERATION

### Mục tiêu

Tìm mọi thông tin có thể từ hệ thống mục tiêu.

### Công cụ chính

* `nmap`

### Thực hiện:

Chạy lệnh sau để scan toàn bộ cổng và nhận diện dịch vụ đang chạy:

```bash
nmap -sC -sV -Pn -T4 -p- 10.3.145.25
```

### Kết quả:

Mục tiêu bật nhiều cổng TCP, trong đó có các dịch vụ đáng chú ý:

* **Kerberos** (port 88)
* **LDAP** (port 389, 3268)
* **SMB** (port 445, 139)
* **MS SQL Server** (port 54681)
* **RDP** (port 3389)
* **Web server** trên các port: 5000 (Werkzeug), 5202, 6009, 11025 (SharePoint), 9998 (Jetty), v.v.

Ví dụ đầu ra:

```
5000/tcp  open  http          Werkzeug httpd 2.0.0 (Python 3.10.10)
9998/tcp  open  http          Jetty 8.y.z-SNAPSHOT
11025/tcp open  http          Microsoft IIS httpd 10.0 (SharePoint)
54681/tcp open  ms-sql-s      Microsoft SQL Server 2014
```

### Nhận xét ban đầu

* Có sự xuất hiện của môi trường Active Directory (Kerberos + LDAP + SMB + domain: xteam.local).
* Có thể khai thác thông qua web services (Werkzeug upload, Jetty PUT, SharePoint).
* Có thể thu thập thông tin người dùng domain bằng Kerberos hoặc LDAP (nếu không cần xác thực).
* MS SQL có thể là vector cho RCE.

→ Tiếp theo sẽ phân tích chi tiết từng vector để xác định điểm tấn công khả thi.
