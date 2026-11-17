# คู่มือเริ่มต้นใช้งาน IMS บน Vercel (ฉบับทำตามทีละขั้น)

คู่มือนี้พาเปลี่ยนจาก Google Apps Script มาเป็นเว็บไซต์ที่ Vercel ให้บริการ โดยหน้าเว็บเรียก API ฝั่งเซิร์ฟเวอร์ในโฟลเดอร์ `api/` และ API ใช้ Google Sheets API อ่าน/เขียน Google Sheet ของคุณ ไม่ต้องเปิดเผยกุญแจ Google ให้ผู้ใช้เว็บเห็น

ถ้าต้องการวิธีตั้งค่า Google Sheets API และสร้าง Service Account อย่างละเอียดทีละคลิก ให้อ่าน [คู่มือ Google Sheets และ Service Account](GOOGLE-SHEETS-SETUP-TH.md) เพิ่มเติม

สำหรับขั้นตอนสร้าง Workload Identity Pool, OIDC Provider, Audience, Attribute condition และมอบสิทธิ์ Workload Identity User แบบละเอียดเป็นพิเศษ ให้อ่าน [คู่มือ Workload Identity แบบจับมือทำ](WORKLOAD-IDENTITY-SETUP-TH.md)

> **สำคัญ:** ไฟล์ `Code.gs` และ `index.html` ที่อยู่โฟลเดอร์หลักยังเป็นต้นฉบับ Google Apps Script และไม่ได้ถูกลบ เวอร์ชัน Vercel อยู่ในโฟลเดอร์ `vercel-app` แยกต่างหาก

## ไฟล์ในโฟลเดอร์นี้คืออะไร

- `index.html` คือหน้าระบบคลังอะไหล่ที่ทำงานบนเว็บ
- `api/ims.js` คือเซิร์ฟเวอร์ API ที่เชื่อมต่อ Google Sheets และตรวจสิทธิ์ Admin/Staff
- `setup.html` คือหน้าสร้างชีตเริ่มต้น ใช้ครั้งเดียวเท่านั้น
- `package.json` ระบุไลบรารี Google API ที่ Vercel ต้องติดตั้ง
- `.env.example` แสดงชื่อตัวแปรที่ต้องตั้ง โดยไม่มีรหัสลับจริง
- `.gitignore` กันไฟล์กุญแจและค่าลับไม่ให้เผลอส่งขึ้น GitHub

## ก่อนเริ่ม ต้องมีอะไรบ้าง

เตรียมบัญชี Google, บัญชี GitHub และบัญชี Vercel (ใช้ GitHub เข้าสู่ระบบ Vercel ได้) ถ้าจะทำตามวิธีในคู่มือนี้ ให้ติดตั้ง GitHub Desktop ด้วย คุณไม่จำเป็นต้องติดตั้งโปรแกรมเขียนโค้ดเพื่อ Deploy ผ่านหน้าเว็บ แต่ต้องมีโฟลเดอร์ `vercel-app` ที่สร้างไว้ให้แล้ว

Google Sheets API ต้องเปิดใน Google Cloud และบัญชีบริการ (Service Account) ต้องได้รับสิทธิ์กับไฟล์ Sheet ที่ต้องการใช้ Google อธิบายว่าบัญชีบริการเข้าถึงไฟล์เฉพาะได้โดยแชร์เอกสารให้กับอีเมลของบัญชีนั้นโดยตรง [1] [2]

## ขั้นที่ 1: สร้าง Google Sheet สำหรับระบบ

1. เปิด [Google Sheets](https://sheets.google.com) แล้วกดสร้างสเปรดชีตเปล่า
2. ตั้งชื่อเช่น `IMS Spare Parts Database`
3. คัดลอก **Spreadsheet ID** จาก URL ของชีต URL จะมีรูปแบบ `https://docs.google.com/spreadsheets/d/ส่วนนี้คือ-ID/edit` ให้คัดลอกเฉพาะข้อความระหว่าง `/d/` กับ `/edit` เก็บไว้ก่อน

ยังไม่ต้องสร้างแท็บเอง หน้า `setup.html` จะสร้างแท็บและหัวตารางให้หลังตั้งค่าการเชื่อมต่อเสร็จ

## ขั้นที่ 2: เชื่อม Google Cloud โดยไม่ใช้ JSON Key

ใช้ Workload Identity Federation (OIDC) ตาม [คู่มือตั้ง Google Sheets แบบไม่ใช้ JSON Key](GOOGLE-SHEETS-SETUP-TH.md) วิธีนี้สร้าง Service Account แต่ไม่สร้าง key แล้วให้ Vercel ขอ credential อายุสั้นผ่าน OIDC แทน Vercel ระบุว่า OIDC federation รองรับทุกแผน [1]

คู่มือแยกจะพาเปิด Google Sheets API และ IAM Service Account Credentials API, สร้าง Service Account, สร้าง Workload Identity Pool/Provider, ให้สิทธิ์เฉพาะ Vercel project/environment แล้วแชร์ชีตให้ service account เป็น Editor

## ขั้นที่ 3: สร้าง GitHub repository แบบส่วนตัว

Vercel จะเชื่อมกับ repository แล้ว Deploy เมื่อมีการส่งเวอร์ชันใหม่ขึ้น GitHub [3]

1. ติดตั้งและเปิด **GitHub Desktop** แล้วเข้าสู่ระบบ GitHub
2. เลือก **File → New repository…**
3. ตั้งชื่อ `ims-spare-parts-vercel`
4. ตรง **Local path** เลือกโฟลเดอร์ `vercel-app` ที่อยู่ในโปรเจกต์นี้ เช่น `C:\Users\ASUS\OneDrive\Desktop\Github-Webapp\IMS - Spare Parts & Service Inventory System\vercel-app`
5. ตั้ง repository เป็น **Private** หากมีตัวเลือกนี้ แล้วกด **Create repository**
6. GitHub Desktop จะแสดงไฟล์ที่เพิ่มเข้ามา ใส่ข้อความในช่อง Summary เช่น `Create Vercel IMS app` แล้วกด **Commit to main**
7. กด **Publish repository** และตรวจอีกครั้งว่าเป็น **Private**

**ตรวจให้แน่ใจ:** อย่า Commit ไฟล์ `.env` หรือ JSON key หากเคยสร้างไว้ก่อนหน้า ลบไฟล์จาก working directory และ Git history ตามความจำเป็น; วิธี OIDC ในโปรเจกต์นี้ไม่ต้องใช้ JSON key

## ขั้นที่ 4: สร้างเว็บไซต์บน Vercel

1. เปิด [Vercel](https://vercel.com) แล้วเลือก **Continue with GitHub**
2. อนุญาตให้ Vercel มองเห็น repository `ims-spare-parts-vercel` ถ้าไม่พบ ให้กลับไปอนุญาต repository ในการตั้งค่า GitHub ของ Vercel
3. ที่หน้า Dashboard กด **Add New… → Project** แล้วเลือก repository นี้ กด **Import**
4. ถ้าคุณสร้าง repository โดยตรงจากโฟลเดอร์ `vercel-app` ให้ตั้ง **Root Directory** เป็น `./` หรือค่าเริ่มต้น หากเอาโฟลเดอร์นี้ไปวางไว้ใต้ repository อื่น ให้ตั้ง Root Directory เป็น `vercel-app`
5. เลือก Framework Preset เป็น **Other** ถ้าระบบถาม ส่วน Build Command และ Output Directory ให้คงค่าเริ่มต้น/ปล่อยว่าง เพราะโปรเจกต์นี้ใช้ไฟล์หน้าเว็บธรรมดากับ Vercel Functions ใน `api/` ไม่ได้ใช้ React build
6. **ยังไม่ต้องกด Deploy** ให้เปิดส่วน **Environment Variables** ก่อน

Vercel สร้างฟังก์ชัน Node.js จากไฟล์ใน `api/` และติดตั้งแพ็กเกจที่ระบุไว้ใน `package.json` ให้ในขั้น Deploy [4]

## ขั้นที่ 5: ใส่ค่าลับใน Vercel

เพิ่มตัวแปรต่อไปนี้ในหน้าตั้งค่า Environment Variables ของ Project (รายละเอียดที่มาของแต่ละค่าอยู่ในคู่มือ OIDC):

- `SPREADSHEET_ID` — รหัสชีตที่คัดลอกไว้ในขั้นที่ 1
- `GCP_PROJECT_NUMBER` — หมายเลขโปรเจกต์ Google Cloud
- `GCP_WORKLOAD_IDENTITY_POOL_ID` — Pool ID
- `GCP_WORKLOAD_IDENTITY_POOL_PROVIDER_ID` — Provider ID
- `GCP_SERVICE_ACCOUNT_EMAIL` — อีเมล Service Account ที่แชร์ให้ Google Sheet
- `GCP_AUDIENCE` — Audience URI ที่คัดลอกจาก OIDC Provider ใน Google Cloud
- `SESSION_SECRET` — สุ่มข้อความยาวอย่างน้อย 32 ตัวอักษร ใช้เป็นกุญแจลงลายเซ็น Session ห้ามใช้รหัสผ่านเดียวกับบัญชีอื่น
- `SETUP_KEY` — ตั้งข้อความลับอีกชุดหนึ่ง ใช้เฉพาะตอนเริ่มสร้างชีต อย่าใช้ค่าเดียวกับ `SESSION_SECRET`

เลือก Environment เป็น **Production, Preview และ Development** ถ้าหน้าจอมีตัวเลือกนี้ แล้วกด Save ทุกตัวแปร ค่า Environment Variable ถูกเก็บแยกจากโค้ด แต่การเปลี่ยนค่าจะมีผลกับ Deployment ใหม่ ดังนั้นหลังแก้ค่าให้กด Redeploy [5]

**ห้ามใส่**`SESSION_SECRET` และ `SETUP_KEY` ในหน้าเว็บหรือ GitHub และอย่าเก็บ Service Account JSON key ไว้ในโปรเจกต์ วิธี OIDC ใช้ค่าระบุ Provider ใน Environment Variables และ Vercel จะใช้ OIDC token อายุสั้นเมื่อ API ทำงาน [5]

## ขั้นที่ 6: Deploy ครั้งแรก

หลังเพิ่ม Environment Variables ครบแล้ว ให้กด **Deploy** และรอจนสถานะสำเร็จ เมื่อเสร็จ Vercel จะแสดง URL เช่น `https://ชื่อโปรเจกต์.vercel.app` เปิด URL นี้ในแท็บใหม่

## ขั้นที่ 7: สร้างแท็บชีตเริ่มต้น

1. นำ URL ของเว็บไซต์มาต่อท้าย `/setup.html` เช่น `https://ชื่อโปรเจกต์.vercel.app/setup.html`
2. พิมพ์ค่า `SETUP_KEY` ที่ตั้งไว้ใน Vercel ลงในช่อง แล้วกดปุ่มสร้างชีต
3. ถ้าสำเร็จ จะเห็นข้อความว่าระบบสร้าง `Users`, `Parts_Inventory`, `Transactions` และ `Audit_Logs` แล้ว
4. กลับไปหน้าแรกของเว็บไซต์ แล้วเข้าสู่ระบบด้วย `admin` / `1234` หรือ `staff` / `1234`
5. บัญชีตัวอย่างเป็นเพียงค่าเริ่มต้น ไม่ควรปล่อยไว้ก่อนเปิดให้ทีมใช้งาน เมื่อเข้าสู่ระบบครั้งแรก รหัสผ่านเดิมจะถูกแปลงเป็นแฮชในชีตโดยอัตโนมัติ ถ้าต้องการเปลี่ยนรหัสเริ่มต้น ให้แก้ช่อง Password ของบัญชีนั้นในแท็บ Users ด้วยรหัสใหม่ แล้วเข้าสู่ระบบด้วยรหัสใหม่นั้นหนึ่งครั้ง ระบบจะแปลงค่าเป็นแฮชให้อัตโนมัติ
6. เพื่อปิดการตั้งค่าครั้งเดียว ให้กลับไป **Vercel → Project → Settings → Environment Variables** ลบ `SETUP_KEY` แล้ว Redeploy จากนั้นลบ `setup.html` ออกจาก GitHub Desktop ทำ Commit และ Push การลบ `SETUP_KEY` ทำให้เรียก setup ซ้ำไม่ได้ แม้ยังมี URL หน้าตั้งค่าอยู่

## ขั้นที่ 8: ทดสอบระบบ

เข้าสู่ระบบ Admin แล้วเพิ่มอะไหล่ทดลองหนึ่งรายการ ลองค้นหาและบันทึกรับเข้า จากนั้นออกจากระบบแล้วเข้าเป็น Staff ตรวจว่าราคาทุนไม่แสดง ลองเบิกออกโดยกรอกเลขที่ใบงาน และเปิดหน้า History ดูรายการ ถ้าทุกอย่างทำงาน ให้ตรวจว่าแถวใหม่ปรากฏใน Google Sheet ด้วย

หากจะใช้กล้อง ให้เปิด URL Vercel โดยตรงใน Chrome หรือเบราว์เซอร์ แล้วกด Allow เมื่อมีหน้าต่างขอสิทธิ์กล้อง หน้าเว็บที่ถูกฝังในแผงด้านข้างอาจถูกนโยบายของกรอบปิดกั้นกล้องได้

## วิธีอัปเดตเว็บไซต์ภายหลัง

หลังแก้ไฟล์ในโฟลเดอร์ `vercel-app` ให้เปิด GitHub Desktop ตรวจรายการเปลี่ยนแปลง ใส่ข้อความ Commit แล้วกด **Commit to main** และ **Push origin** Vercel เชื่อม GitHub แล้วสร้าง Deployment ใหม่ตามการ Push [3] เมื่อ Deployment สำเร็จ เปิด URL เดิมเพื่อดูเวอร์ชันล่าสุด

## ถ้ามีปัญหา ให้ไล่เช็กแบบนี้

- **ขึ้น `ยังไม่ได้ตั้งค่าตัวแปร ...`** — ตรวจว่าเพิ่ม Environment Variable สะกดตรง และ Redeploy หลังเพิ่มแล้ว
- **ขึ้น 403 หรือ permission denied จาก Google** — แชร์ Google Sheet ให้ Service Account ด้วยสิทธิ์ Editor และตรวจว่า principal ของ Vercel ได้ Role `Workload Identity User` บนบัญชีบริการ
- **ขึ้น `Requested entity was not found`** — ตรวจ Spreadsheet ID และตรวจว่าไฟล์ชีตถูกแชร์ให้ Service Account
- **ขึ้น `API has not been used` หรือ `disabled`** — กลับไป Google Cloud ตรวจว่าเปิด Google Sheets API และ IAM Service Account Credentials API ในโปรเจกต์ที่ตั้งค่าไว้
- **ขึ้น `Invalid audience` หรือ WIF exchange ล้มเหลว** — ตรวจ `GCP_AUDIENCE`, Issuer URL, Team slug, Pool ID, Provider ID และ attribute condition ตาม [คู่มือ OIDC](GOOGLE-SHEETS-SETUP-TH.md)
- **ขึ้น `Setup key ไม่ถูกต้อง`** — ค่าในช่องต้องตรงกับ `SETUP_KEY` ใน Vercel ทุกตัวอักษร ถ้าเพิ่งเปลี่ยนตัวแปร ให้ Redeploy ก่อน
- **เปิดเว็บได้แต่เข้าใช้งานไม่ได้** — ใน Vercel เปิดแท็บ **Deployments → deployment ล่าสุด → Functions / Logs** ดูข้อความผิดพลาด แล้วกลับมาเช็กตัวแปรที่เกี่ยวข้อง
- **แก้ Environment Variables แล้วเว็บยังใช้ค่าเก่า** — สร้าง Deployment ใหม่ เพราะค่าใหม่ไม่ย้อนกลับไปแก้ Deployment เก่า [5]

## ข้อควรทราบเรื่องความปลอดภัยและข้อมูล

Google Sheet เก็บข้อมูลคลังและประวัติจริง ควรจำกัดการแชร์ Sheet ให้เฉพาะผู้เกี่ยวข้อง ดาวน์โหลดสำรองเป็นระยะ และเปลี่ยนรหัสตัวอย่างก่อนให้พนักงานใช้งาน รุ่นนี้แฮชรหัสผ่านผู้ใช้ใหม่ และอัปเกรดรหัสผ่านเก่าหลังล็อกอินสำเร็จ แต่ Session ในหน้าเว็บเก็บใน `localStorage` จึงควรใช้เครื่องที่เชื่อถือได้และกด Logout ทุกครั้งหลังใช้งาน

Google Sheets เหมาะกับระบบเริ่มต้นหรือปริมาณงานไม่มาก แต่การทำรายการจำนวนมากพร้อมกันอาจมีข้อจำกัดด้านการชนกันของการอัปเดตและโควตา API หากระบบกลายเป็นระบบงานหลักที่มีผู้ใช้จำนวนมาก ควรย้ายฐานข้อมูลไปฐานข้อมูลธุรกรรมโดยตรง

## แหล่งข้อมูลทางการ

[1]: https://vercel.com/docs/oidc/gcp "Vercel OpenID Connect Federation with Google Cloud"
[2]: https://developers.google.com/workspace/guides/create-credentials "Google Workspace credentials and service accounts"
[3]: https://vercel.com/docs/git/vercel-for-github "Deploying GitHub projects with Vercel"
[4]: https://vercel.com/docs/functions/runtimes/node-js "Using the Node.js Runtime with Vercel Functions"
[5]: https://vercel.com/docs/environment-variables "Vercel environment variables"
