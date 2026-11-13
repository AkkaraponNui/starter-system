# รูปแบบ RLS ที่ใช้บ่อย

ปรับชื่อตารางและคอลัมน์ให้ตรง `docs/design.md` ทุกครั้ง ไม่คัดลอกทั้งชุดโดยไม่ตรวจ

## ข้อมูลสาธารณะ อ่านได้ทุกคน

```sql
alter table public.events enable row level security;

create policy "events: everyone can read"
on public.events for select
to anon, authenticated
using (true);
```

ถ้าไม่มี policy สำหรับ insert, update หรือ delete การกระทำนั้นจะถูกปฏิเสธ (deny by default)

## ข้อมูลของเจ้าของ

```sql
alter table public.registrations enable row level security;

create policy "registrations: owner can read"
on public.registrations for select
to authenticated
using ((select auth.uid()) = user_id);

create policy "registrations: owner can insert"
on public.registrations for insert
to authenticated
with check ((select auth.uid()) = user_id);

create policy "registrations: owner can delete"
on public.registrations for delete
to authenticated
using ((select auth.uid()) = user_id);

create index if not exists registrations_user_id_idx on public.registrations (user_id);
```

- `using` กรองแถวที่มองเห็นหรือแก้ได้ ส่วน `with check` ตรวจแถวใหม่ที่กำลังจะเขียน
- policy ของ update ต้องมีทั้ง `using` และ `with check`

## ห้ามลงซ้ำ

```sql
alter table public.registrations
  add constraint registrations_event_user_unique unique (event_id, user_id);
```

## ตัวเลขรวมข้ามแถว (เช่น ที่นั่งคงเหลือ)

ผู้ใช้อ่านได้เฉพาะการลงทะเบียนของตัวเอง จึงนับของคนอื่นตรง ๆ ไม่ได้ ให้คืนเฉพาะตัวเลข

```sql
create or replace function public.event_seats_left(p_event_id uuid)
returns integer
language sql
stable
security definer
set search_path = ''
as $$
  select e.capacity - count(r.id)::int
  from public.events e
  left join public.registrations r on r.event_id = e.id
  where e.id = p_event_id
  group by e.capacity;
$$;

revoke all on function public.event_seats_left(uuid) from public;
grant execute on function public.event_seats_left(uuid) to anon, authenticated;
```

## ตรวจที่นั่งเต็มตอนลงทะเบียน

ตรวจในฐานข้อมูลด้วย trigger `before insert` ที่ล็อกแถวของกิจกรรม (`select ... for update`) แล้วนับจำนวน ถ้าเต็มให้ `raise exception` พร้อมข้อความที่แอปแปลงเป็นข้อความสำหรับผู้ใช้ได้ การล็อกแถวช่วยกันกรณีหลายคนกดลงทะเบียนพร้อมกัน

## ห้ามทำ

- `disable row level security` เพื่อให้ query ผ่าน
- policy `using (true)` บนตารางข้อมูลส่วนบุคคล
- ใช้ secret key ฝั่ง browser เพื่อหลบ RLS
