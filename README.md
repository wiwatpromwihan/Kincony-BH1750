# Kincony BH1750 · Seedling Monitor Dashboard

หน้าเว็บ Dashboard สำหรับติดตามค่าจาก ESP32/Kincony ผ่าน Firebase Realtime Database และ GitHub Pages พร้อมปุ่มควบคุม Relay ผ่าน MQTT over WebSocket

## ไฟล์

- `index.html` — responsive dashboard และกราฟ Chart.js
- `firebase-config.js` — Firebase Web App config และ path ของข้อมูล

## ตั้งค่า Firebase

1. ใน Firebase Console เปิดโปรเจกต์ที่เก็บ RTDB และเพิ่ม Web App หากยังไม่มี
2. คัดลอก config ของ Web App แล้วแทนค่า `YOUR_FIREBASE_WEB_API_KEY`, `YOUR_PROJECT_ID`, `YOUR_MESSAGING_SENDER_ID` และ `YOUR_FIREBASE_WEB_APP_ID` ใน `firebase-config.js` (รวม `authDomain`/`storageBucket` หากโปรเจกต์ใช้ค่าต่างจากตัวอย่าง)
3. ตรวจว่า RTDB URL และ path ตรงกับปลายทางของอุปกรณ์: `https://iot-209-224-default-rtdb.asia-southeast1.firebasedatabase.app/lab/esp32-209-224/latest.json` และ `/history.json`
4. RTDB Rules ต้องอนุญาตให้หน้าเว็บอ่าน path `/lab/esp32-209-224/latest` และ `/history` ได้ หากเปิด Authentication หรือ Rules ปฏิเสธการอ่าน listener จะแสดงข้อผิดพลาดในแถบด้านบน

ค่า Firebase Web config ถูกออกแบบให้ใช้งานใน client ได้ แต่สิทธิ์การอ่าน/เขียนต้องกำหนดด้วย RTDB Rules ห้ามนำ service account key หรือ credential ฝั่ง admin มาใส่ในเว็บ

## เผยแพร่บน GitHub Pages

จาก root ของ repository:

```bash
git add index.html firebase-config.js README.md
git commit -m "Add realtime seedling monitoring dashboard"
git push origin main
```

จากนั้นเปิด **Settings → Pages** ใน GitHub repository เลือก **Deploy from a branch**, branch `main`, folder `/ (root)` แล้วกด Save รอ build เสร็จ หน้าเว็บจะอยู่ที่ `https://wiwatpromwihan.github.io/Kincony-BH1750/` หากเปิด Pages ไว้แล้ว การ push ไป `main` จะ deploy อัตโนมัติ

## ข้อมูลและพฤติกรรมหน้าเว็บ

- `/lab/esp32-209-224/latest`: Firebase JS SDK modular `onValue` listener
- `/lab/esp32-209-224/history`: `orderByChild('timestamp')` และ `limitToLast(60)`; ไม่ดึง history ทั้งหมด
- สถานะบอร์ดคำนวณจาก `latest.timestamp`: online เมื่อข้อมูลมีอายุไม่เกิน 90 วินาที ไม่อาศัย `board.online`
- กราฟแสงแสดง `light.lux`; กราฟที่เลือก Mijia แสดง `temp` และ `humi` จากแถว history โดยใช้ timestamp หน่วยวินาที
- แสดง Mijia 01–10 รวมถึงสถานะยังไม่พบสัญญาณ, ค่าอุณหภูมิ/ความชื้น/แบตเตอรี่ และสถานะ Relay 1–6
- ปุ่มเปิด/ปิด Relay ส่ง `TOGGLE` ไปยัง `pkru/kc868_a6_01/209-219/relay/{1..6}/toggle` (ตรงกับ firmware ที่ใช้อยู่); ปุ่มปิดทั้งหมดส่ง `OFF` ไป `.../all/set`
- ปุ่มควบคุมจะเปิดใช้เมื่อ browser เห็น retained status `online` จากบอร์ดผ่าน MQTT และข้อมูล Firebase สดไม่เกิน 90 วินาที ปุ่มยังใช้ได้ระหว่างรอสถานะกลับจาก Firebase; หากไม่มีการยืนยันใน 45 วินาทีจะแสดงให้ลองใหม่

หน้าเว็บโหลด Firebase JS SDK และ Chart.js จาก CDN จึงต้องเชื่อมอินเทอร์เน็ตใน browser เพื่อเปิดหน้า Dashboard

## คำสั่ง Relay และข้อจำกัดความปลอดภัย

การควบคุมหน้าเว็บอาศัย MQTT config ใน firmware: `test.mosquitto.org:1883` และ topic ที่กำหนดใน `on_message` ของอุปกรณ์ ส่วน browser เชื่อม `wss://test.mosquitto.org:8081/mqtt` ตัว broker นี้เป็น broker สาธารณะ ไม่ต้องใช้บัญชี และทุกคนที่รู้ topic สามารถ publish ได้ จึงใช้กับ Relay จำลองเพื่อสาธิตเท่านั้น ห้ามต่อโหลดจริงหรืออุปกรณ์ที่ทำให้เกิดอันตราย ควรย้ายไป broker ส่วนตัวที่ใช้ TLS, username/password และ ACL จำกัดสิทธิ์ topic ก่อนใช้งานจริง

Firmware ตัวอย่างยังสร้างเสียงแจ้งเตือนด้วยการกระพริบ Relay 1–6 ตาม event บางอย่าง เช่นแสงและแบตเตอรี่ ดังนั้น event เหล่านี้อาจเปลี่ยนสถานะ Relay หลังสั่งจาก Dashboard ได้ หากต้องการให้ Relay เป็นเอาต์พุตควบคุมคงที่ ให้ปิด/ย้าย animation เหล่านั้นไปยังเอาต์พุตเฉพาะก่อน

## แก้สถานะ BH1750 ที่รายงานปกติทั้งที่ไม่มีค่า

Firmware เดิมตั้ง `light.healthy` จากตัวตรวจค่ากระโดดเท่านั้น จึงอาจเป็น `true` ตั้งแต่ยังไม่มีค่า `lux` (ค่าเซนเซอร์เป็น NaN) หน้าเว็บใหม่นี้ถือว่าแสงปกติเฉพาะเมื่อมีค่า lux เป็นตัวเลขและ `healthy` เป็น true; เมื่อไม่มี lux จะแสดง “ไม่มีข้อมูล lux · ตรวจสอบ BH1750/I²C”. ได้แก้ตัวสร้าง snapshot ในไฟล์ทำงาน `209_224.txt` ให้ `healthy=false` และส่งสถานะตรวจสอบ I²C เมื่อ BH1750 ยังไม่มีค่าด้วย ต้องคอมไพล์และแฟลช firmware ฉบับแก้ไขลงบอร์ดเพื่อให้ Firebase แสดงสถานะถูกต้องด้วย
