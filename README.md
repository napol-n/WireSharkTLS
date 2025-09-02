# Wireshark TLS Decryption 

คู่มือนี้อธิบายวิธีการถอดรหัส (decrypt) การรับส่งข้อมูล TLS (HTTPS) ใน Wireshark ด้วยการใช้ session keys จากเบราว์เซอร์

---

## Workflow Diagram

![TLS Decryption Workflow](https://user-images.githubusercontent.com/yourusername/placeholder-tls-diagram.png)

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
