# Sass สำหรับสไตล์ของโปรเจกต์

โปรเจกต์นี้เก็บ SCSS สำหรับหน้าและ component ในระบบหลัก เช่น งานครุภัณฑ์ ตารางเวร ฟอร์ม ปุ่ม และ badge โดยใช้ Dart Sass คอมไพล์เป็น CSS แล้วนำไปโหลดในโปรเจกต์ PHP/HTML อีกที ใช้ตัวแปรและ mixins ของ Bootstrap 5.3 เป็นฐาน แต่ไม่ได้ build Bootstrap CSS เต็มชุด

ข้อมูลด้านล่างตรวจจาก [package.json](package.json), [package-lock.json](package-lock.json), [Gulp pipeline](src/cli/gulp.sass.mjs) และไฟล์ใน `src/sass/` ณ วันที่ 5 พฤศจิกายน 2026

## เริ่มใช้งาน

รันคำสั่งจาก root ของ repository โดยต้องติดตั้ง dependencies รวม `devDependencies` เพราะ SCSS อ้างอิง Bootstrap ที่อยู่ในกลุ่มนี้

```bash
npm ci
npm run build
```

ระหว่างแก้สไตล์ ใช้คำสั่ง watch:

```bash
npm run dev
```

`package.json` ยังไม่ได้กำหนด `engines` ของ Node.js โดย dependency `del@8.0.0` ใน lockfile ต้องการ Node.js `>=18` การตรวจคอมไพล์ครั้งนี้ใช้ Node.js `24.21.0` และ Dart Sass `1.89.2`

## โครงสร้างและผลลัพธ์

```text
src/
├── sass/
│   ├── *.scss                  # สไตล์ของแต่ละหน้า/ฟีเจอร์
│   ├── _btn_minimal.scss       # mixins ปุ่ม minimal
│   ├── _btn_sm_minimal.scss    # mixins ปุ่ม minimal ขนาดเล็ก
│   ├── _badge_soft.scss        # mixins badge โทนอ่อน
│   ├── bootstrap_bs5/          # functions และ variables ที่เก็บใน repo
│   ├── shift_schedule/         # สไตล์ตารางเวร
│   └── select2/                # สไตล์ Select2 ที่เก็บใน repo
└── cli/
    ├── gulp.sass.mjs           # pipeline ทางเลือก
    ├── read-log.mjs            # แปลง MariaDB slow log เป็น JSON
    └── read-json.mjs           # อ่านและเรียงข้อมูล slow log
dist/
├── sass/                      # ผลลัพธ์จาก Sass CLI
└── gulp/                      # ผลลัพธ์จาก Gulp
```

- Sass CLI คอมไพล์ไฟล์ `.scss` ที่ไม่ขึ้นต้นด้วย `_` แบบ recursive และเก็บโครงสร้างโฟลเดอร์เดิม ขณะตรวจมี 35 entry points
- ไฟล์ `_*.scss` เป็น partial สำหรับ import/include จึงไม่มี CSS แยกจากการ build แบบ directory
- ไม่มี entry point รวมชื่อ `main.scss` แต่ละหน้าเลือกลิงก์ CSS ที่ต้องการ
- `dist/` และ `node_modules/` อยู่ใน `.gitignore`

| ต้นทาง | ผลลัพธ์จาก Sass CLI |
| --- | --- |
| `src/sass/manage_multiple_assets.scss` | `dist/sass/manage_multiple_assets.css` |
| `src/sass/shift_schedule/assign_scheduleing.scss` | `dist/sass/shift_schedule/assign_scheduleing.css` |
| `src/sass/select2/core.scss` | `dist/sass/select2/core.css` |
| `src/sass/select2/theme/default/layout.scss` | `dist/sass/select2/theme/default/layout.css` |

`layout.scss` ไม่ขึ้นต้นด้วย `_` จึงถูกคอมไพล์แยกด้วย แม้จะถูก import เข้า `select2/core.scss` อยู่แล้ว

## คำสั่งใน package.json

| คำสั่ง | คำสั่งที่เรียกจริง | พฤติกรรม |
| --- | --- | --- |
| `npm run dev` | `sass --watch src/sass:dist/sass --no-source-map --style=expanded` | คอมไพล์และเฝ้าดูการเปลี่ยนแปลง CSS แบบอ่านง่าย ไม่มี source map |
| `npm run build` | `sass src/sass:dist/sass --no-source-map --style=compressed` | คอมไพล์ครั้งเดียวแบบย่อขนาด ไม่มี source map |
| `npm run dev:gulp` | `gulp --gulpfile src/cli/gulp.sass.mjs` | ขอรัน task `default` แต่ไฟล์ปัจจุบันไม่มี task นี้ จึงยังใช้เป็นคำสั่งพัฒนา/watch ไม่ได้ |
| `npm run build:gulp` | `gulp --gulpfile src/cli/gulp.sass.mjs sass` | รัน `sassAll()` คอมไพล์เฉพาะ SCSS ระดับ root ไป `dist/gulp/` แล้วจัดรูปแบบ CSS |
| `npm run logging:read` | `node src/cli/read-log.mjs` | อ่าน `src/cli/mariadb-slow.log` แล้วเขียนทับ `src/cli/mariadb-slow.json` พร้อมแสดงข้อมูล |
| `npm run json:read` | `node src/cli/read-json.mjs` | อ่าน JSON ดังกล่าว เพิ่มค่า `queryTime_sec` ในหน่วยความจำ เรียง `queryTime` จากมากไปน้อย แล้วแสดงผล ไม่มีการเขียนไฟล์ |
| `npm test` | `echo "Error: no test specified" && exit 1` | เป็น placeholder และจบด้วยสถานะล้มเหลว ยังไม่มี test suite |

คำสั่ง `logging:read` และ `json:read` เป็นเครื่องมือจัดการ log ไม่ได้อยู่ในขั้นตอน build Sass

คอมไพล์ไฟล์เดียวเพื่อดูผลได้ เช่น:

```bash
npx sass src/sass/manage_multiple_assets.scss dist/sass/manage_multiple_assets.css --style=expanded --no-source-map
```

## Dependencies ที่ใช้งาน

เวอร์ชันใน `package.json` เป็นช่วงที่อนุญาต ส่วนคอลัมน์ lockfile คือเวอร์ชันที่ล็อกไว้สำหรับ `npm ci`

| Package | package.json | lockfile | การใช้งานที่ตรวจพบ |
| --- | --- | --- | --- |
| `sass` | `^1.89.2` | `1.89.2` | Dart Sass สำหรับ CLI และ compiler ของ `gulp-sass` |
| `gulp` | `^5.0.1` | `5.0.1` | จัดการ task และ stream ใน pipeline |
| `gulp-sass` | `^6.0.1` | `6.0.1` | เชื่อม Dart Sass เข้ากับ Gulp ผ่าน `gulpSass(dartSass)` |
| `gulp-less` | `^5.0.0` | `5.0.0` | ถูกเรียกหลัง Sass ใน `sassAll()` และ `sassProd()` ไม่พบไฟล์ `.less` ใน `src/` |
| `gulp-cssbeautify` | `^3.0.1` | `3.0.1` | จัดรูปแบบ CSS ใน `sassAll()` และ `sassSelect2()` |
| `gulp-clean-css` | `^4.3.0` | `4.3.0` | ย่อ CSS ใน `sassProd()` |
| `gulp-rename` | `^2.1.0` | `2.1.0` | เปลี่ยนชื่อผลลัพธ์ Select2 และ production pipeline |
| `del` | `^8.0.0` | `8.0.0` | ลบ `dist/gulp/` เมื่อโหลด Gulp file |
| `select2` | `^4.1.0-rc.0` | `4.1.0-rc.0` | มี dependency นี้ แต่ SCSS ของ Select2 ใน pipeline มาจาก `src/sass/select2/` |
| `@fancyapps/ui` | `^6.1.6` | `6.1.6` | ประกาศไว้ แต่ไม่พบการ import ใน SCSS หรือ CLI ของ repo นี้ |
| `bootstrap` (devDependency) | `^5.3.8` | `5.3.8` | ใช้ SCSS variables และ mixins จาก `node_modules/bootstrap/scss/` |

## Pipeline และฟังก์ชันใน Gulp

อ้างอิง [src/cli/gulp.sass.mjs](src/cli/gulp.sass.mjs) โดย path ในไฟล์อิงจาก `src/cli/` เมื่อเรียกผ่าน Gulp CLI ตาม script ของโปรเจกต์

| ฟังก์ชัน/task | ขั้นตอนและผลลัพธ์ |
| --- | --- |
| `sassAll()` / task `sass` | `gulp.src('../sass/*.scss')` → `sass()` → `less()` → `gulpCssbeautify()` → `gulp.dest('../../dist/gulp')` ชื่อไฟล์คงตามต้นทาง |
| `sassSelect2()` / task `select2` | อ่าน `../sass/select2/core.scss` → `sass()` → `gulpCssbeautify()` → `rename('select2-custom.css')` → `dist/gulp/` |
| `sassProd()` | อ่าน SCSS ระดับ root → `sass()` → `less()` → `cleanCSS()` → ตั้งชื่อทุกไฟล์เป็น `main.min.css` → `dist/gulp/` |
| `sassWatch()` | เรียก `gulp.watch(paths.styles.src, sassDev)` แต่ไม่มีนิยาม `sassDev` จึงใช้งานไม่ได้ในสถานะปัจจุบัน |
| task `all` | `gulp.series('sass', 'select2')` รันสอง task ตามลำดับ |

แต่ละ pipeline ที่คอมไพล์ Sass ผูก error handler ด้วย `sass().on('error', sass.logError)`

ข้อจำกัดของ Gulp ที่ต้องทราบก่อนใช้:

- มี `export const clean = await deleteAsync(['../../dist/gulp'], {force: true})` ที่ระดับบนสุด การโหลด Gulp file จึงลบ `dist/gulp/` ก่อนเริ่ม task รวมถึงคำสั่งที่มาจบด้วยข้อผิดพลาดเรื่อง task; `clean` เป็นผลลัพธ์ของการลบ ไม่ใช่ฟังก์ชัน task สำหรับเรียกภายหลัง
- `sassAll()`/`sassProd()` ใช้ glob `*.scss` เฉพาะระดับ root จึงไม่ครอบคลุม `shift_schedule/` ขณะตรวจมี root entry points 27 ไฟล์
- `build:gulp` เรียก `sassAll()` จึงจัดรูปแบบ CSS และไม่ได้เรียก `sassProd()` แม้ชื่อคำสั่งจะเป็น build
- `sassProd()` เปลี่ยนชื่อผลลัพธ์ทุกไฟล์เป็นชื่อเดียวกัน ทำให้เขียนทับกัน ไม่ใช่การรวม CSS หลายไฟล์
- `sassWatch()` มีทั้งปัญหา callback ที่ไม่มีนิยามและ glob ที่ไม่ครอบคลุมโฟลเดอร์ย่อย; ไม่มีการผูกเป็น `default` task

หากต้องการ build/watch ตามโครงสร้าง SCSS ทั้งโปรเจกต์ ให้ใช้ `build` และ `dev` สำหรับ Gulp มี task เฉพาะที่เรียกได้ดังนี้ โดยทุกคำสั่งจะล้าง `dist/gulp/` ก่อนตามพฤติกรรมข้างต้น:

```bash
npx gulp --gulpfile src/cli/gulp.sass.mjs select2
npx gulp --gulpfile src/cli/gulp.sass.mjs all
```

## การโหลด Sass และ Bootstrap

หัวไฟล์ SCSS ระดับ root ที่ใช้ Bootstrap ส่วนใหญ่มีรูปแบบนี้:

```scss
@use 'sass:color';
@import "bootstrap_bs5/functions";
@import "../../node_modules/bootstrap/scss/variables";
@import "../../node_modules/bootstrap/scss/mixins";
```

- `@use 'sass:color'` เปิดใช้ฟังก์ชันที่เรียกผ่าน namespace `color.*` ไม่ได้แปลว่าแต่ละไฟล์จะเรียกฟังก์ชันเหล่านี้เสมอไป
- `bootstrap_bs5/functions` คือ [ไฟล์ functions ใน repo](src/sass/bootstrap_bs5/_functions.scss) ไม่ใช่ `node_modules/bootstrap/scss/_functions.scss`
- ตัวแปรมีสองเส้นทาง: บางไฟล์ import จาก `node_modules/bootstrap/scss/variables` ส่วนบางไฟล์ใช้ [bootstrap_bs5/variables](src/sass/bootstrap_bs5/_variables.scss) ซึ่ง import `_variables-dark.scss` ต่อ การแก้ variables ภายใน repo จึงไม่ได้มีผลกับทุก entry point
- ไฟล์ใน `shift_schedule/` ใช้ relative path เพิ่มอีกหนึ่งระดับ ส่วน `select2/core.scss` ประกอบจาก partial และ theme ภายในโฟลเดอร์ของตัวเอง
- ตรวจพบทั้ง `@use` และ `@import` แต่ไม่พบ `@forward` ใน `src/sass/`

## ฟังก์ชัน Sass ที่เรียกใช้

### ฟังก์ชันที่เรียกในสไตล์หน้าและ component

ตารางนี้แสดงการเรียกในไฟล์ฟีเจอร์และ partial ของโปรเจกต์ โดยแยกจาก helper ภายใน `bootstrap_bs5/`

| ฟังก์ชัน | หน้าที่ | ตัวอย่างไฟล์ |
| --- | --- | --- |
| `color.scale()` | ปรับ lightness/saturation ตามสัดส่วน | [risk.scss](src/sass/risk.scss), [gtw_datatable.scss](src/sass/gtw_datatable.scss) |
| `color.change()` | กำหนดค่า channel ของสี เช่น lightness, saturation, alpha | [attach_files.scss](src/sass/attach_files.scss), [compensations_manager.scss](src/sass/compensations_manager.scss) |
| `color.adjust()` | เพิ่ม/ลดค่า channel ของสีตามจำนวนที่ระบุ | [asset_running_fsn.scss](src/sass/asset_running_fsn.scss) |
| `map-get()` | อ่านค่าจาก map เช่น `$grays` และ `$theme-colors` | [appeal_approval.scss](src/sass/appeal_approval.scss), [custom.scss](src/sass/custom.scss) |
| `rgb()`, `rgba()` | สร้างสีหรือกำหนด alpha; Sass สามารถประเมินได้เมื่อ argument เป็นค่าที่ทราบตอน compile | [manage_multiple_assets.scss](src/sass/manage_multiple_assets.scss), [_btn_minimal.scss](src/sass/_btn_minimal.scss) |
| `tint-color()` | helper ของ Bootstrap ใน repo ผสมสีกับสีขาวด้วย `color.mix()` | [badge_style.scss](src/sass/badge_style.scss), [_btn_minimal.scss](src/sass/_btn_minimal.scss) |
| `shade-color()` | helper ของ Bootstrap ใน repo ผสมสีกับสีดำด้วย `color.mix()` | [manage_multiple_assets.scss](src/sass/manage_multiple_assets.scss), [_badge_soft.scss](src/sass/_badge_soft.scss) |
| `escape-svg()` | helper สำหรับแทนที่อักขระใน SVG data URI | [shift_schedule/assign_scheduleing.scss](src/sass/shift_schedule/assign_scheduleing.scss) |

### Helper functions ที่ประกาศใน bootstrap_bs5

[bootstrap_bs5/_functions.scss](src/sass/bootstrap_bs5/_functions.scss) ประกาศ `@function` 19 ตัวดังนี้ รายการประกาศนี้ไม่เท่ากับรายการที่ถูกเรียกโดยตรงจากไฟล์ฟีเจอร์

| กลุ่ม | ฟังก์ชันที่ประกาศ | หน้าที่ |
| --- | --- | --- |
| สี | `to-rgb()`, `rgba-css-var()` | แปลงสีเป็น RGB channels / สร้างค่าสีจาก CSS variables |
| สีอ่อน/เข้ม | `tint-color()`, `shade-color()`, `shift-color()` | ผสมสีขาว/ดำ; `shift-color()` เลือกตามเครื่องหมายของ weight |
| Contrast | `color-contrast()`, `contrast-ratio()`, `luminance()`, `opaque()` | เลือกสี foreground และคำนวณ contrast โดยรองรับ alpha |
| Map/list | `map-loop()`, `varify()`, `negativify-map()`, `map-get-multiple()`, `map-merge-multiple()` | วน map, สร้าง CSS variable references, สร้างค่าติดลบ, เลือก/รวมค่า |
| String/SVG | `str-replace()`, `escape-svg()` | แทนที่ข้อความและ escape SVG data URI |
| ตัวเลข/หน่วย | `add()`, `subtract()`, `divide()` | คำนวณค่าตัวเลขและค่าที่มีหน่วย รวมถึงสร้าง `calc()` ในบางกรณี |

ใน source ภายใน repo พบว่า variables เรียก `to-rgb()`, `color-contrast()`, `shift-color()`, `add()` และ `subtract()` ส่วน `str-replace()`, `contrast-ratio()`, `luminance()`, `opaque()` และ `divide()` ถูกเรียกต่อกันภายใน helper อีกที สำหรับ `rgba-css-var()`, `map-loop()`, `varify()`, `negativify-map()`, `map-get-multiple()` และ `map-merge-multiple()` พบเพียงนิยาม ไม่พบจุดเรียกใน source ของ repo โดยไม่นับ source ใน `node_modules/`

ภายใน helper และ variables ยังพบ built-ins เหล่านี้:

| กลุ่ม | ฟังก์ชันที่มีการเรียกใน source |
| --- | --- |
| Color module | `color.mix()`, `color.red()`, `color.green()`, `color.blue()` |
| สีแบบ global | `mix()`, `rgba()`, `opacity()` |
| Map/list | `map-get()`, `map-merge()`, `map-has-key()`, `map-values()`, `append()`, `length()`, `nth()`, `index()` |
| String | `str-index()`, `str-slice()`, `str-length()`, `quote()`, `unquote()` |
| ตัวเลข/ชนิดข้อมูล | `abs()`, `unit()`, `comparable()`, `type-of()` |
| เงื่อนไข/เรียกฟังก์ชัน | `if()`, `get-function()`, `call()` โดย `map-loop()` ใช้สองตัวหลังสำหรับเรียก helper จากชื่อ |

### CSS functions ที่พบร่วมกัน

`var()`, `calc()`, `clamp()`, `minmax()`, `repeat()`, `linear-gradient()`, `translate()`/`translateX()`/`translateY()`, `rotate()`, `scale()`, `cubic-bezier()`, `url()`, `attr()` และ `counter()` เป็นรูปแบบฟังก์ชันของ CSS ไม่ใช่ `@function` ที่โปรเจกต์ประกาศเอง บางนิพจน์ เช่น `calc()` อาจถูก Sass ย่อก่อนส่งออก

ตัวอย่าง `scale()` ใน `transform` ต่างจาก `color.scale()` ซึ่งเป็นฟังก์ชันของ Sass ส่วน `:not()`, `:has()` และ `:nth-child()` เป็น pseudo-class selectors

## Mixins ที่ใช้งาน

Mixin สร้างชุด declarations/selectors ผ่าน `@include` ต่างจาก `@function` ที่คืนค่าผ่าน `@return`

| Mixin | นิยาม | การเรียก/ผลลัพธ์ |
| --- | --- | --- |
| `btn-minimal-variant($color, $name)` / `btn-minimal-variants($colors)` | [_btn_minimal.scss](src/sass/_btn_minimal.scss) | ตัว plural วน color map แล้วเรียกตัว singular เพื่อสร้าง `.btn-minimal-*`; ใช้ใน `button_minimal.scss`, `appeal_approval.scss`, `modal_edit_fsn.scss`, `manage_multiple_assets.scss` |
| `btn-small-minimal-variant($color, $name)` / `btn-small-minimal-variants($colors)` | [_btn_sm_minimal.scss](src/sass/_btn_sm_minimal.scss) | สร้าง `.btn-sm-minimal-*`; ใช้ใน `button_small_minimal.scss` และ `shift_schedule/assign_scheduleing.scss` |
| `badge-soft-variant($color, $name)` / `badge-soft-variants($colors)` | [_badge_soft.scss](src/sass/_badge_soft.scss) | สร้าง `.badge-soft` และ variant ตาม map; เรียกผ่านตัว plural ใน `manage_multiple_assets.scss` |
| `badge-soft-variant($color, $name)` | [badge_style.scss](src/sass/badge_style.scss) | มีนิยามชื่อเดียวกันอีกชุดใน entry point นี้ และเรียกภายใน `@each` ของ `$theme-colors` |
| `btn-gtw($color, $name)` | [custom.scss](src/sass/custom.scss) | เรียกใน `@each` เพื่อสร้างปุ่มตาม color map |
| `btn-outline-soft($color, $name)` | [risk.scss](src/sass/risk.scss), [schedule_signature_setting.scss](src/sass/shift_schedule/schedule_signature_setting.scss) | แต่ละไฟล์มีนิยามและเรียกของตัวเอง |
| `btn-colors($color, $name)` | [show_scheduling.scss](src/sass/shift_schedule/show_scheduling.scss) | เรียกใน `@each` เพื่อสร้างปุ่มตาม color map |
| `media-breakpoint-down($name)` | `node_modules/bootstrap/scss/mixins/_breakpoints.scss` | เรียกด้วย `lg` และ `sm` ใน `manage_multiple_assets.scss` เพื่อสร้าง media queries |
| `_assert-ascending()` / `_assert-starts-at-zero()` | [bootstrap_bs5/_functions.scss](src/sass/bootstrap_bs5/_functions.scss) | เรียกจาก variables เพื่อตรวจลำดับและค่าเริ่มต้นของ breakpoint map |

`show_scheduling.scss` ยังมีนิยาม `btn-outline-soft()` อีกชุด แต่บรรทัด `@include` ของชุดนี้ถูก comment ไว้ การประกาศ mixin เพียงอย่างเดียวจึงไม่สร้าง CSS ของชุดนั้น

ตัวอย่างสร้างปุ่มและ badge เฉพาะสีที่ต้องการ สำหรับไฟล์ระดับ `src/sass/`:

```scss
@use 'sass:color';
@import "bootstrap_bs5/functions";
@import "../../node_modules/bootstrap/scss/variables";
@import "btn_minimal";
@import "badge_soft";

@include btn-minimal-variants(("blue": $primary, "green": $success));
@include badge-soft-variants(("yellow": $warning, "red": $danger));
```

ตัวอย่างใช้งาน CSS ที่ได้ในระบบหลัก ซึ่งต้องโหลด Bootstrap CSS ของระบบหลักไว้ด้วย:

```html
<link rel="stylesheet" href="/path/to/dist/sass/manage_multiple_assets.css">
<button type="button" class="btn btn-minimal-blue">บันทึก</button>
<span class="badge badge-soft badge-soft-yellow">รอดำเนินการ</span>
```

## การตรวจสอบและข้อจำกัดปัจจุบัน

- ตรวจคอมไพล์ entry points 35 ไฟล์ด้วย `sass.compile()` แบบ `compressed` ในหน่วยความจำผ่านทั้งหมดเมื่อวันที่ 5 พฤศจิกายน 2026 โดยไม่เขียนทับ `dist/` ผลนี้ยืนยันการคอมไพล์ SCSS แต่ไม่ได้ยืนยัน pipeline Gulp หรือหน้าตาใน browser
- Compiler `1.89.2` ยังแสดง deprecation warnings กลุ่ม `import`, `global-builtin` และ `color-functions` จาก source และ dependencies แม้คอมไพล์ผ่าน
- ยังไม่มี automated tests ใน `npm test` และยังไม่ได้แก้ข้อจำกัด Gulp ที่บันทึกไว้ข้างต้น

ดูบริบทของโมดูลและแผนที่ไฟล์เพิ่มเติมใน [CONTEXT.md](CONTEXT.md) โดยรายละเอียดคำสั่งใน README นี้อ้างอิง implementation ปัจจุบัน
