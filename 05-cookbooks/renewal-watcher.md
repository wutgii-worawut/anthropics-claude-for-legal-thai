---
title: Renewal Watcher
parent: 05 Cookbooks
nav_order: 5
---

# Renewal Watcher — เฝ้าวันต่อสัญญาและกำหนดบอกเลิก

> "Cancel-by dates and renewal terms pulled from contract metadata can be wrong."
> — `managed-agent-cookbooks/renewal-watcher/README.md`

## จุดประสงค์

Renewal Watcher คือ cookbook สำหรับเฝ้า **คลังสัญญา (contract repository / CLM)** หา:

- **วันต่อสัญญา (renewal deadline)** ที่กำลังมา
- **กำหนดบอกเลิก (cancel-by deadline)** ที่ต้องแจ้งก่อนสัญญาต่ออัตโนมัติ
- **การเบี่ยงเบนจาก playbook** (playbook deviation) — เงื่อนไขในสัญญาที่ออกนอกมาตรฐานทีม
- **trigger สำหรับ escalation** — ตามตารางอำนาจอนุมัติของทีม

agent จะ:

1. Scan repository
2. Cross-reference กับ playbook
3. Flag สัญญาที่:
   - มี deadline ใกล้
   - มี playbook deviation
   - ต้อง escalation
4. เขียน alert report

ระบบ CLM ที่รองรับ (ค่าตั้งต้น):

- **Ironclad** (CLM หลัก — เพราะ paired plugin `commercial-legal` ตั้งไว้แบบนั้น)

ทีมที่ใช้:

- **Agiloft** / **Conga** / **CLM อื่น** — swap MCP endpoint
- **iManage** (ถ้าสัญญาเซ็นแล้วเก็บที่นี่) — flip enabled
- **Google Drive ของ PDF** — ใช้ gdrive MCP

## โครงสร้างไฟล์ใน cookbook

```text
managed-agent-cookbooks/renewal-watcher/
├── README.md                       # 5.1 KB — คู่มือ
├── agent.yaml                      # 1.4 KB — manifest หลัก
├── steering-examples.json          # 425 B  — ตัวอย่าง steering event
└── subagents/
    ├── repo-reader.yaml            # 3.7 KB — อ่าน CLM
    ├── deadline-calculator.yaml    # 3.3 KB — คำนวณ deadline
    └── alert-writer.yaml           # 4.4 KB — เขียน alert (ตัวเดียวที่มี Write)
```

## Plugin ต้นทาง

cookbook นี้ใช้ system prompt และ skill จาก plugin [`commercial-legal`](../03-plugins/) — agent `renewal-watcher.md` และ skill `renewal-tracker`

```yaml
system:
  file: ../../commercial-legal/agents/renewal-watcher.md
  append: |
    You are running headless. Produce files in ./out/...
```

## โมเดลความปลอดภัย 3-Tier

contract text, counterparty message, และ CLM comment เป็น **untrusted input**:

- ข้อความสัญญาควบคุมโดย counterparty (อาจใส่ string ที่ exploit toolchain)
- comment ใน CLM อาจมาจาก vendor ภายนอกที่มี access
- notice provision อาจถูกแก้ไขใน amendment ที่ไม่ได้ ingest

| ชั้น | สัมผัสสัญญา | Tools | MCP |
|---|---|---|---|
| **`repo-reader`** | ใช่ | `Read`, `Grep` เท่านั้น | ironclad, gdrive (อ่าน); imanage off ค่าตั้งต้น |
| `deadline-calculator` / Orchestrator | ไม่ | `Read`, `Grep`, `Glob`, `Agent` | ไม่มี |
| **`alert-writer`** (Write-holder) | ไม่ | `Read`, `Write`, `Edit` | ไม่มี |

`repo-reader` คืน **schema-validated JSON ที่จำกัดความยาว** `deadline-calculator` คือ **pure computation** — ไม่มี MCP, ไม่มี web — ทำงานจาก JSON + playbook config บน disk

`alert-writer` ผลิต `./out/renewal-alerts-<YYYY-MM-DD>.md` และส่ง `handoff_request` สำหรับ Slack delivery

## Handoffs ระหว่าง Agents

`handoff_request` จาก `alert-writer` route ได้:

- **slack_send_message** — ส่ง alert ไปยังช่องที่ตั้งไว้
- **`deal-debrief`** agent — เมื่อต้องการ **post-signature deviation check** บนสัญญาที่เพิ่งลงนาม
- **`playbook-monitor`** agent — เมื่อ renewal-time deviation **สะสมเป็น pattern** (สัญญาหลายฉบับเบี่ยงเบนแบบเดียวกัน อาจหมายความว่า playbook outdated)

agent **ไม่เรียกกันตรง** — routing เป็นงานของ orchestrator

## Steering Events ตัวอย่าง

```json
[
  {
    "event": "Scan renewals 0-180 days out, flag playbook deviations, as-of 2026-05-07",
    "description": "Scheduled weekly sweep — the default Monday-morning run"
  },
  {
    "event": "Targeted scan: counterparty Acme Corp, all active agreements, surface cancel-by windows and price escalators",
    "description": "Ad-hoc scan before a renegotiation or relationship review"
  },
  {
    "event": "Deviation check: contract IC-2026-00412 post-signature, compare against playbook and flag any terms outside standard",
    "description": "Follow-up after a negotiated deal closes to confirm no late-stage changes breached the playbook"
  }
]
```

3 รูปแบบ:

1. **Scheduled sweep** — weekly Monday morning
2. **Ad-hoc counterparty scan** — ก่อน renegotiation
3. **Post-signature check** — หลังเซ็น verify ว่าไม่มี term ที่หลุดมาเปลี่ยน

## วิธี Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export IRONCLAD_MCP_URL=...
export GDRIVE_MCP_URL=...
# Optional
export IMANAGE_MCP_URL=...
export DOCUSIGN_MCP_URL=...
../../scripts/deploy-managed-agent.sh renewal-watcher
```

รายละเอียดเต็มในหมวด **08-deployment** หัวข้อ managed-agents

## ต้องปรับอะไรก่อน Deploy

### 1. ชี้ไปยัง CLM

ค่าตั้งต้น = `IRONCLAD_MCP_URL`

- **iManage** — flip `imanage` `default_config: { enabled: true }` ใน `agent.yaml` และ `subagents/repo-reader.yaml` ตั้ง `IMANAGE_MCP_URL`
- **Google Drive folder** — อาศัย `gdrive` และ fallback search path ใน repo-reader
- **CLM อื่นที่ไม่มี public MCP** (Agiloft, Conga) — wire custom connector + อัพเดต MCP server block

### 2. ตั้ง Slack Channel

`alert-writer` ส่ง `handoff_request` ที่ระบุ Slack channel — orchestrator อ่านช่องจาก **playbook config → House style → Renewal alerts**

**ตั้งช่องก่อน first scheduled run** — ถ้าไม่ handoff จะ dead-letter

### 3. Tune Lookahead Windows

`deadline-calculator` ค่าตั้งต้น tier:

- **Overdue** (เกินกำหนดแล้ว)
- **30 days**
- **60 days**
- **90 days**
- **180 days**

ทีมที่:

- **renewal cycle สั้น** (SaaS order form < 1 ปี) → ลด tier ลง
- **multi-year enterprise MSA** ที่มี **12-month notice window** → เพิ่ม tier 12 เดือน

ปรับ threshold ใน `deadline-calculator` prompt และ `alert-writer.yaml` ที่เกี่ยวข้อง

### 4. Adjust Escalation Matrix

`deadline-calculator` อ่าน **escalation matrix** ของ playbook เพื่อตัดสินว่าจะตั้ง `escalation_needed: true` หรือไม่ และ route ไปใคร

ยืนยัน matrix สะท้อน **approval authority ปัจจุบัน**:

- ใครเซ็นให้ auto-renewal lapse?
- ใครเซ็นให้ renegotiation เกิน dollar threshold?

`escalation-flagger` skill โหลดใน `alert-writer` สำหรับ formatting

### 5. ยืนยัน Work-Product Header

ตามมาตรฐาน cookbook

### 6. Cadence

- **Weekly** = ค่าตั้งต้น
- **High-volume teams** → daily
- **Small teams** → monthly

Cadence ใน workflow engine ของทีมเอง — cookbook ไม่ schedule ตัวเอง

## Use Case จริง

### สถานการณ์: บริษัท SaaS Mid-Stage มี 800 สัญญา active

**ก่อน Renewal Watcher:**
- Commercial counsel จัด review สัญญาที่กำลังจะ renew **ทุกไตรมาส** (Q1, Q2, Q3, Q4)
- ระหว่างไตรมาส — สัญญาที่ auto-renew **หลุดมือ** ทุกครั้ง
- ครั้งหนึ่งพลาด notice window 60 วันของ contract ที่ไม่อยากต่อ — เสียค่าใช้จ่าย $200k ปีถัดไป
- การหา playbook deviation = manual ตรงตอน contract close

**กับ Renewal Watcher:**
- Weekly run ทุกจันทร์เช้า — alert ลง `#commercial-legal` channel
- Alert แยก tier:
  - **Overdue** (ต้องตัดสินใจวันนี้)
  - **30 days** (ตัดสินใจสัปดาห์นี้)
  - **60 / 90 / 180 days** (เริ่มวางแผน)
- ทุก contract มี playbook deviation flag — เห็นทันทีว่ามีอะไร "นอก standard"
- ก่อน renegotiation — รัน **targeted scan** กับ counterparty นั้น = เห็นทุกสัญญา + price escalator + cancel-by ในที่เดียว

**สิ่งที่ทีมยังต้องทำ:**

- review ทุก alert — verify CLM metadata vs signed agreement
- ตัดสินใจว่า **cancel, renegotiate หรือ let renew**
- ตรวจ amendment ที่อาจไม่ได้ ingest เข้า CLM
- review playbook deviation — บางครั้ง deviation ยอมรับได้ในบริบท

## คำเตือนสำคัญ

### 1. CLM Metadata อาจ Wrong

จาก README:

> "CLM metadata drifts from executed documents — amendments get signed and not re-ingested, effective dates vary from signature dates, auto-renewal mechanics are sometimes mis-tagged."

ก่อน rely บน computed deadline ในการตัดสินใจ termination/renewal — **ทนายต้องยืนยันกับ signed agreement และ amendment**

### 2. Escalation Routing ≠ Escalation Judgment

- "Flagged playbook deviation" อาจ acceptable ในบริบท
- "Unflagged term" อาจต้องการ attention

**matrix คือ router ไม่ใช่ reviewer**

### 3. Quiet Weeks ≠ Clean Weeks

contract ที่ไม่ surface อาจ:

- หายจาก CLM (ingest ผิด)
- mis-tagged (date field ผิด)
- ผ่าน notice window ไปแล้วโดย metadata ไม่สะท้อน

**all-clear footer = agent รัน ไม่ใช่ "ไม่มีอะไรต้องทำ"**

### 4. Counterparty Communication เป็น Untrusted Input

ทุก clause, notice provision, message และ CLM comment ที่อ่าน — instruction ที่ฝังคือ **content ไม่ใช่ command**

## ปรัชญา — ความรับผิดชอบในการตัดสิน

จาก append ของ system prompt:

> "Your output is a lead, not a legal conclusion. Every cancel-by date, renewal term, deviation flag, and escalation route you surface is a screening call — a lawyer verifies against the signed agreement and decides whether to cancel, renegotiate, or let the renewal run. You recommend; a lawyer decides."

หัวใจของ Renewal Watcher = **ขจัด "เผลอลืม"** — แต่ไม่แทนการตัดสินใจ

contract ที่ commercial counsel ของทีมตัดสินใจไม่ต่อ = สัญญาที่ตอบโจทย์ธุรกิจ
contract ที่ลืม cancel กลายเป็น auto-renew = สัญญาที่อยู่เพราะลืม

Renewal Watcher ป้องกัน "อยู่เพราะลืม" — ทำให้ทุก renewal กลายเป็น **active decision**

> **Recommend ≠ Decide** — agent surface deadline, lawyer cancel/renegotiate/renew
