# ตั้งค่าเชื่อม Outlook (Microsoft 365 บัญชีบริษัท)

ทำครั้งเดียว ประมาณ 15–30 นาที — ถ้าบัญชีของคุณไม่มีสิทธิ์สร้างแอป (ข้อ 1 กดไม่ได้ / ขึ้นว่าไม่มีสิทธิ์) ให้ส่งไฟล์นี้ให้ฝ่าย IT ทำให้

## สิ่งที่โปรแกรมทำ (สำหรับฝ่าย IT)
- เป็นเว็บแบบ static (Cloudflare Workers) ไม่มีเซิร์ฟเวอร์หลังบ้าน
- ล็อกอินด้วย Microsoft Entra ID แบบ **SPA + Authorization Code + PKCE** (ไลบรารี MSAL.js 4.30.0 ของ Microsoft)
- ขอสิทธิ์แบบ **Delegated** เท่านั้น: `User.Read`, `Mail.Read` — **อ่านอย่างเดียว** ไม่ส่ง ไม่ลบ ไม่แก้อีเมล
- อ่านได้เฉพาะกล่องจดหมายของคนที่ล็อกอินเอง เรียก Graph จากเบราว์เซอร์ของผู้ใช้โดยตรง
  (`GET /me/messages?$search="participants:<อีเมลลูกค้า>"`) ไม่มีข้อมูลผ่านเซิร์ฟเวอร์อื่น
- Token เก็บใน localStorage ของเบราว์เซอร์ผู้ใช้ (ใครใช้เครื่อง/เบราว์เซอร์เดียวกันได้ จะเข้าถึงได้ — ควรใช้กับเครื่องส่วนตัว และกด "ออกจากระบบ" เมื่อเลิกใช้)

## ขั้นตอน

### 1) สร้างแอป
1. เข้า https://entra.microsoft.com → **Applications** → **App registrations** → **New registration**
2. **Name:** `Lead Tracker`
3. **Supported account types:** `Accounts in this organizational directory only (Single tenant)`
4. **Redirect URI:** เลือกประเภท **Single-page application (SPA)** (ห้ามเลือก Web)
   แล้วใส่ URL จากช่อง **Redirect URI** ในโปรแกรม (แท็บ ⚙️ ตั้งค่า → กด "คัดลอก")
   หน้าตาจะประมาณ `https://leadprogram.<ชื่อบัญชี>.workers.dev/auth-redirect`
5. กด **Register**

### 2) คัดลอกรหัส 2 ตัว
ในหน้า **Overview** ของแอป คัดลอก
- **Application (client) ID**
- **Directory (tenant) ID**

### 3) เพิ่มสิทธิ์อ่านอีเมล
**API permissions** → **Add a permission** → **Microsoft Graph** → **Delegated permissions** → ติ๊ก **Mail.Read** → **Add permissions**
(`User.Read` มีให้อยู่แล้ว)

### 4) อนุมัติสิทธิ์ (ถ้าบริษัทบังคับ)
หลายบริษัทปิดไม่ให้พนักงานอนุญาตแอปเอง — ถ้าตอนล็อกอินขึ้นว่า *"Need admin approval"*
ให้ผู้ดูแล (Global Admin / Privileged Role Admin) กด **Grant admin consent for <ชื่อบริษัท>** ในหน้า API permissions

### 5) ใส่ในโปรแกรม
แท็บ **⚙️ ตั้งค่า / สำรองข้อมูล** → ใส่ Client ID และ Tenant ID → **บันทึกการตั้งค่า** → **ล็อกอิน Microsoft**

## วิธีใช้
- ลูกค้าต้องมีอีเมลในช่อง "อีเมล / LINE" (ใส่ได้หลายอีเมล คั่นด้วย , )
- เปิดหน้าลูกค้า → ส่วน **📧 อีเมลกับลูกค้า** จะแสดงอีเมลล่าสุด (สูงสุด 25 ฉบับต่ออีเมล) ทั้งที่ลูกค้าส่งมาและที่เราส่งไป
- กด **+ บันทึกเป็น Log** เพื่อเก็บอีเมลนั้นไว้ในประวัติการคุย (เก็บหัวข้อ + เนื้อหาย่อ 500 ตัวอักษร)

## ปัญหาที่พบบ่อย
| ข้อความ | แก้ไข |
|---|---|
| Redirect URI ไม่ตรง (AADSTS50011) | คัดลอก Redirect URI จากโปรแกรมไปใส่ใหม่ ต้องตรงทุกตัวอักษร |
| ต้องเป็น Single-page application (AADSTS9002326) | ลบ URI ที่อยู่ใต้ "Web" แล้วเพิ่มใหม่ใต้ "Single-page application" |
| Need admin approval (AADSTS65001/90094) | ให้ IT ทำข้อ 4 |
| เบราว์เซอร์บล็อกป๊อปอัป | อนุญาตป๊อปอัปสำหรับเว็บนี้ |
| เปิดจากไฟล์ในเครื่องแล้วใช้ไม่ได้ | ต้องเปิดผ่านเว็บ (https) เท่านั้น |
