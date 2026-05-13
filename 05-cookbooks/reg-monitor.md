---
title: Reg Monitor
parent: 05 Cookbooks
nav_order: 4
---

# Reg Monitor — เฝ้ากฎใหม่และตรวจช่องโหว่ Policy

> "Digest items are screened leads, not legal conclusions."
> — `managed-agent-cookbooks/reg-monitor/README.md`

## จุดประสงค์

Reg Monitor คือ cookbook สำหรับเฝ้า **feed ของกฎและประกาศจากผู้กำกับดูแล (regulatory feeds)** — ตามตารางที่ทีมตั้ง agent จะ:

1. ตรวจ feed ตาม schedule
2. **กรองตาม materiality threshold** ของทีม
3. **ตรวจ gap** กับ policy library สำหรับรายการที่ "ต้องสำคัญเสมอ"
4. เขียน **digest** ส่งให้ทีมรีวิว

ทำงานบนแหล่งข้อมูล:

- **Federal Register** (API สาธารณะของรัฐบาลกลางสหรัฐฯ — ฟรี)
- **Agency RSS** (SEC, FINRA, FDA, FCC ฯลฯ)
- **Thomson Reuters Regulatory Intelligence** (optional — paid)

## โครงสร้างไฟล์ใน cookbook

```text
managed-agent-cookbooks/reg-monitor/
├── README.md                       # 5.7 KB — คู่มือ
├── agent.yaml                      # 2.0 KB — manifest หลัก
├── steering-examples.json          # 622 B  — ตัวอย่าง steering event
└── subagents/
    ├── feed-reader.yaml            # 3.7 KB — อ่าน feed
    ├── materiality-filter.yaml     # 2.3 KB — กรองความสำคัญ
    └── digest-writer.yaml          # 4.0 KB — เขียน digest (ตัวเดียวที่มี Write)
```

## Plugin ต้นทาง

cookbook นี้ใช้ system prompt จาก plugin [`regulatory-legal`](../03-plugins/) — agent `reg-change-monitor.md` และ skill `reg-feed-watcher` + `policy-diff`

```yaml
system:
  file: ../../regulatory-legal/agents/reg-change-monitor.md
  append: |
    You are running headless. Write the digest...
```

## โมเดลความปลอดภัย 3-Tier + Egress Allowlist

feed content (Federal Register entries, agency RSS posts, TR alert notifications) คือ **untrusted input**:

- regulator ที่เป็น federal agency เชื่อถือได้ — แต่ feed format อาจมี payload ที่ฝัง
- attacker อาจส่ง phishing ผ่าน fake "Federal Register" link
- TR alert อาจมี URL ที่ link ไปยังเว็บอื่น

| ชั้น | สัมผัส feed | Tools | MCP |
|---|---|---|---|
| **`feed-reader`** | ใช่ | `Read`, `Grep`, `WebFetch` เท่านั้น | ไม่มี |
| `materiality-filter` / Orchestrator | ไม่ | `Read`, `Grep`, `Glob`, `Agent` | gdrive (orchestrator only) |
| **`digest-writer`** (Write-holder) | ไม่ | `Read`, `Write`, `Edit` | ไม่มี |

`feed-reader` คืน **schema-validated JSON ที่จำกัดความยาว** — และยังมี **egress allowlist** ที่ enforced ระดับ tool:

```text
allowed_hosts:
- federalregister.gov
- *.sec.gov
- *.gpo.gov
- *.regulations.gov
- *.europa.eu
- *.gov.uk
+ domains ที่ทีม configure
```

ถ้า feed item มี URL นอก allowlist — return เป็น flagged item (summary prefix "host not on configured allowlist") **ไม่ fetch** — ไม่ rewrite URL ไม่ "แก้" ให้ดูเหมือนได้รับอนุญาต

`materiality-filter` คือ **pure computation** — ไม่มี MCP, ไม่มี web — ทำงานจาก validated JSON + regulatory-legal config บน disk

## Handoffs

`digest-writer` ผลิต `./out/reg-digest-<YYYY-MM-DD>.md` และส่ง `handoff_request` สำหรับ Slack delivery

agent **ไม่ส่ง Slack message เอง** — orchestrator route handoff ไปยัง Slack send worker โดยใช้ช่องจาก **House style configuration** ของทีม

## Steering Events ตัวอย่าง

```json
[
  {
    "event": "Check feeds as-of 2026-05-12, materiality threshold: medium",
    "description": "Scheduled weekly digest run"
  },
  {
    "event": "Deep check: new EU AI Act implementing regulation, policy area: ai-governance",
    "description": "Targeted check on a known development"
  },
  {
    "event": "Gap check against FINRA Rule 4512 amendment, docket SR-FINRA-2026-007",
    "description": "Specific gap analysis on a flagged item"
  }
]
```

3 ตัวนี้ครอบ 3 รูปแบบการใช้:

1. **Scheduled sweep** — weekly run ตามตาราง
2. **Targeted deep check** — เมื่อทีมรู้แล้วว่ามี development สำคัญ
3. **Gap analysis** — เมื่อ item ถูก flag แล้ว ต้องการเปรียบเทียบกับ policy ปัจจุบัน

## วิธี Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export GDRIVE_MCP_URL=...
../../scripts/deploy-managed-agent.sh reg-monitor
```

รายละเอียดเต็มในหมวด **08-deployment** หัวข้อ managed-agents

## ต้องปรับอะไรก่อน Deploy

### 1. ชี้ `feed-reader` ไปยังแหล่งของทีม

ค่าตั้งต้น = **Federal Register** (API สาธารณะ ไม่ต้อง MCP)

ถ้าทีมสมัครสมาชิก:

- **Thomson Reuters Regulatory Intelligence**
- **Bloomberg Law**
- **Direct agency RSS** ของ regulator ที่เกี่ยวข้อง

→ เพิ่ม endpoint ใน feed-reader web_fetch allowlist + ปรับ scan plan ของ orchestrator

ทีมที่มี **free sources เท่านั้น** — Federal Register API alone ใช้งานได้

### 2. ตั้ง (optionally) Thomson Reuters MCP URLs

TR comment-out ใน manifest — wire และ flip `enabled: true` ถ้าทีมมี subscription

### 3. Digest Delivery Channel

`digest-writer` ส่ง `handoff_request` ที่ระบุ Slack channel — orchestrator อ่านช่องจาก **`regulatory-legal` config → House style → Reg digest**

**ตั้งช่องก่อน first scheduled run** — ถ้าไม่ handoff จะ dead-letter

ทีมที่อยาก digest ผ่าน email หรือ Confluence page — swap handoff target ใน orchestrator allowlist

### 4. Tune Materiality Threshold

`materiality-filter` อ่าน config `## Materiality threshold` ของทีม:

- **Always material** — ต้อง flag เสมอ
- **Review-worthy** — ทีมตัดสิน
- **FYI** — แค่บอกให้รู้

ยืนยัน tiers สะท้อน risk posture ปัจจุบันก่อน enable scheduled run:

- ตั้งต่ำเกินไป → digest ล้น
- ตั้งสูงเกินไป → พลาดภาระที่มี deadline

### 5. Update Watchlist

`materiality-filter` อ่าน `## Regulators we watch` table — เพิ่ม/ลบ regulator ตาม footprint ของทีม:

- เปิดสายธุรกิจใหม่ที่มี regulator ใหม่ → ต้องเพิ่ม
- ขยายไปต่างประเทศ → ต้องเพิ่ม international regulator
- ทิ้งสายธุรกิจ → ลบ regulator ที่ไม่เกี่ยว

### 6. ยืนยัน Work-Product Header

ตามมาตรฐาน cookbook — ตรวจ header กับ GC ก่อน

### 7. Cadence

ค่าตั้งต้น = weekly

- **Active regulatory environments** (financial services rulemaking cycles, cross-border AI regulation) — daily อาจเหมาะ
- **Stable industries** — monthly อาจพอ

Cadence อยู่ใน workflow engine ของทีมเอง — cookbook ไม่ schedule ตัวเอง

## Use Case จริง

### สถานการณ์: ทีม regulatory ของสถาบันการเงินขนาดกลาง

**ก่อน Reg Monitor:**
- ทนาย regulatory อ่าน Federal Register email digest ทุกเช้า (1-2 ชั่วโมง)
- ต้องจำว่ากฎอะไรของ regulator ใดมา trigger policy review
- บางครั้งพลาด FINRA notice ที่ไม่ได้สมัคร alert
- เมื่อพบ regulation ใหม่ — ค้น policy library ด้วยมือเพื่อหา gap

**กับ Reg Monitor:**
- Weekly run จันทร์เช้า — digest ลง `#regulatory` channel
- Digest แยก:
  - **Material** (ต้องทำอะไรบางอย่าง)
  - **Review-worthy** (รีวิว ตัดสิน)
  - **FYI** (รู้ไว้)
- ทุก item ที่ Material — agent ทำ **policy gap check** อัตโนมัติ — แสดงว่า policy ใดของทีมอาจมีช่องโหว่
- ทีมเปิด digest 15 นาที = เห็นทุกอย่าง

**สิ่งที่ทีมยังต้องทำ:**

- review ทุก digest item — แม้ FYI (อาจมีของหลุดมา)
- ตัดสินใจว่ารายการนั้น **ต้องการ action, disclosure, policy change หรือ escalation**
- ตรวจ gap analysis — agent บอกว่า "อาจมี gap" ไม่ใช่ "มี gap แน่นอน"

## คำเตือนสำคัญ

### 1. Digest Items คือ Screened Leads ไม่ใช่ Legal Conclusion

- materiality filter ใช้ **heuristic** ไม่ใช่ legal judgment
- "Informational" อาจเป็น material กับธุรกิจของทีม
- "Material" อาจไม่เกี่ยวจริง

### 2. Policy Gap Check คือ First Pass

- compare ใช้ heuristic
- "gap" = lead ให้ทนายประเมิน ไม่ใช่ verdict
- "aligned" = ไม่ได้รับรอง compliance

### 3. Materiality Threshold คือ Calibration ไม่ใช่ Law

ถ้า `## Materiality threshold` ของทีม stale หรือ tune ไว้สำหรับ risk posture เดิม → triage stale

**ตรวจก่อน enable scheduled run**

### 4. Watchlist คือ Coverage Assertion

regulator ที่ไม่อยู่ใน watchlist อาจ publish อะไรที่ material — **ขาด regulator = config bug ไม่ใช่ feed bug**

## ปรัชญา — Lead ไม่ใช่ Calendar

หัวใจของ Reg Monitor คือ **surfacing** — agent นำสิ่งที่ ควรเห็น ขึ้นมาให้เห็น — แต่ไม่ตัดสิน

จาก append ของ system prompt:

> "Your output is a lead, not a legal conclusion. A materiality classification, a policy-gap flag, and an 'informational' tag are all screening calls — a lawyer reviews every digest item and decides whether a regulatory change requires action, disclosure, policy change, or escalation."

ทีมที่ใช้ Reg Monitor ดี = ทีมที่ rotation ของทนายอ่าน digest ทุกครั้ง — ไม่ใช่ทีมที่ "trust the agent"

> **Surfacing ≠ Filtering** — agent ลด volume ของ noise ไม่ใช่กรอง signal ออก
