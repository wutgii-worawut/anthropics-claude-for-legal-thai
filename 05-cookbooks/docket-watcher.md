---
title: Docket Watcher
parent: 05 Cookbooks
nav_order: 2
---

# Docket Watcher — เฝ้าสำนวนความและกำหนดวันยื่นเอกสาร

> "Computed deadlines are leads, not calendar entries. Missing a court deadline has malpractice consequences."
> — `managed-agent-cookbooks/docket-watcher/README.md`

## จุดประสงค์

Docket Watcher คือ cookbook สำหรับเฝ้า **สำนวนความ (court docket)** ของคดีในแฟ้มลิติเกชั่นที่กำลังดำเนินอยู่

- **Trellis** ครอบคลุมศาลรัฐ (state trial courts) ในสหรัฐฯ
- **CourtListener** / **PACER** ครอบคลุมศาลกลาง (federal courts)

สำหรับทุกคดีที่กำลัง active agent จะ:

1. ดึง **filing ใหม่** ตั้งแต่การตรวจครั้งล่าสุด
2. **map filing type → candidate deadline** (เช่น ยื่น Motion to Dismiss → opposition deadline = 21 วัน)
3. **cross-reference** กับประวัติคดี + deliverable ที่ยังเปิดอยู่
4. ผลิต:
   - รายงานสถานะคดี (narrative)
   - **deadline feed** ที่ structured (สำหรับ docketing system นำเข้า)

## โครงสร้างไฟล์ใน cookbook

```text
managed-agent-cookbooks/docket-watcher/
├── README.md                       # 5.3 KB — คู่มือ
├── agent.yaml                      # 2.1 KB — manifest หลัก
├── steering-examples.json          # 464 B  — ตัวอย่าง steering event
└── subagents/
    ├── docket-reader.yaml          # 2.9 KB — อ่าน filing
    ├── deadline-mapper.yaml        # 2.5 KB — map filing → deadline
    └── tracker-writer.yaml         # 6.3 KB — เขียนรายงาน (ตัวเดียวที่มี Write)
```

## Plugin ต้นทาง

cookbook นี้ใช้ system prompt และ skill จาก plugin [`litigation-legal`](../03-plugins/) — โดยเฉพาะ agent `docket-watcher.md` ที่มี logic เดียวกัน

```yaml
system:
  file: ../../litigation-legal/agents/docket-watcher.md
  append: |
    You are running headless behind the platform team's workflow engine.
    Produce files in ./out/...
```

ผลคือ — agent ตัวเดียวกัน ใช้ใน Claude Code interactive ได้ และ deploy เป็น scheduled job ได้

## โมเดลความปลอดภัย 3-Tier

filing ใน court docket เป็น **public records** — ใคร ๆ อ่านได้ แต่ก็คือ **untrusted input**:

- ผู้ยื่นควบคุมข้อความที่ใส่ใน motion, brief, exhibit
- ผู้ยื่นอาจฝัง prompt injection ใน footnote, exhibit หรือ URL
- ผู้ยื่นที่เป็น pro se litigant อาจตั้งใจหรือไม่ตั้งใจใส่ข้อความที่ทำให้ agent สับสน

| ชั้น | สัมผัส filing | Tools | MCP |
|---|---|---|---|
| **`docket-reader`** | ใช่ | `Read`, `Grep` เท่านั้น | trellis, courtlistener (อ่าน) |
| `deadline-mapper` / Orchestrator | ไม่ — เห็น JSON | `Read`, `Grep`, `Glob`, `Agent` | gdrive (อ่าน — สำหรับ jurisdiction rule config) |
| **`tracker-writer`** (Write-holder) | ไม่ | `Read`, `Write`, `Edit` | ไม่มี |

`docket-reader` คืน **schema-validated JSON ที่จำกัดความยาว** — `deadline-mapper` ไม่มี MCP, ไม่มี web เพราะใช้กฎที่ทีม configure เอง `tracker-writer` ผลิต `./out/docket-report-<date>.md` และ `./out/deadlines.yaml` โดยไม่เห็น filing ดิบ

## Steering Events ตัวอย่าง

```json
[
  {
    "event": "Watch docket 2:26-cv-00315 in N.D. Cal., matter M-2026-042",
    "description": "Single-matter federal docket sweep"
  },
  {
    "event": "Daily sweep: all active matters with hearings in next 14 days",
    "description": "Scheduled portfolio sweep for near-term hearings"
  },
  {
    "event": "New filing detected: Motion to Dismiss in matter M-2026-042, compute opposition deadline",
    "description": "Event-triggered deadline computation for a specific filing"
  }
]
```

ตัวที่สามน่าสนใจ — เป็น **event-triggered** ที่กระตุ้นโดย webhook จาก docketing system ที่ตรวจพบ filing ใหม่ — ไม่ต้องรอ scheduled sweep

## วิธี Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export TRELLIS_MCP_URL=...
export COURTLISTENER_MCP_URL=...
export GDRIVE_MCP_URL=...
../../scripts/deploy-managed-agent.sh docket-watcher
```

รายละเอียดเต็มในหมวด **08-deployment** หัวข้อ managed-agents

## ต้องปรับอะไรก่อน Deploy

### 1. MCP URLs

- `TRELLIS_MCP_URL` — endpoint Trellis ของทีม + authentication ที่ platform ต้องการ
- `COURTLISTENER_MCP_URL` — endpoint CourtListener
- `GDRIVE_MCP_URL` — ที่ที่ jurisdiction rule table อยู่

### 2. โหลด Portfolio

agent อ่าน `matters/_log.yaml` พร้อม `docket_id` และ `court` ต่อ matter จาก `litigation-legal` configuration

ถ้า **docketing system เป็น source of truth** — front มันด้วย MCP หรือ scheduled sync เข้ามาที่ config path

### 3. Jurisdiction Rules

`deadline-mapper` ต้องการ **local-rule table** สำหรับทุกศาลใน portfolio:

- **Federal rules** — encode ครั้งเดียว ใช้ได้ทั้ง circuit
- **State trial courts** — แตกต่างทุกรัฐ
- **Individual judges** — บางผู้พิพากษามี standing order แก้กฎ default

หากศาลที่ไม่อยู่ใน table ปรากฏ — mapper ต้องผลิต `confidence: low` + `needs_verification: true` **ไม่มี silent default**

### 4. Wire Delivery

- `./out/deadlines.yaml` → docketing system นำเข้า
- รายงาน narrative → Slack, email หรือ matter management workspace
- Flag ที่ critical → woke-up routing (ใครได้รับโทรศัพท์)

### 5. ตั้ง Schedule

- **Weekly** สำหรับ matter ส่วนใหญ่
- **Daily** สำหรับ:
  - matter ที่มี hearing ภายใน 14 วัน
  - matter ใน `trial` posture หรือ late-`discovery`
  - matter ที่ `risk: critical`

## คำเตือนสำคัญ — Computed Deadlines คือ Lead ไม่ใช่ Calendar Entry

นี่คือคำเตือนที่ปรากฏใน README ของ cookbook นี้ **เด่นที่สุด**:

> "The computed deadlines this agent produces require human verification against the controlling local rule, standing order, and case management order before they are calendared. Missing a court deadline has malpractice consequences."

ทำไม?

- **กฎกำหนดเวลาของศาล** แตกต่างกันตาม jurisdiction, court, judge และ local rule
- **Standing order** ของผู้พิพากษาแต่ละคนอาจแก้กฎ default
- **Case management order** เฉพาะคดีอาจแก้กฎอีกชั้น
- **การพลาด deadline ของศาล มีผลต่อ malpractice insurance** ของทนาย

ทุก deadline ที่ agent ผลิตจึงมี field:

- `confidence: low | medium | high`
- `needs_verification: true | false`

รายงานแยก low-confidence entries และ stamp **verification callout** บนทุกอย่างที่ไม่ได้มาจาก federal rule ที่ชัดเจน

> **Treat that as the minimum — not the ceiling — of human review.**

## คำเตือนเพิ่ม

### 1. Filing Classification เป็น Heuristic

filing ที่ classify ผิด อาจให้ deadline rule ผิด — ทนาย docketing **อ่าน filing เอง** ไม่ trust label

### 2. Unknown Court ≠ Default

ถ้า jurisdiction-rule table ไม่ครอบคลุมศาลใด — mapper ต้องคืน `confidence: low` ไม่ใช่ silent default — ถ้าเห็น confident deadline บนศาลที่ไม่คุ้น ถือว่า table stale จนกว่าจะพิสูจน์

### 3. Quiet Docket ≠ Clean Docket

- เสมียนศาล (clerk) docket ช้า
- Minute entry บางครั้งมาช้าหลายวันหลังเหตุการณ์
- "No new filings" = ข้อความเกี่ยวกับ feed ไม่ใช่ข้อความเกี่ยวกับคดี

## Use Case จริง

### สถานการณ์: สำนักงานลิติเกชั่นมี 80 คดี active กระจายหลายศาล

**ก่อน Docket Watcher:**
- Paralegal เปิด PACER + Trellis ทุกเช้า ตรวจ filing ใหม่
- เมื่อพบ Motion to Dismiss — ค้นกฎ FRCP + local rule + standing order ของผู้พิพากษา
- คำนวณ deadline ด้วยมือ ใส่ใน docketing system
- ใช้เวลาเฉลี่ย **45 นาที** ต่อ filing ใหม่ × 5-10 filing/วัน = 4-8 ชั่วโมง/วัน

**กับ Docket Watcher:**
- ทุกเช้า scheduled run ที่ 7:00
- ทุกคดีที่มี hearing ภายใน 14 วัน — daily run
- รายงานส่งเข้า Slack `#docket-alerts` พร้อม:
  - filing ใหม่ทั้งหมด
  - deadline ที่ map ได้ พร้อม `confidence` และ `needs_verification`
  - link กลับไป filing
- `./out/deadlines.yaml` → docketing system นำเข้าเป็น **draft**
- Paralegal **ตรวจ** ทุก deadline (ไม่ใช่คำนวณใหม่) — ลดเวลาจาก 45 นาที เหลือ 5-10 นาที

**สิ่งที่ทีมยังต้องทำ:**

- ตรวจ filing classification ที่ agent ทำ
- ยืนยัน deadline กับ local rule + standing order
- รับผิดชอบ deadline ที่ docket — Docket Watcher **ไม่รับผิดชอบ malpractice**

## ปรัชญา — Loud is Correct

จุดเด่นของ cookbook นี้คือ **ไม่ลด volume ของ warning** ใน append ของ system prompt:

> "Filing classifications are heuristic... Missing a court deadline has malpractice consequences... Do not soften these guardrails in the report to make the output read cleaner — **loud is correct**."

แปลว่า — Anthropic ไม่ต้องการให้ agent "ตัดเสียงเตือน" ให้ output ดู clean — เพราะการพลาด deadline = malpractice = ความเสียหายจริง

**ดังเป็นถูก** เสียงเตือนที่รก หน้า > deadline ที่พลาดเงียบ ๆ
