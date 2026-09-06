# skill-for-dev

รวมสกิลสำหรับ pipeline การ implement โค้ด ใช้กับ AI coding agent (Copilot / Claude Code
หรือเครื่องมืออื่นที่รองรับไฟล์ SKILL.md แบบเดียวกัน)

## สกิลในชุดนี้

| สกิล | ภาษา | หน้าที่ |
|---|---|---|
| `dev-workflow` | English | Pipeline หลักสำหรับ implement/fix/change โค้ดตาม requirement — ครอบคลุมตั้งแต่เข้าใจ requirement, gap check, วางแผน, implement, test, ไปจนถึงสรุปงาน |
| `dev-workflow-th` | ภาษาไทย | เนื้อหาเดียวกับ `dev-workflow` ทุกประการ แปลเป็นไทยทั้งฉบับ |
| `requirement-version-resolver` | English | ตัวช่วยแกะ requirement จาก Jira/Confluence ที่ใช้สี/ขีดฆ่าบอก version ให้กลายเป็นข้อความสะอาดก่อนส่งต่อ |

## ความสัมพันธ์ระหว่างสกิล

`dev-workflow` (หรือ `dev-workflow-th`) เป็นตัวหลักที่ orchestrate ทั้ง pipeline:

```
Step 1  Requirement + Gap Check + สกัด AC
        └─ ถ้า source เป็น Jira/Confluence → เรียก requirement-version-resolver ก่อน
Step 2+3  Explore + Plan
        └─ อ่าน docs/codebase/CODEBASE.md (มาจากสกิลตระกูล codebase-summary-*
           ซึ่งไม่ได้อยู่ใน repo นี้ — ต้องติดตั้งแยกต่างหากสำหรับ stack ที่ใช้)
Step 4  Implement
Step 5  Test
        └─ อ่านคำสั่งรันเทสต์จาก CODEBASE.md's Testing section โดยตรง
Step 6  Summary
```

พูดง่าย ๆ คือ `dev-workflow` เป็นสกิล **ผู้ใช้บริการ** ส่วน `requirement-version-resolver`
และสกิลตระกูล `codebase-summary-*` เป็นสกิล **ผู้ให้บริการ** ที่ `dev-workflow` เรียกใช้เป็น
ช่วง ๆ ตามเงื่อนไข ไม่ใช่ทุกครั้ง

## เลือกใช้ `dev-workflow` หรือ `dev-workflow-th` ยังไง

เนื้อหาและ logic เหมือนกันทุกประการ ต่างกันแค่ภาษาที่ใช้สื่อสาร — เลือกตัวที่ตรงกับภาษาที่
ทีม/agent ใช้คุยกันสะดวกกว่า ไม่จำเป็นต้องติดตั้งพร้อมกันทั้งสองตัวในบัญชีเดียว เพราะ
trigger phrase ในสองไฟล์คล้ายกันมาก (ต่างแค่ตัวอย่างภาษา) การมีทั้งคู่เพิ่มความเสี่ยงที่ agent
จะเลือกผิดตัวโดยไม่มีประโยชน์เพิ่มจริง

## วิธีติดตั้ง

แต่ละสกิลถูกเก็บเป็นไฟล์ `.skill` (ไฟล์ zip ที่ข้างในมีโฟลเดอร์ชื่อสกิลกับ `SKILL.md`) ไม่ใช่
โฟลเดอร์แตกอยู่ตรง ๆ ใน repo — ต้องแตกไฟล์เองก่อนใช้:

1. ดาวน์โหลด/copy ไฟล์ `.skill` ของสกิลที่ต้องการ (เช่น `dev-workflow.skill`)
2. แตกไฟล์ (มันคือ zip ธรรมดา ใช้ `unzip dev-workflow.skill` หรือโปรแกรมแตกไฟล์ทั่วไปก็ได้)
   จะได้โฟลเดอร์ `dev-workflow/SKILL.md` ออกมา
3. เอาโฟลเดอร์ที่แตกแล้วไปวางในตำแหน่งที่เครื่องมือ AI agent ของคุณอ่านสกิลอยู่ (ตำแหน่ง
   ต่างกันไปตามเครื่องมือ — เช่น `.claude/skills/`, หรือช่องทาง import ของ Copilot)
4. ตรวจว่าเครื่องมือนั้น pick up ไฟล์ `SKILL.md` และ frontmatter (`name`, `description`)
   ได้ถูกต้อง
5. ทดสอบด้วยการพิมพ์คำสั่งที่ตรงกับ trigger phrase ใน `description` ของแต่ละสกิล (เช่น
   พิมพ์ requirement ตรง ๆ ในแชท หรือเรียกชื่อสกิลตรง ๆ ถ้าเครื่องมือรองรับ)

## ข้อควรระวังก่อนใช้จริง

- **`dev-workflow` ต้องพึ่ง `docs/codebase/CODEBASE.md` มีอยู่ก่อน** — ถ้ายังไม่มีไฟล์นี้
  ในโปรเจกต์ Gap Check และ Explore+Plan จะไม่มีอะไรให้อ้างอิง ต้องติดตั้งสกิลตระกูล
  `codebase-summary-*` ที่ตรงกับ stack ของโปรเจกต์นั้นแยกต่างหาก (ไม่รวมอยู่ใน repo นี้)
- **`requirement-version-resolver` ต้องมี MCP tool ต่อกับ Jira/Confluence จริง** ถึงจะ
  ทำงานได้ — ถ้าไม่มีการเชื่อมต่อ สกิลนี้ทำอะไรไม่ได้เลย
- **`dev-workflow` เขียนไฟล์ลง `.task/{slug}-ac.md` ทุกครั้งที่รัน** และไม่ลบไฟล์นี้เอง —
  เป็นพฤติกรรมตั้งใจ (เก็บไว้เป็นหลักฐาน pass/fail ของ AC) ไม่ใช่บั๊ก ต้องลบเองเมื่อ
  ตรวจสอบเสร็จแล้ว

## ไฟล์อื่นใน repo นี้

- `codebase-summary-kt-springboot.skill` — สกิลสำหรับสแกน repo Kotlin/Spring Boot เพื่อสร้าง
  `docs/codebase/CODEBASE.md` ที่ `dev-workflow` ใช้ต่อใน Step 2+3 และ Step 5 (อ่านคำสั่ง
  test) — ควรติดตั้งคู่กับ `dev-workflow`/`dev-workflow-th` เสมอถ้าโปรเจกต์เป็น
  Kotlin/Spring Boot เพราะ `dev-workflow` พึ่งพา `CODEBASE.md` โดยตรง
- `README-codebase-summary-kt-springboot.md` — คู่มือฉบับเต็มของสกิลด้านบน อธิบาย mode
  Generate/Update, การเชื่อมกับ Copilot, และกลไกดูแลตัวเองไม่ให้เก่า
