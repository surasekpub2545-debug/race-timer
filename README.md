# GPS Race Timer

แอปจับเวลารถแข่งด้วย GPS แบบ PWA — ใช้บนมือถือได้เลย ติดตั้งบนหน้าจอโฮมและใช้งานออฟไลน์ได้

## ฟีเจอร์

- แสดงความเร็วเรียลไทม์จาก GPS (km/h) + Peak Speed
- โหมด **Arm** → รถขยับเมื่อไหร่ จับเวลาทันที (≥ 1 km/h)
- ใส่ระยะทางเป้าหมายเองได้ (เมตร) — เลือก preset 50/100/201/402/1000/1609 ก็ได้
- เวลาจะ **ตัดอัตโนมัติ** เมื่อรถวิ่งครบระยะ (แบบ interpolate ระหว่าง GPS frame เพื่อความแม่นยำ)
- แสดง GPS accuracy ตลอดเวลา (เขียว/เหลือง/แดง)
- บันทึกประวัติทุก run ลงเครื่อง (localStorage) + ไฮไลต์ Best Time ของแต่ละระยะ
- กันจอดับขณะใช้งาน (Wake Lock API)
- ใช้งานออฟไลน์ได้หลังเปิดครั้งแรก

## วิธีใช้งาน

1. กดเลือกระยะทาง หรือพิมพ์เลขในช่อง (หน่วยเมตร)
2. กดปุ่ม **ARM** — สถานะจะเปลี่ยนเป็น "ARMED — รอรถขยับ"
3. ออกรถได้เลย — แอปจะเริ่มจับเวลาทันทีที่ตรวจพบความเร็ว ≥ 1 km/h
4. พอวิ่งครบระยะที่ตั้งไว้ เวลาจะหยุดอัตโนมัติและบันทึกประวัติ
5. กด **RESET** เพื่อเริ่มใหม่

## ติดตั้งบนมือถือ (ต้องใช้ HTTPS)

GPS API ต้องการ HTTPS — ใช้ `file://` หรือ HTTP จะไม่ทำงานบนมือถือ มีหลายวิธีให้เลือก:

### วิธีที่ 1: GitHub Pages (ฟรี ง่ายสุด แนะนำ)

1. สร้าง repo ใหม่บน GitHub (เช่น `race-timer`)
2. อัปโหลดไฟล์ทั้งหมดในโฟลเดอร์นี้ (`index.html`, `manifest.json`, `sw.js`, `icon.svg`)
3. ไปที่ **Settings → Pages** → เลือก branch `main` / folder `/ (root)` → Save
4. รอ ~1 นาที จะได้ URL เช่น `https://username.github.io/race-timer/`
5. เปิด URL นั้นบนมือถือ → กด "Add to Home Screen" / "เพิ่มไปยังหน้าจอหลัก"

### วิธีที่ 2: Netlify Drop (เร็วสุด ไม่ต้องสมัคร)

1. ไปที่ <https://app.netlify.com/drop>
2. ลากโฟลเดอร์ `gps-race-timer` ทั้งโฟลเดอร์ลงไป
3. รับ URL แบบ HTTPS ทันที — เปิดบนมือถือ

### วิธีที่ 3: รัน Local Server (ทดสอบในเครือข่ายเดียวกัน)

ถ้า PC กับมือถืออยู่ใน Wi-Fi เดียวกัน:

```bash
# Python
cd D:\Surasek\GPS\gps-race-timer
python -m http.server 8080
```

แล้วเปิดบนมือถือ: `http://<IP-ของ-PC>:8080`

**แต่!** GPS บนมือถือต้องการ HTTPS เท่านั้น — ยกเว้น `localhost`
วิธีแก้: ใช้ `ngrok` หรือ `cloudflared` เพื่อทำ HTTPS tunnel

```bash
# ngrok (สมัครฟรี https://ngrok.com)
ngrok http 8080
# จะได้ URL HTTPS เช่น https://abc123.ngrok-free.app
```

### วิธีติดตั้งบนหน้าจอโฮม (Add to Home Screen)

**Android (Chrome):**
- เปิด URL → กดเมนูสามจุดมุมขวาบน → "Install app" หรือ "Add to Home screen"

**iPhone (Safari):**
- เปิด URL ใน Safari (ต้อง Safari เท่านั้น) → กดปุ่ม Share → "Add to Home Screen"
- ⚠️ iOS จะถามสิทธิ์ GPS ครั้งแรกที่กด ARM

## ข้อแนะนำเพื่อความแม่นยำ

- ใช้บริเวณโล่ง ไม่มีตึก/ต้นไม้บัง (GPS ทำงานได้แม่นยำกว่า)
- รอ accuracy ต่ำกว่า 8 เมตร (สถานะเขียว) ก่อนจะ ARM
- ตั้งโทรศัพท์ในที่ที่เห็นท้องฟ้า (เช่น cradle บนแดชบอร์ด)
- GPS มือถือทั่วไปอัปเดต ~1 Hz (ทุก 1 วิ) — แอปใช้การ **interpolate** ช่วยเพิ่มความแม่นยำของเวลาที่ตัด แต่ยังไม่ละเอียดเท่ากล่อง drag race เฉพาะทาง (10 Hz+) อยู่ดี
- iPhone และ Android รุ่นใหม่บางรุ่นรองรับ GPS dual-band ที่แม่นกว่ามาก

## โครงสร้างไฟล์

```
gps-race-timer/
├── index.html        — UI + Logic ทั้งหมด
├── manifest.json     — PWA manifest
├── sw.js             — Service Worker (offline cache)
├── icon.svg          — ไอคอนแอป
└── README.md         — ไฟล์นี้
```

ทุกอย่างอยู่ในไฟล์ HTML ไฟล์เดียว แก้ไขง่าย ไม่มี dependency ภายนอก

## ปรับแต่ง

ใน `index.html` ส่วนบนสุดของ `<script>`:

```js
const MOVE_THRESHOLD_KMH = 1.0;   // เปลี่ยนความเร็วขั้นต่ำที่ trigger จับเวลา
const ACC_GOOD = 8;               // accuracy ที่ถือว่า "ดี" (เมตร)
const ACC_OK   = 20;              // accuracy ที่ถือว่า "พอใช้"
```
