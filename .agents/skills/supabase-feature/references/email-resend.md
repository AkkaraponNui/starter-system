# ส่งอีเมลยืนยันด้วย Resend (US-6)

ส่งอีเมลจาก Server Action หลังบันทึกการลงทะเบียนสำเร็จ ใช้ Resend ผ่านแพ็กเกจ `resend` ถ้ารุ่นที่ติดตั้งเรียก API ต่างจากนี้ ให้ดูเอกสารของรุ่นนั้น

## แพ็กเกจและ env

```bash
npm install resend server-only
```

| ชื่อ | ค่า | ที่ใช้ |
|---|---|---|
| `RESEND_API_KEY` | รหัสลับจาก Resend → API Keys (ขึ้นต้นด้วย `re_`) | ฝั่ง server เท่านั้น **ห้ามขึ้นต้นด้วย `NEXT_PUBLIC_`** |
| `EMAIL_FROM` (ไม่บังคับ) | เช่น `ระบบลงทะเบียน <noreply@โดเมนที่ยืนยันแล้ว>` | ถ้าไม่ตั้ง ใช้ `onboarding@resend.dev` |

เปิดบรรทัด `RESEND_API_KEY` ใน `.env.example` (ค่าตัวอย่าง ไม่ใช่ค่าจริง) และห้ามอ่านหรือแสดงค่าจาก `.env.local`

## `lib/email.ts`

```ts
import "server-only";
import { Resend } from "resend";

type RegistrationEmail = {
  to: string;
  title: string;
  startsAt: string;
  location: string | null;
};

// ส่งอีเมลยืนยันการลงทะเบียน คืนผลว่าสำเร็จไหม โดยไม่โยน error ออกไป
// เพื่อให้การลงทะเบียนสำเร็จได้แม้ส่งอีเมลไม่ได้ (AC-6.3)
export async function sendRegistrationEmail(
  input: RegistrationEmail,
): Promise<boolean> {
  const apiKey = process.env.RESEND_API_KEY;
  if (!apiKey) return false;

  const when = new Intl.DateTimeFormat("th-TH", {
    dateStyle: "long",
    timeStyle: "short",
    timeZone: "Asia/Bangkok",
  }).format(new Date(input.startsAt));

  try {
    const resend = new Resend(apiKey);
    const { error } = await resend.emails.send({
      from:
        process.env.EMAIL_FROM ?? "ระบบลงทะเบียนกิจกรรม <onboarding@resend.dev>",
      to: input.to,
      subject: `ยืนยันการลงทะเบียน: ${input.title}`,
      text: [
        "คุณลงทะเบียนกิจกรรมสำเร็จแล้ว",
        "",
        `กิจกรรม: ${input.title}`,
        `วันเวลา: ${when}`,
        `สถานที่: ${input.location ?? "-"}`,
      ].join("\n"),
    });
    return !error;
  } catch {
    return false;
  }
}
```

## เรียกใช้ใน Server Action ของการลงทะเบียน

1. ตรวจผู้ใช้ด้วย `supabase.auth.getUser()` แล้วใช้ `user.email` เป็นผู้รับ **ห้ามรับอีเมลผู้รับจากฟอร์มหรือคำขอ** (AC-6.4)
2. บันทึกการลงทะเบียนตามเดิม ถ้าไม่สำเร็จ (ซ้ำ หรือเต็ม) ให้คืนข้อความผิดพลาดเดิมและ**ไม่ส่งอีเมล** (AC-6.2)
3. สำเร็จแล้วค่อยอ่านชื่อ วันเวลา และสถานที่ของกิจกรรมจากฐานข้อมูล แล้วเรียก `sendRegistrationEmail`
4. ถ้าได้ `false` ให้คืนผลว่าลงทะเบียนสำเร็จพร้อมข้อความ "ลงทะเบียนแล้ว แต่ส่งอีเมลยืนยันไม่สำเร็จ" ห้ามยกเลิกการลงทะเบียน (AC-6.3)
5. ห้าม import `lib/email.ts` ในไฟล์ที่ขึ้นต้นด้วย `"use client"` (`server-only` จะทำให้ build ไม่ผ่านถ้าเผลอ import)

ไม่ต้องสร้างตารางหรือ migration ใหม่สำหรับ US-6

## สิ่งที่คุณต้องทำเอง (เขียนให้ผู้ใช้ตามนี้)

1. สมัคร [resend.com](https://resend.com) ด้วย**อีเมลเดียวกับที่ใช้ล็อกอินเว็บของเรา** (แบบฟรีพอสำหรับทดลอง)
2. Resend → **API Keys → Create API Key** → สิทธิ์ **Sending access** → คัดลอกรหัส (เห็นได้ครั้งเดียว)
3. เปิด `.env.local` เพิ่มบรรทัด `RESEND_API_KEY=` แล้ววางรหัส → บันทึก (**ห้ามเติม `NEXT_PUBLIC_` ข้างหน้า** และห้ามวางรหัสในแชท)
4. หยุด `npm run dev` (กด Ctrl+C) แล้วรันใหม่ เพื่อให้อ่านค่าใหม่
5. ลงทะเบียนกิจกรรม 1 รายการ แล้วเปิดกล่องอีเมล (ดูในโฟลเดอร์สแปมด้วย)
6. ตอน deploy: Vercel → Settings → Environment Variables → เพิ่ม `RESEND_API_KEY` → **Redeploy**

**ข้อจำกัดช่วงทดลอง:** ผู้ส่ง `onboarding@resend.dev` ส่งถึงได้เฉพาะอีเมลที่ใช้สมัคร Resend ถ้าลงทะเบียนด้วยอีเมลอื่น หน้าจอจะแจ้งว่าส่งอีเมลไม่สำเร็จ ซึ่งถูกต้องตาม AC-6.3 ถ้าจะส่งถึงทุกคน ต้องยืนยันโดเมนของตัวเองใน Resend → Domains แล้วตั้ง `EMAIL_FROM`
