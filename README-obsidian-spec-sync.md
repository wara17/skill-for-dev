# สกิล Obsidian Spec Sync

**ชื่อสกิล:** `obsidian-spec-sync`

## สกิลนี้ทำอะไร

รับ spec summary ของ service หนึ่งตัว (markdown ที่มาจาก Confluence/Rovo หรือคนเขียนเองก็ได้)
แล้ว:
1. เทียบกับโค้ดจริงใน repo — หา match / mismatch / ส่วนที่ไม่มีใน spec
2. อ่านไฟล์ test แล้วสกัดเป็น test case พร้อม label ว่ามาจาก unit test จริงหรือ AI คาดเดา
3. สร้าง Mermaid diagram ของ flow
4. เขียนผลลัพธ์ลง `docs/{service}.md` + `docs/{service}/cases.json` ซึ่งเป็น Obsidian vault
   พร้อมอัพเดท `docs/index.md` และ `docs/.sync-state.json`
5. **หยุดรอ review — ไม่ commit/push เอง**

สกิลนี้**ไม่คุยกับ Confluence หรือ Rovo โดยตรง** — เป็นสกิลฝั่ง Copilot/Claude Code ที่รับ
input เป็น markdown ที่มีอยู่แล้วเท่านั้น ส่วนขั้นตอนดึงข้อมูลจาก Confluence ยังเป็น manual
ตามคู่มือด้านล่าง

สกิลนี้**ไม่แตะ git เลยในเวอร์ชันนี้** — ไม่รัน `git log`/`commit`/`push` และไม่ใช้ commit
hash ใน field ไหนเลย tracking ทั้งหมดใช้วันที่ (`code_verified_date`,
`last_extracted_date`, timestamp ใน `.sync-state.json`) แทน ถ้าจะเพิ่ม commit-based
tracking กลับมาทีหลังค่อยแก้ `obsidian-spec-sync/SKILL.md` เพิ่มเอง

## ที่มา

สกิลนี้แยกมาจาก pipeline "Obsidian Knowledge Base" เดิมที่รวม Rovo (อ่าน Confluence) กับ
Copilot (verify โค้ด + เขียนไฟล์) ไว้ในคู่มือเดียว — แยกส่วน Copilot ออกมาเป็นสกิลที่ trigger
ได้เอง ส่วน Rovo prompt ยังคงเป็น manual เพราะ Rovo เป็นคนละ tool ไม่ได้อ่านไฟล์ `SKILL.md`
แบบเดียวกับ Copilot/Claude Code

## Setup ครั้งแรก (ทำครั้งเดียวต่อ repo)

สร้างไฟล์ `docs/.service-map.json`:

```json
{
  "payment-service": {
    "confluence_space": "PAYM",
    "confluence_keywords": ["payment", "payment-service", "payment api"],
    "repo_path": "src/services/payment/",
    "test_path": "tests/payment/",
    "owner": "team-payments"
  }
}
```

เพิ่ม service อื่นต่อในไฟล์เดียวกันเรื่อยๆ ตามต้องการ แล้วเปิด Obsidian → "Open folder as
vault" → เลือก `your-repo/docs/`

## ขั้นตอน manual (ก่อนเรียกสกิล) — เช็ค diff แล้วดึง spec จาก Confluence

### 1. เช็คว่าต้อง sync ไหม

- Confluence page ล่าสุด `lastModified` เปลี่ยนจาก `docs/.sync-state.json` หรือไม่
- ไฟล์ใน `repo_path` มีการแก้ไขหลังวันที่ sync ล่าสุดใน `.sync-state.json` หรือไม่ (ดูวันที่
  แก้ไขไฟล์ล่าสุด/`git log -1` ก็ได้ตามสะดวก — ขั้นตอนนี้เป็นการเช็คของคนเอง สกิลเองไม่ยุ่ง
  กับ git)

ถ้าไม่มีอะไรเปลี่ยนเลย ข้ามได้ ไม่ต้องเสีย Rovo credit และไม่ต้องเรียกสกิลนี้

- **Confluence เปลี่ยน** → ทำข้อ 2 (ยิง Rovo ใหม่) แล้วค่อยเรียกสกิล `obsidian-spec-sync`
  แบบ "fresh spec"
- **Code เปลี่ยนอย่างเดียว** → ข้ามข้อ 2 ไปเลย เรียกสกิลนี้ตรงๆ แบบ "code-only refresh"
  (สกิลจะไปอ่าน `docs/{service}.md` เดิมเอง ไม่ต้องเตรียม spec ใหม่)

### 2. Prompt สำหรับ Rovo (ยิงเฉพาะตอน Confluence เปลี่ยน)

Prompt เต็มอยู่ที่ `rovo-prompt-template.md` — เปิด Rovo Chat (หรือยิงผ่าน MCP) แล้ววาง prompt
จากไฟล์นั้น ใส่ค่า service/space/keyword ตามที่ต้องการ

**ผลที่ได้**: markdown 1 ก้อน (spec summary) — เก็บไว้แล้วแนบให้ Copilot ในขั้นตอนถัดไป

ถ้า Rovo หา page ไม่เจอเลย หรือ keyword คลุมเครือจน summary ว่าง/ดูไม่น่าเชื่อถือ **อย่าส่งต่อ
ให้สกิลตรงๆ** — ปรับ keyword ใน `.service-map.json` แล้วยิงใหม่ก่อน spec ที่ว่างเปล่าจะทำให้
Step 2 (compare) ของสกิลรายงานผลผิดทั้งหมด

### 3. เรียกสกิล `obsidian-spec-sync`

เปิด Copilot Chat / Claude Code ใน repo เดิม แล้ว:
- **fresh spec**: แนบ markdown จาก Rovo ต่อท้าย prompt เรียกสกิล เช่น "verify spec นี้กับ
  src/services/payment/ แล้ว sync เข้า Obsidian" พร้อม paste markdown
- **code-only refresh**: บอกตรงๆ "code เปลี่ยน ไม่มี spec ใหม่ ตรวจ payment-service กับโค้ด
  ปัจจุบันอีกที" — สกิลจะไปอ่าน `docs/payment-service.md` เดิมเอง

รายละเอียดขั้นตอนภายในสกิล ดูที่ `obsidian-spec-sync/SKILL.md`

### 4. Review แล้ว commit เอง

สกิลจะหยุดหลังเขียนไฟล์เสร็จ ไม่ commit ให้อัตโนมัติ:
1. เปิดไฟล์ `.md` ใน Obsidian ดูว่า render ถูก (mermaid ขึ้นภาพ, frontmatter ไม่พัง)
2. เปิด `cases.json` เช็คว่าทุก case มี field `source` ระบุครบ
3. เช็คว่า section ที่คนเขียนเพิ่มเองด้วยมือ (ถ้ามี) ยังอยู่ ไม่ถูกทับ
4. `git add docs/` → `git commit -m "docs: update {service} knowledge base"` → `git push`

## Checklist สรุปย่อ

- [ ] เช็ค diff ก่อน ไม่ต้อง sync ถ้าไม่มีอะไรเปลี่ยน
- [ ] Rovo prompt ระบุ service + space + keyword ให้ตรง ป้องกันได้ page ไม่เกี่ยว
- [ ] ถ้า Rovo คืน spec ว่าง/ไม่น่าเชื่อถือ อย่าส่งต่อให้สกิล ปรับ keyword แล้วยิงใหม่ก่อน
- [ ] แนบ output จาก Rovo ให้สกิลครบ ไม่ตัดทอน (เฉพาะกรณี fresh spec)
- [ ] เช็ค `cases.json` ทุก case มี `source` label ชัดเจน (unit_test vs ai_predicted)
- [ ] เปิดไฟล์ดูใน Obsidian ก่อน commit จริง
- [ ] เช็คว่า index.md + .sync-state.json ถูกอัพเดทแล้วก่อน commit
- [ ] commit/push เอง — สกิลจะไม่ทำให้อัตโนมัติ

## ข้อจำกัดที่ควรรู้

- สกิลนี้ไม่ได้เชื่อมกับ Confluence/Rovo เอง ต้องอาศัยคนหรือ automation แยกต่างหากคอย
  เช็ค diff และยิง prompt Rovo ตามขั้นตอน manual ด้านบน — ถ้าลืมทำขั้นตอนนี้เป็นระยะ
  Obsidian vault จะไม่รู้ตัวว่า Confluence เปลี่ยนไปแล้ว
- ความถูกต้องของผลลัพธ์ขึ้นกับคุณภาพของ spec ที่ Rovo สรุปมา — ถ้า spec เดิมสรุปมาไม่ครบ
  หรือเข้าใจผิด สกิลนี้จะ verify กับ spec ที่ผิดนั้นตรงๆ ไม่มีทางรู้ได้เอง
- `.service-map.json` ต้องแม่นและอัพเดทตามจริง — `repo_path`/`test_path` ผิดจะทำให้สกิล
  สแกนโค้ด/เทสต์คนละตัวโดยไม่รู้ตัว
- ตั้งใจไม่ auto commit/push เพื่อให้มี checkpoint ให้คนตรวจก่อนข้อมูลเข้า version control —
  ถ้าต้องการ automate ต่อ (เช่น CI cron ที่ commit เองได้) ต้องเพิ่ม gate ตรวจสอบเอง
  แยกต่างหาก ไม่ใช่ปิด guardrail นี้ทิ้ง
