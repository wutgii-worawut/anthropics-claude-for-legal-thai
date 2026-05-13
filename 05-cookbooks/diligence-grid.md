---
title: Diligence Grid
parent: 05 Cookbooks
nav_order: 1
---

# Diligence Grid — M&A Due Diligence แบบตาราง

> "Every cell is a lead, not a finding."
> — `managed-agent-cookbooks/diligence-grid/README.md`

## จุดประสงค์

Diligence Grid คือ cookbook สำหรับงาน **การตรวจสอบความถูกต้องในการควบรวมกิจการ (M&A due diligence)** — งานที่ในสำนักงานกฎหมายดั้งเดิม ใช้ทีมทนาย junior นั่งอ่านสัญญาเป็นพัน ๆ ฉบับเพื่อกรอกตาราง

cookbook นี้ทำงานใน 2 โหมด:

### โหมด `watch`

เฝ้า **virtual data room (VDR)** สำหรับเอกสารใหม่ที่ upload เข้ามาตั้งแต่ cut-off timestamp ล่าสุด:

- จัดประเภทเอกสารแต่ละชิ้นตาม **request-list categories** (รายการประเภทเอกสารที่ทีม diligence ขอ)
- ทำเครื่องหมายเอกสารที่อยู่ในหมวด **high-priority** (Material Contracts, Litigation, IP)
- ผลิตรายงาน `./out/vdr-update-<date>.md`

**ไม่อ่านเนื้อหาเต็มของเอกสารในโหมดนี้** — อ่านแค่ metadata และ first-page preview เพราะการอ่านเนื้อหาเต็มคืองานของโหมด `grid` หรือทนายมนุษย์

### โหมด `grid`

รัน **tabular review** กับ column schema เหนือโฟลเดอร์เอกสาร:

- **หนึ่งแถวต่อหนึ่งเอกสาร**
- **หนึ่งคอลัมน์ต่อหนึ่งข้อมูลที่ต้องดึง** (เช่น: คู่สัญญา, วันมีผล, เงื่อนไขการบอกเลิก, ค่าตอบแทน)
- **ทุก cell มี citation กลับไปยังข้อความต้นฉบับ** — ถ้าไม่มี quote = cell ที่คุณกุขึ้นมาเอง

นี่คือ **M&A diligence workhorse** — งานหลักของทีม corporate ในการตรวจสอบบริษัทเป้าหมายก่อนเซ็นสัญญา

## โครงสร้างไฟล์ใน cookbook

```text
managed-agent-cookbooks/diligence-grid/
├── README.md                       # 6.6 KB — คู่มือ
├── agent.yaml                      # 4.9 KB — manifest หลัก
├── steering-examples.json          # 759 B  — ตัวอย่าง steering event
└── subagents/
    ├── doc-reader.yaml             # 3.4 KB — อ่าน VDR
    ├── extractor.yaml              # 3.4 KB — ดึงข้อมูลตาม schema
    ├── normalizer.yaml             # 3.2 KB — ทำความสะอาดข้อมูล
    └── grid-writer.yaml            # 4.3 KB — เขียนผล CSV (ตัวเดียวที่มี Write)
```

## Plugin ต้นทาง

cookbook นี้ใช้ **system prompt** และ **skill** จาก plugin [`corporate-legal`](../03-plugins/) — โดยเฉพาะ skill `tabular-review` ที่เป็นหัวใจของการทำงาน

```yaml
skills:
  - { from_plugin: ../../corporate-legal }
```

## โมเดลความปลอดภัย 4-Tier

VDR — Box, Datasite, Intralinks, iManage — เก็บ **เอกสารที่คู่สัญญาฝั่งตรงข้ามอัปโหลด** ดังนั้นข้อความทุกฉบับคือ **untrusted input** ที่อาจมี prompt injection ฝังในตัว

cookbook นี้ใช้ **4 ชั้นแยก** เพื่อป้องกัน:

| ชั้น | สัมผัสเอกสาร | Tools | MCP |
|---|---|---|---|
| **`doc-reader`** | ใช่ (อ่านอย่างเดียว) | `Read`, `Grep` | Box, Drive, iManage (อ่าน) |
| **`extractor`** | ใช่ (อ่านอย่างเดียว) | `Read`, `Grep` | ไม่มี |
| `normalizer` / Orchestrator | ไม่ | `Read`, `Grep`, `Glob`, `Agent` | Definely (optional, อ่าน) |
| **`grid-writer`** (Write-holder) | ไม่ | `Read`, `Write` | ไม่มี |

`doc-reader` และ `extractor` คืนผลเป็น **JSON ที่ผ่าน schema validation และจำกัดความยาว** — orchestrator และ `normalizer` เห็นเฉพาะ structured data ไม่เคยเห็นข้อความสัญญาดิบ

`grid-writer` ผลิตไฟล์ 3 ไฟล์:

1. `./out/diligence-grid-<date>.csv` — ตารางค่า (values grid)
2. `./out/diligence-grid-<date>_sources.csv` — ตาราง quote ต้นฉบับ + ตำแหน่ง (sources grid)
3. `./out/diligence-grid-<date>-summary.md` — สรุปสำหรับมนุษย์

## CSV Formula Injection — Defense ที่จำเป็น

`grid-writer` ตรวจสอบทุก cell ที่จะเขียนลง CSV ว่าตัวอักษรแรกตรงกับอันตรายหรือไม่: `=`, `+`, `-`, `@`, tab, carriage return

ถ้าตรง — prefix ด้วย apostrophe เดี่ยว `'` ก่อน

ทำไมต้องทำ? เพราะ:

- สัญญาจาก counterparty อาจมีข้อความเช่น `=HYPERLINK("evil.com", "click")` ที่เมื่อ Excel/Sheets เปิด — จะกลายเป็น hyperlink สำหรับ exfiltrate ข้อมูล
- รุ่นเก่าของ Excel มี **DDE injection** ผ่าน `=cmd|...` — คำสั่งจะรันทันทีที่เปิดไฟล์
- `sources.csv` คือพื้นที่เสี่ยงที่สุด — verbatim quote คือสิ่งที่ attacker ควบคุมได้

## Xlsx เป็นเรื่องของฝั่ง Deploy

cookbook ส่งออก **CSV เท่านั้น** — ทีมที่ต้องการ `.xlsx` พร้อม:

- hidden `_source` column
- cell comment ที่แสดง quote เมื่อ hover
- state-based fill (สีตามสถานะ answered / not_present / unclear)
- `Verified` dropdown ต่อคอลัมน์
- `_schema` และ `_summary` sheet

ต้อง transform CSV → xlsx ด้วยเครื่องมือฝั่งตัวเอง (Claude in Excel, openpyxl, หรือ Google Sheets API) — agent ไม่ส่ง xlsx จากเซิร์ฟเวอร์ headless เพราะต้องการ trusted runtime และ macro surface ที่ cookbook **เจตนาไม่สมมุติว่ามี**

## Steering Events ตัวอย่าง

จาก `steering-examples.json`:

```json
[
  {
    "event": "Review folder /02-Contracts against schema ma-diligence",
    "description": "Grid mode — รันชุดคอลัมน์มาตรฐาน M&A diligence เหนือโฟลเดอร์ material contracts และผลิต CSV + summary"
  },
  {
    "event": "Watch VDR for new uploads since 2026-05-01, flag Material Contracts and IP",
    "description": "Watch mode — แสดงเอกสารใหม่ทุกชิ้น จัดประเภทตาม request-list categories และ flag ที่ high-priority"
  },
  {
    "event": "Re-run grid for documents [DOC-001, DOC-002, DOC-003] with added column termination_for_convenience",
    "description": "Grid mode — แก้ schema + รันกับ subset เอกสาร โดยรักษา row เดิมสำหรับเอกสารที่ไม่ได้เปลี่ยน"
  }
]
```

## วิธี Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export BOX_MCP_URL=...
export GDRIVE_MCP_URL=...
export IMANAGE_MCP_URL=...          # optional
export DEFINELY_MCP_URL=...         # optional — สำหรับ clause-structure QA
../../scripts/deploy-managed-agent.sh diligence-grid
```

รายละเอียดเต็มในหมวด **08-deployment** หัวข้อ managed-agents

## ต้องปรับอะไรก่อน Deploy

### 1. ชี้ MCP ไปยัง VDR จริง

ตั้ง `BOX_MCP_URL` / `GDRIVE_MCP_URL` / `IMANAGE_MCP_URL` ตาม data room ที่ทีมใช้ — ค่าตั้งต้นเปิด Box + Google Drive ถ้าใช้ iManage หรือ Datasite เป็นหลัก ให้สลับ `default_config` ใน `agent.yaml`

ถ้าใช้ Intralinks หรือ Datasite — เพิ่ม entry ใน `mcp_servers` และ `tools`

### 2. Column Schema

ค่าตั้งต้นใช้ **M&A diligence standard column set** ใน `corporate-legal/skills/tabular-review/references/ma-diligence-columns.md`

ปรับตามประเภท deal ของทีม:

- Tech / IP — เพิ่มคอลัมน์ assignment, joint development, work-for-hire
- Healthcare — เพิ่ม HIPAA, Medicare/Medicaid, FDA
- Real Estate — เพิ่ม encumbrance, easement, zoning
- Government Contractor — เพิ่ม flow-down, ITAR, FAR
- Regulated Financial — เพิ่ม BSA, AML, GLBA

### 3. Output Destination

ค่าตั้งต้น output ลง `./out/` — wire ไปยัง deal folder, Google Drive, iManage, หรือ Box ผ่าน deploy pipeline ของทีม

**อย่าให้ `grid-writer` มี MCP สำหรับ upload** — handoff ไปยังขั้น upload สะอาดกว่า และเก็บ Write tier ให้แยก

### 4. Request-List Categories

โหมด `watch` จัดประเภทตาม category ที่อยู่ใน `corporate-legal` `CLAUDE.md` ของทีม — รัน `/corporate-legal:cold-start-interview` ที่นั่นก่อน wire watch mode เข้ากับ deal จริง

### 5. Work-Product Header

`grid-writer` นำ header จาก `## Outputs` configuration ของทีมมา prepend — ยืนยัน header กับทีมกฎหมายก่อน deploy (แตกต่างกันระหว่าง reviewer ที่เป็น lawyer vs non-lawyer)

### 6. Slack Routing

agent **ไม่ post ตรง** — รายงานเป็นไฟล์ `handoff_request` บอก orchestrator ว่าจะส่งไปช่องไหน ตั้งช่อง deal ใน `CLAUDE.md` House style section

## Use Case จริง

### สถานการณ์: ทีม corporate กำลังตรวจสอบบริษัทเป้าหมายในการเข้าซื้อกิจการมูลค่า 200 ล้านดอลลาร์

**ก่อน Diligence Grid:**
- ทีมเปิด VDR ที่ counterparty เปิดให้
- แบ่งทนาย 5 คน แต่ละคนรับ 200 สัญญา
- 2 สัปดาห์อ่านทุกฉบับ กรอกตาราง Excel ที่ความถูกต้องต่างกันตามคน
- ทุก cell ใน Excel ไม่มี citation — ตอนรีวิวต้องเปิดสัญญาทุกฉบับซ้ำ

**กับ Diligence Grid:**
- ทีม IT ตั้ง `BOX_MCP_URL` ชี้ไป VDR
- รัน steering event: `Review folder /02-Material-Contracts against schema ma-diligence`
- 2 ชั่วโมงผ่าน — ได้ CSV + sources CSV + summary
- ทนาย Senior **ไม่อ่านสัญญาดิบ** อ่าน summary + คอลัมน์ที่ normalizer flag
- เมื่อต้องการตรวจ cell หนึ่ง — เปิด sources CSV ดู verbatim quote + location → เปิดสัญญาตรง paragraph นั้น

**สิ่งที่ทีมยังต้องทำ:**

- รีวิว `unclear` และ `needs_review` ทุก cell
- ตรวจ flag จาก `normalizer` (เช่น contract ที่ระบุ governing law แบบไม่มาตรฐาน)
- ตัดสินใจว่าอะไรไป **representation**, **disclosure schedule**, หรือ **diligence memo**

> **คำเตือนสำคัญ:** ทุก cell คือ **lead** ไม่ใช่ **finding** — Diligence Grid ไม่ใช่ representation, disclosure schedule, หรือ diligence memo จนกว่าทนายจะอ่านเอกสารต้นฉบับและลงนาม

## ปรัชญา — Reviewer Time Scales with Quantity

จุดสำคัญที่ทีมต้องเข้าใจ: **เวลาของ reviewer ไม่ได้ลดเหลือศูนย์**

- เวลาเดิม: อ่านสัญญา 1000 ฉบับ = 1000 × เวลาต่อฉบับ
- เวลาใหม่: รีวิว 1000 cell ที่มี state + quote = 1000 × **เวลาตรวจ quote** (เร็วกว่า 5-10 เท่า)

แต่ **เวลาไม่เป็นศูนย์** — agent **เร่งการรีวิว** ไม่ได้ **แทนการรีวิว**

`Watch mode` ก็เช่นกัน — classifier อาจ tag "low priority" กับเอกสารที่จริง ๆ คือ **side letter ที่เปลี่ยนแปลง deal** — treat รายงานเป็น **queue** ไม่ใช่ **filter**

ทุก agent ใน cookbook นี้ทำงานบนหลักการเดียวกัน: **agent ผลิต lead — lawyer ตัดสิน**
