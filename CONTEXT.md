# CONTEXT — โปรเจกต์ Sass

เอกสารนี้สำหรับ **นักพัฒนาและ AI assistant** ให้เข้าใจโปรเจกต์ก่อนแก้ไขโค้ด  
อ่านไฟล์นี้ก่อน แล้วค่อยดู `README.md` (สั้น) หรือไฟล์ `.scss` ที่เกี่ยวข้อง

---

## โปรเจกต์นี้คืออะไร

- Repo นี้เป็น **ไลบรารีสไตล์ SCSS แยกต่างหาก** ไม่ใช่แอปเต็มรูปแบบ
- แต่ละไฟล์ `.scss` มักผูกกับ **หน้าหรือฟีเจอร์ในระบบหลัก** (เช่น ตาราง FSN, ตารางเวร, แนบไฟล์, SweetAlert)
- คอมไพล์ออกเป็น `.css` แล้วนำไป **ลิงก์ในโปรเจกต์ PHP / HTML อื่น** (นอก repo นี้)
- ใช้ **Bootstrap 5.3** เป็นฐานตัวแปรสี, spacing, component tokens

---

## โครงสร้างโฟลเดอร์

```
sass/
├── CONTEXT.md          ← เอกสารนี้
├── README.md           ← คำอธิบายสั้น
├── package.json        ← สคริปต์ build
├── src/
│   ├── sass/           ← แหล่ง SCSS หลัก (แก้ที่นี่)
│   │   ├── bootstrap_bs5/   ← override functions/variables ของ Bootstrap
│   │   ├── select2/         ← สไตล์ Select2
│   │   ├── shift_schedule/  ← โมดูลตารางเวร
│   │   └── *.scss             ← ไฟล์ต่อหน้า/ฟีเจอร์ (หนึ่งไฟล์ ≈ หนึ่ง CSS)
│   └── cli/
│       └── gulp.sass.mjs      ← pipeline แบบ Gulp (ทางเลือก)
└── dist/               ← ผลลัพธ์ CSS (ไม่ commit — อยู่ใน .gitignore)
    ├── sass/           ← จาก `npm run build:sass`
    └── gulp/           ← จาก Gulp (ถ้าใช้)
```

---

## วิธี build

| คำสั่ง | ทำอะไร |
|--------|--------|
| `npm install` | ติดตั้ง dependencies (ครั้งแรก) |
| `npm run dev:sass` | watch คอมไพล์ `src/sass` → `dist/sass` (expanded) |
| `npm run build:sass` | build ทั้งหมดแบบ compressed |
| `npm run dev:gulp` | watch ผ่าน Gulp |
| `npm run build:gulp` | build ผ่าน Gulp |

**ความต่างสำคัญ**

- **`npm run build:sass`** — คอมไพล์ **ทุกไฟล์ `.scss` ใน `src/sass` รวม subfolder** → `dist/sass/` (โครงสร้าง mirror กับต้นทาง)  
  ตัวอย่าง: `src/sass/badge_style.scss` → `dist/sass/badge_style.css`
- **Gulp** (`gulp.sass.mjs`) — คอมไพล์เฉพาะ **`src/sass/*.scss` ระดับ root** ไป `dist/gulp/`  
  ไฟล์ใน `shift_schedule/` ต้องใช้ `npm run build:sass` (หรือคอมไพล์ไฟล์เดี่ยวด้วย `npx sass`)

คอมไพล์ไฟล์เดียว (debug):

```bash
npx sass src/sass/badge_style.scss dist/sass/badge_style.css --no-source-map
```

---

## แนวทางเขียน SCSS ในโปรเจกต์นี้

### 1. หัวไฟล์มาตรฐาน (Bootstrap)

ไฟล์ส่วนใหญ่ขึ้นต้นแบบนี้:

```scss
@use 'sass:color';
// Bootstrap 5
@import "bootstrap_bs5/functions";
@import "../../node_modules/bootstrap/scss/variables";
@import "../../node_modules/bootstrap/scss/mixins";
```

- แก้ token สี/ธีมระดับโปรเจกต์ที่ `src/sass/bootstrap_bs5/_variables.scss`
- ใช้ตัวแปร Bootstrap เช่น `$primary`, `$theme-colors`, `$grays`, `$badge-*`, `$border-radius-*`
- ปรับโทนสีอ่อน/เข้มด้วยฟังก์ชัน Bootstrap: **`tint-color()`**, **`shade-color()`**  
  หรือ module `sass:color` เช่น **`color.scale()`**, **`color.change()`** (ดู `risk.scss`, `custom.scss`)

> โปรเจกต์ยังใช้ `@import` อยู่ (ตามแพทเทิร์นเดิม) แม้ Sass จะ deprecate — อย่า refactor ทั้ง repo เป็นครั้งเดียวถ้าไม่ได้ขอ

### 2. หนึ่งไฟล์ = หนึ่งชุดสไตล์

- ตั้งชื่อไฟล์ตาม **หน้าหรือ component** เช่น `swal_create_fsn.scss`, `modal_edit_fsn.scss`, `badge_style.scss`
- ไม่มี entry point รวม (`main.scss`) — แต่ละหน้าในระบบหลักเลือกลิงก์ CSS ที่ต้องการเอง
- ไฟล์ที่ขึ้นต้นด้วย `_` เป็น partial (ไม่ถูกคอมไพล์เป็นคss แยก) เช่น `bootstrap_bs5/_variables.scss`

### 3. การตั้งชื่อ class

| รูปแบบ | ตัวอย่าง | หมายเหตุ |
|--------|----------|----------|
| ตามหน้า / wrapper | `.swal2-container-fsn` | scope ภายในฟีเจอร์ |
| variant สี | `.badge-soft-primary`, `.btn-minimal-blue` | สร้างจาก `@each` + map |
| soft / minimal | `tint-color` + `shade-color` | โทนอ่อน — ดู `badge_style.scss`, `button_minimal.scss` |
| outline soft | `.btn-outline-soft-primary` | ใช้ `color.scale()` — ดู `risk.scss` |

### 4. EditorConfig

- indent: **tab**, size 4  
- line ending: **CRLF**  
- ไม่บังคับ final newline

---

## แผนที่ไฟล์ (โดยโดเมน)

| โฟลเดอร์ / ไฟล์ | ใช้กับ |
|-----------------|--------|
| `badge_style.scss` | Badge โทนอ่อน `.badge-soft-*` |
| `button_minimal.scss` | ปุ่ม minimal `.btn-minimal-*` |
| `swal_create_fsn.scss`, `modal_edit_fsn.scss`, `asset_running_fsn.scss` | งาน FSN |
| `shift_schedule/*` | ตารางเวร / ลายเซ็น / scheduling |
| `attach_files.scss`, `dropdown_filter.scss`, `filter_multiple_selected.scss` | UI ฟอร์ม / ตัวกรอง |
| `table_gtw.scss`, `style_dom_pdf.scss`, `style_dom_table.pdf.scss` | ตาราง / PDF |
| `risk.scss`, `supply.scss`, `custom.scss` | หน้าทั่วไป + ปุ่ม soft แบบเก่า |
| `select2/` | ธีม Select2 |
| `vehicle_normal.scss`, `leader_vehicle_allocate.scss` | ยานพาหนะ |
| `hovercard.shift.scss` | hover card เวร |

รายการอาจเพิ่มตามหน้าใหม่ในระบบหลัก — **อัปเดตตารางนี้เมื่อเพิ่มไฟล์**

---

## ตัวอย่างการใช้งานใน HTML (ฝั่งระบบหลัก)

```html
<link rel="stylesheet" href="/path/to/dist/sass/badge_style.css">

<span class="badge badge-soft badge-soft-success">อนุมัติแล้ว</span>
<button type="button" class="btn btn-minimal-blue">บันทึก</button>
```

Bootstrap CSS หลักต้องโหลดแยกจากโปรเจกต์หลัก — repo นี้ **ไม่รวม** Bootstrap compiled เต็มชุด

---

## สิ่งที่ควรทำ / ไม่ควรทำ (สำหรับ AI)

**ควรทำ**

- อ่านไฟล์ `.scss` ใกล้เคียงก่อนเขียนใหม่ (เลียนแพทเทิร์นเดิม)
- ใช้ตัวแปร Bootstrap แทน hex ฮาร์ดโค้ด เมื่อทำได้
- แก้เฉพาะไฟล์ที่เกี่ยวกับงานที่ขอ — scope เล็ก
- รัน `npm run build:sass` หรือคอมไพล์ไฟล์เดียวหลังแก้ เพื่อตรวจ error
- อัปเดตตาราง “แผนที่ไฟล์” ใน CONTEXT ถ้าเพิ่มโมดูลใหม่สำคัญ

**ไม่ควรทำ**

- Refactor ทั้ง repo เป็น `@use` / design system ใหม่โดยไม่ได้รับคำสั่ง
- Commit โฟลเดอร์ `dist/` (ถูก ignore)
- สมมติว่ามี React/Vue ใน repo นี้ — **มีแต่ SCSS → CSS**
- ยึด `SKILL.md` หรือ `.agents/skills/scss/*` เป็นข้อบังคับของ repo นี้ทั้งหมด — เอกสารเหล่านั้นเป็น **แนวทางอ้างอิงทั่วไป** โปรเจกต์จริงใช้ Bootstrap 5 + แพทเทิร์นใน `src/sass/`

---

## Dependencies หลัก

| แพ็กเกจ | ใช้ทำอะไร |
|---------|-----------|
| `bootstrap` (dev) | ตัวแปร / mixins SCSS |
| `sass` | คอมไพล์ CLI |
| `gulp`, `gulp-sass`, … | pipeline ทางเลือก |
| `select2` | อ้างอิงสไตล์ใน `select2/` |

---

## Checklist เมื่อเพิ่มสไตล์ใหม่

1. สร้าง `src/sass/<ชื่อฟีเจอร์>.scss` (หรือใส่ใน subfolder ที่เหมาะสม)
2. ใส่หัว import Bootstrap ตามแบบด้านบน (ถ้าต้องใช้ token)
3. เขียน class ให้สอดคล้อง HTML ในระบบหลัก
4. `npm run build:sass` แล้วตรวจ `dist/sass/<ชื่อ>.css`
5. แจ้งทีมระบบหลักให้ `<link>` ไฟล์ CSS ใหม่
6. (ถ้าต้องการ) เพิ่มแถวใน **แผนที่ไฟล์** ด้านบน

---

## คำถามที่ยังไม่ได้กำหนดใน repo (เติมเองได้)

ใส่ข้อมูลต่อไปนี้เมื่อทีมพร้อม — ช่วยให้ AI ทำงานแม่นขึ้น:

- [ ] URL / path จริงที่ deploy `dist/sass/*.css` บน server
- [ ] เวอร์ชัน Bootstrap CSS ที่โหลดในระบบหลัก (CDN หรือ local)
- [ ] ฟอนต์หลัก (เช่น Kanit — ใช้ในบางไฟล์เช่น `risk.scss`)
- [ ] โปรเจกต์ PHP/repo หลักที่ consume CSS นี้ (ลิงก์ Git หรือชื่อโฟลเดอร์)
- [ ] กฎ code review / ใคร approve ก่อน merge

---

## เอกสารที่เกี่ยวข้อง

| ไฟล์ | เนื้อหา |
|------|---------|
| `README.md` | คำอธิบายสั้น |
| `.cursor/skills/sass-js-ts-html/SKILL.md` | แนวทาง SCSS/JS/HTML ทั่วไปสำหรับ Cursor |
| `src/sass/bootstrap_bs5/_variables.scss` | token สีและ theme ของโปรเจกต์ |

---

*อัปเดตล่าสุด: 2026-05 — ปรับแผนที่ไฟล์และ checklist เมื่อโครงสร้างเปลี่ยน*
