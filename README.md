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
Sử dụng nuclei kiểm thử từng port 
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

Lấy cookie phòng trường hợp mất PIN 
<img width="446" height="68" alt="{AEF7E8E7-6FAB-4620-ADAC-9F0CF97C638D}" src="https://github.com/user-attachments/assets/52965e26-bb45-4245-885c-b2d48f07fdef" />
Lỗi khi thực thi lệnh python => console đã bị vô hiệu hóa
<img width="1132" height="340" alt="{8A7C8980-3D07-437C-8C10-D8E6B4BC5EAD}" src="https://github.com/user-attachments/assets/2e818572-7c05-49d6-9d45-bd158340584a" />



#### Port 9998 - Jetty

* Jetty là Java web server.
* Cho phép PUT file lên server.
* Kiểm tra bằng curl:

<img width="795" height="803" alt="{60551902-DD23-4228-A90F-893E8241D5BD}" src="https://github.com/user-attachments/assets/c1e520fb-6cea-41a6-9933-25bcc55bb5f9" />

<img width="870" height="425" alt="{72287263-1A27-487A-9731-F7EF6B2B96A9}" src="https://github.com/user-attachments/assets/c1fce0f2-e584-49f5-8d53-493c7e6bc630" />



#### Port 11025 - SharePoint

* Header:

```
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
```

* Truy cập: `http://10.3.145.25:11025/_layouts/15/start.aspx#/default.aspx`
* Là một site SharePoint nội bộ.

<img width="913" height="808" alt="{DD46C3AB-A08E-488E-8C5B-448F89AAC4AE}" src="https://github.com/user-attachments/assets/59c6d449-d1e0-4d6d-b7f0-9870713a92c9" />

<img width="829" height="315" alt="{00334761-7EC4-42A3-8269-BA143461DEBB}" src="https://github.com/user-attachments/assets/65ef19fc-a1c7-4b4c-865b-096fc54d4ea3" />
<img width="1082" height="325" alt="{66627054-9D47-40B8-81E7-56E80BABBD08}" src="https://github.com/user-attachments/assets/2171ca9b-6100-4ea4-be4a-67857f54e11e" />


<img width="1538" height="734" alt="{FF001C93-F300-4D8B-9D6D-306797A4A8B7}" src="https://github.com/user-attachments/assets/fce8da7e-49fe-470d-ae8a-25a43d4c97bf" />

<img width="1295" height="811" alt="{56FA720A-03B2-4FFF-A320-433DAA17F385}" src="https://github.com/user-attachments/assets/a75fb54e-51c9-4b74-ac6c-e684b2694bd9" />

<img width="866" height="301" alt="{9208D7AB-F0AA-4730-B0D6-B2844BBFEA82}" src="https://github.com/user-attachments/assets/fcc5778c-b41c-4fb1-97ef-f3f5a751d0ca" />

