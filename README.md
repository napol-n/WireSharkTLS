# Wireshark TLS Decryption 

คู่มือนี้อธิบายวิธีการถอดรหัส (decrypt) การรับส่งข้อมูล TLS (HTTPS) ใน Wireshark ด้วยการใช้ session keys จากเบราว์เซอร์

---

## Workflow Diagram

# Wireshark TLS Decryption Flowchart

```mermaid
flowchart TD
    A[Browser เปิดเว็บไซต์ HTTPS] --> B[สร้าง Session Keys สำหรับ TLS]
    B --> C[บันทึก Keys ลงไฟล์ SSLKEYLOGFILE]
    A --> D[ส่งข้อมูล HTTPS เข้ารหัสไปยัง Network]
    D --> E[Wireshark จับแพ็กเก็ตจาก Interface]
    C --> E
    E --> F[Wireshark โหลด SSLKEYLOGFILE และถอดรหัส TLS]
    F --> G[Plaintext Application Data]
    G --> H[Follow TLS Stream ตรวจสอบ HTTP/JSON/HTML Content]

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#ff9,stroke:#333,stroke-width:2px
    style C fill:#9f9,stroke:#333,stroke-width:2px
    style D fill:#9ff,stroke:#333,stroke-width:2px
    style E fill:#fc9,stroke:#333,stroke-width:2px
    style F fill:#cff,stroke:#333,stroke-width:2px
    style G fill:#ffc,stroke:#333,stroke-width:2px
    style H fill:#9ff,stroke:#333,stroke-width:2px


**คำอธิบาย Diagram:**

1. **Browser**  
   - สร้าง session keys สำหรับการเชื่อมต่อ HTTPS  
   - บันทึก keys ลงไฟล์ `SSLKEYLOGFILE`

2. **Network Traffic**  
   - การรับส่งข้อมูล HTTPS ถูกเข้ารหัส  
   - Wireshark จับแพ็กเก็ตทั้งหมด

3. **Wireshark**  
   - โหลดไฟล์ `SSLKEYLOGFILE` เพื่อนำ keys มาถอดรหัสข้อมูล TLS  

4. **Decrypted Application Data**  
   - ข้อมูล HTTP/JSON/HTML สามารถอ่านเป็น plaintext  

5. **Follow TLS Stream**  
   - ใช้ Follow → TLS Stream ใน Wireshark เพื่อดูเนื้อหาแต่ละ connection

---

## ขั้นตอน TLS Decryption

### 1. ตั้งค่าเบราว์เซอร์ให้บันทึก TLS Keys

**Chromium/Chrome:**
```bash
export SSLKEYLOGFILE=$HOME/sslkeylog.log
chromium-browser
# หรือ google-chrome


Firefox:
export SSLKEYLOGFILE=$HOME/sslkeylog.log
firefox
ตรวจสอบไฟล์ log:
ls -l ~/sslkeylog.log
cat ~/sslkeylog.log
ตัวอย่างเนื้อหาไฟล์:
CLIENT_HANDSHAKE_TRAFFIC_SECRET ...
SERVER_HANDSHAKE_TRAFFIC_SECRET ...


2. จับ Traffic ด้วย Wireshark
เปิด Wireshark
เลือก interface เครือข่าย (เช่น eth0)
กด Start
เรียกดูเว็บไซต์ HTTPS


3. โหลด SSLKEYLOGFILE ใน Wireshark
ไปที่ Edit → Preferences → Protocols → TLS
ตั้งค่า (Pre)-Master-Secret log filename:
~/sslkeylog.log
คลิก Apply → OK


4. กรอง Traffic TLS
ใช้ filter:
tls
http
เลือกแพ็กเก็ต → คลิกขวา → Follow → TLS Stream
จะเห็น HTTP request และ response ที่ถอดรหัสแล้ว


5. ตัวอย่างผลลัพธ์
ก่อนถอดรหัส:
Application Data (Encrypted)
หลังถอดรหัส:
GET /time/1/current?cup2key=xxx HTTP/1.1
Host: clients2.google.com
User-Agent: Chrome/Firefox
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json
{
  "time": "2025-09-02T14:53:54Z"
}
หมายเหตุ
เบราว์เซอร์ต้องรองรับ SSLKEYLOGFILE
เริ่มเบราว์เซอร์ หลังจาก export SSLKEYLOGFILE
เว็บไซต์ที่ใช้ certificate pinning หรือ HSTS อาจไม่สามารถถอดรหัสได้
ใช้ filter ใน Wireshark เพื่อแยก traffic ที่ต้องการ
แหล่งอ้างอิง
Wireshark TLS Decryption
SSLKEYLOGFILE Environment Variable

---

 ****  
