# ระบบลงทะเบียนกิจกรรม (โปรเจกต์ตั้งต้นสำหรับ workshop)

โปรเจกต์นี้ตั้ง Next.js และเชื่อม Supabase ไว้ให้แล้ว คุณจะใช้ Codex สร้างฟีเจอร์ต่อทีละขั้น

## สิ่งที่ต้องมีก่อน
- ติดตั้ง VS Code, Node.js (รุ่น LTS) และ Git แล้ว
- ติดตั้ง Codex ใน VS Code และล็อกอินแล้ว
- ติดตั้ง Workshop Skills แล้ว (พิมพ์ `$` ในช่องแชทของ Codex ต้องเห็น `sdlc-spec`, `supabase-feature` และตัวอื่น)
- มีบัญชี GitHub, Supabase และ Vercel

## เริ่มใช้งาน (ทำครั้งเดียว ประมาณ 10 นาที)

1. **เอาโปรเจกต์มาไว้ในเครื่อง**
   บน GitHub กดปุ่ม **Use this template → Create a new repository** ตั้งชื่อ repo ของคุณ
   แล้วใน VS Code กด **Clone Git Repository** วางลิงก์ repo ที่เพิ่งสร้าง เลือกโฟลเดอร์ แล้วกด **Open**
2. **ติดตั้งส่วนประกอบ**
   ใน VS Code เปิด Terminal (เมนู **Terminal → New Terminal**) แล้วพิมพ์

   ```
   npm install
   ```

3. **ใส่ค่าการเชื่อม Supabase**
   - คัดลอกไฟล์ `.env.example` แล้วเปลี่ยนชื่อสำเนาเป็น `.env.local`
   - ใน Supabase เปิดโปรเจกต์ → **Project Settings → API Keys** คัดลอก **Publishable key**
   - คัดลอก **Project URL** (ปุ่ม **Connect** ด้านบนของหน้าโปรเจกต์)
   - วางทั้งสองค่าใน `.env.local` แทนค่าตัวอย่าง แล้วบันทึกไฟล์
4. **เปิดเว็บบนเครื่อง**

   ```
   npm run dev
   ```

   เปิด http://localhost:3000 ต้องเห็นกล่องสีเขียว **"เชื่อม Supabase สำเร็จ"**
   ถ้าเห็นกล่องสีเหลือง ให้อ่านข้อความในกล่องแล้วแก้ตามนั้น

## ทำงานต่อกับ Codex

| ขั้น | พิมพ์ใน Codex |
|---|---|
| ออกแบบ | `ใช้ $sdlc-spec อ่าน docs/plan.md และ docs/requirements.md แล้วเขียน docs/design.md โดยไม่แก้สองไฟล์แรก` |
| ดูกิจกรรม | `ใช้ $supabase-feature ทำ US-2` |
| ล็อกอิน | `ใช้ $supabase-feature ทำ US-1` |
| ลงทะเบียน | `ใช้ $supabase-feature ทำ US-3` |
| งานเสริม | `ใช้ $supabase-feature ทำ US-4` แล้ว `ทำ US-5` |
| ทดสอบ | `ใช้ $acceptance-test ทำรายการทดสอบของ US ที่ทำแล้ว` |
| ก่อน deploy | `ใช้ $security-check ตรวจโปรเจกต์ รายงานอย่างเดียว` แล้ว `ใช้ $vercel-deploy` |

ทุกครั้งที่ Codex บอกว่ามีไฟล์ SQL ให้รัน: เปิด Supabase → **SQL Editor → New query** วางเนื้อหาไฟล์ แล้วกด **Run**

## บันทึกงาน (commit)
ใน VS Code กดไอคอน **Source Control** ที่แถบซ้าย พิมพ์ข้อความสั้น ๆ ว่าทำอะไร กด **Commit** แล้วกด **Sync Changes** เพื่อส่งขึ้น GitHub

## ห้ามทำ
- ห้ามส่งไฟล์ `.env.local` ให้ใคร หรือวางค่าในแชท
- ห้ามปิด RLS ใน Supabase แม้ข้อมูลจะไม่แสดง ให้ถาม Codex ด้วย `$bug-fix` แทน
