# AGENTS.md — โปรเจกต์ Sass

Repo นี้เป็นไลบรารี SCSS แยกต่างหาก คอมไพล์เป็น CSS แล้วให้โปรเจกต์ PHP/HTML ภายนอกลิงก์ไปใช้ทีละไฟล์ ใช้ Bootstrap 5.3 เป็นฐานของ token

## 1. ก่อนเริ่มงาน

- อ่าน [`CONTEXT.md`](CONTEXT.md) ทุกครั้งก่อนเสนอหรือแก้สไตล์ เพราะมีแผนที่ไฟล์ตามโดเมน แนวทางตั้งชื่อ class และ checklist
- เปิดไฟล์ `.scss` ที่จะแก้ และไฟล์ข้างเคียงใน `src/sass/` เพื่อเขียนตามแพทเทิร์นเดิม
- ถ้าข้อมูลใน `CONTEXT.md` หรือ `README.md` ไม่ตรงกับ `package.json` หรือโค้ดจริง ให้ยึดโค้ดจริงและบอกผู้ใช้

## 2. ลำดับความสำคัญของเอกสาร

1. โค้ดจริงใน `src/sass/` และ `package.json`
2. `CONTEXT.md`
3. `README.md`
4. `SKILL.md` และ `.agents/skills/scss/*` เป็นแนวทาง SCSS ทั่วไปเท่านั้น (เช่น `@use`, cascade layers, HSL token, cva) ถ้าขัดกับข้อ 1–3 ให้ทำตามข้อ 1–3

## 3. แพทเทิร์น SCSS ของโปรเจกต์

- ไฟล์ที่ต้องใช้ token ให้ขึ้นต้นด้วย header มาตรฐานใน `CONTEXT.md` หัวข้อ "หัวไฟล์มาตรฐาน (Bootstrap)" และใช้ `@import` ตามไฟล์เดิม
- ใช้ตัวแปร Bootstrap (`$primary`, `$theme-colors`, `$grays`, `$border-radius-*`) แทนค่า hex
- ปรับโทนสีด้วย `tint-color()`, `shade-color()` หรือ `color.scale()` / `color.change()` จาก `sass:color`
- หนึ่งไฟล์ต่อหนึ่งหน้าหรือ component ไฟล์ partial ให้ขึ้นต้นด้วย `_`
- จัดรูปแบบตาม `.editorconfig`: indent เป็น tab ขนาด 4 และขึ้นบรรทัดแบบ CRLF

## 4. ขอบเขตที่ต้องยึด

- แก้เฉพาะไฟล์ที่เกี่ยวกับงานที่ขอ และรักษาโค้ดหรือการแก้ไขอื่นใน working tree ที่ไม่เกี่ยวข้องไว้
- คง `@import` และโครงสร้างแยกไฟล์ไว้ การย้ายไปใช้ `@use` หรือรวมเป็น entry point เดียว ทำเมื่อผู้ใช้สั่งเท่านั้น เพราะระบบหลักลิงก์ CSS ทีละไฟล์
- ขอยืนยันกับผู้ใช้ก่อนแก้ `src/sass/bootstrap_bs5/_variables.scss` หรือ partial ที่หลายไฟล์ import ร่วมกัน เพราะกระทบ CSS ทุกหน้าที่ใช้
- `dist/` เป็นผล build ที่อยู่ใน `.gitignore` ให้แก้ที่ `src/sass/` เสมอ
- ไม่ commit, push หรือเปลี่ยน dependencies ถ้าผู้ใช้ไม่ได้สั่ง

## 5. ขั้นตอนทำงาน

1. อ่านไฟล์เป้าหมายและแถวที่เกี่ยวข้องในแผนที่ไฟล์ของ `CONTEXT.md`
2. แก้โดยคงสไตล์และรูปแบบรอบข้างไว้
3. ตรวจงานด้วย `npm run build` (หรือคอมไพล์ไฟล์เดียว) ต้องจบโดยไม่มี error ส่วน deprecation warning ที่มีอยู่เดิม (`import`, `global-builtin`, `color-functions` ซึ่งส่วนใหญ่มาจาก Bootstrap) ไม่ต้องแก้ ถ้ามี warning ประเภทอื่นที่เกิดจากโค้ดที่เพิ่งแก้ ให้แจ้งผู้ใช้
4. ถ้าเพิ่มไฟล์ SCSS ใหม่ที่สำคัญ ให้เพิ่มแถวในแผนที่ไฟล์ของ `CONTEXT.md`
5. รายงานไฟล์ที่แก้พร้อมลิงก์ และผลการ build ที่รันจริง

## 6. คำสั่ง

- Build ทั้งหมด (compressed): `npm run build` ใช้ตรวจ error หลังแก้
- Watch ระหว่างพัฒนา (expanded): `npm run dev`
- คอมไพล์ไฟล์เดียว: `npx sass src/sass/<name>.scss dist/sass/<name>.css --no-source-map`
- คำสั่งอื่นดูที่ `package.json`
