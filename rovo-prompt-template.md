# Rovo Prompt Template — ดึง Spec จาก Confluence

ใช้ตอนขั้นตอน manual ก่อนเรียกสกิล `obsidian-spec-sync` (ดู `README-obsidian-spec-sync.md`
สำหรับ context เต็ม) — วางใน Rovo Chat (หรือยิงผ่าน MCP) แล้วแก้ค่าตาม service ที่ต้องการ:
`{SERVICE_NAME}` → `confluence_space` และ `{KEYWORD_LIST}` → `confluence_keywords` ตรงจาก
entry ของ service นั้นใน `docs/.service-map.json` (จำนวน keyword ไม่จำกัด ใส่กี่ตัวก็ได้
ตามที่มีจริงในไฟล์ ไม่ต้องพอดี 3 ตัว)

ยิงเฉพาะตอน Confluence page เปลี่ยนเท่านั้น — ถ้า code เปลี่ยนอย่างเดียว ข้าม prompt นี้ไปเลย
แล้วเรียกสกิล `obsidian-spec-sync` แบบ "code-only refresh" แทน

```
รวบรวมและสรุปข้อมูลทั้งหมดเกี่ยวกับ service "{SERVICE_NAME}"
จาก Confluence space "{CONFLUENCE_SPACE}"

ค้นหา page ที่เกี่ยวข้องโดยใช้ keyword: {KEYWORD_LIST}

สำหรับแต่ละ page ที่พบ ให้:
1. ระบุ title, URL, และวันที่แก้ไขล่าสุด (last modified)
2. สรุปเนื้อหาสำคัญแบบไม่ตัดรายละเอียดที่จำเป็นต่อการ implement
   (เช่น API endpoint, request/response schema, business rule,
   edge case, error handling)
3. ถ้ามีหลาย page ที่เนื้อหาซ้ำหรือขัดแย้งกัน ให้ flag ไว้ชัดเจน
   พร้อมระบุว่า page ไหนดูเหมือนเป็นเวอร์ชันล่าสุด/authoritative

จัดกลุ่มผลลัพธ์ทั้งหมดเป็นโครงสร้างนี้:

## Overview
[สรุปภาพรวมว่า service นี้ทำอะไร]

## API Specification
[รวม endpoint ทั้งหมดที่เจอจากทุก page ไม่ซ้ำกัน]

## Business Rules
[กฎทางธุรกิจที่เกี่ยวข้อง]

## Data Model
[schema/entity ที่เกี่ยวข้อง]

## Source Pages
[list ของทุก page ที่ใช้อ้างอิง พร้อม URL + last modified date]

## Conflicts/Ambiguity Found
[ถ้ามี page ที่ขัดแย้งกัน ระบุตรงนี้]

Output เป็น Markdown format พร้อม YAML frontmatter ด้านบนสุด:
---
service: {SERVICE_NAME}
confluence_space: {CONFLUENCE_SPACE}
source_pages: [list of URLs]
last_synced: [วันที่วันนี้]
---
```

## ตัวอย่างการแทนค่า (payment-service)

จาก `.service-map.json` ตัวอย่าง:
```json
"payment-service": {
  "confluence_space": "PAYM",
  "confluence_keywords": ["payment", "payment-service", "payment api"]
}
```

แทนค่าลง template ด้านบนได้ตรงๆ:
- `{SERVICE_NAME}` → `payment-service`
- `{CONFLUENCE_SPACE}` → `PAYM`
- `{KEYWORD_LIST}` → `payment, payment-service, payment api`

ส่วนที่เหลือของ prompt (คำสั่ง 1-3, โครงสร้าง output, frontmatter) คงเดิมทุกตัวอักษร ไม่ต้อง
แก้อะไรเพิ่ม

## หลังได้ output จาก Rovo

- ถ้า Rovo หา page ไม่เจอเลย หรือ summary ว่าง/ดูไม่น่าเชื่อถือ **อย่าส่งต่อให้สกิล** — ปรับ
  keyword ใน `docs/.service-map.json` แล้วยิงใหม่ก่อน
- ถ้า output ดูสมเหตุสมผล → copy markdown ทั้งก้อนไปแนบให้ Copilot ตอนเรียกสกิล
  `obsidian-spec-sync` (โหมด "fresh spec")
