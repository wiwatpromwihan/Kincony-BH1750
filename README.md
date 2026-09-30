# Kincony BH1750 · Seedling Monitor Dashboard

หน้าเว็บ Dashboard สำหรับติดตามค่าจาก ESP32/Kincony ผ่าน Firebase Realtime Database และ GitHub Pages หน้าเว็บนี้อ่านข้อมูลอย่างเดียว การควบคุม Relay ทำผ่าน ESPHome Web Server ที่บอร์ดตามที่กำหนดไว้

## ไฟล์

- `index.html` — responsive dashboard และกราฟ Chart.js
- `firebase-config.js` — Firebase Web App config placeholders และ path ของข้อมูล

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
- Dashboard ไม่มีปุ่มควบคุม Relay

หน้าเว็บโหลด Firebase JS SDK และ Chart.js จาก CDN จึงต้องเชื่อมอินเทอร์เน็ตใน browser เพื่อเปิดหน้า Dashboard
