---
title: Launch Radar
parent: 05 Cookbooks
nav_order: 3
---

# Launch Radar — เฝ้าการปล่อยผลิตภัณฑ์ที่ต้องตรวจสอบทางกฎหมาย

> "The radar triage is a routing decision, not a legal review."
> — `managed-agent-cookbooks/launch-radar/README.md`

## จุดประสงค์

Launch Radar คือ cookbook สำหรับทีม **product counsel** (ทนายที่ดูแลผลิตภัณฑ์) — ทำงานเฝ้า **launch tracker** ของทีมผลิตภัณฑ์เพื่อหาการเปิดตัวที่ "น่าจะต้องการการตรวจสอบทางกฎหมาย" ในอีกไม่กี่สัปดาห์

ระบบติดตามที่รองรับ:

- **Jira** (Atlassian)
- **Linear**
- **Asana**

agent จะ **triage** แต่ละ launch ตาม **risk calibration** ของ product counsel แล้วผลิต **weekly radar memo** สรุปว่า:

- อะไรกำลังมา
- อะไรต้องการ legal attention
- อะไรกระตุ้น flag

## โครงสร้างไฟล์ใน cookbook

```text
managed-agent-cookbooks/launch-radar/
├── README.md                       # 6.3 KB — คู่มือ
├── agent.yaml                      # 2.0 KB — manifest หลัก
├── steering-examples.json          # 489 B  — ตัวอย่าง steering event
└── subagents/
    ├── tracker-reader.yaml         # 3.0 KB — อ่าน tracker
    ├── risk-classifier.yaml        # 2.7 KB — จัดประเภทความเสี่ยง
    └── memo-writer.yaml            # 4.4 KB — เขียน memo (ตัวเดียวที่มี Write)
```

## Plugin ต้นทาง

cookbook นี้ใช้ system prompt และ skill จาก plugin [`product-legal`](../03-plugins/) — โดยเฉพาะ agent `launch-watcher.md`

```yaml
system:
  file: ../../product-legal/agents/launch-watcher.md
  append: |
    You are running headless...
```

## โมเดลความปลอดภัย 3-Tier

ticket ใน tracker เป็น **untrusted input**:

- **PM ใส่อะไรก็ได้** ใน title, description, comment
- **Attacker อาจสร้าง ticket** ในระบบที่ภายในมีการเข้าถึงเปิด
- การ triage **routing ตามเนื้อหา** — agent ไม่รับรองว่า ticket จริง

| ชั้น | สัมผัส tracker content | Tools | MCP |
|---|---|---|---|
| **`tracker-reader`** | ใช่ | `Read`, `Grep` เท่านั้น | Linear, Jira, Asana (อ่าน) |
| `risk-classifier` / Orchestrator | ไม่ | `Read`, `Grep`, `Glob`, `WebFetch`, `Agent` | Orchestrator: Linear / Jira / Asana / Drive (อ่าน) |
| **`memo-writer`** (Write-holder) | ไม่ | `Read`, `Write`, `Edit` | ไม่มี |

`tracker-reader` คืน schema-validated JSON list ของ launch `risk-classifier` **ไม่มี MCP, ไม่มี network** — ทำงานจาก validated list + calibration file ของผู้ใช้

`memo-writer` ผลิต `./out/launch-radar-<date>.md` orchestrator ไม่มี Write และไม่อ่าน ticket body ดิบเอง

## Handoff — เมื่อ Launch ต้องการ Review เต็มรูปแบบ

เมื่อ launch ที่ต้องการ **full legal review memo** (ไม่ใช่แค่ radar entry):

- orchestrator **ไม่ draft memo ตรง** ใน session นี้
- ส่ง `handoff_request` สำหรับ **`launch-review` skill** ที่จะรันใน fresh session
- `scripts/orchestrate.py` route

นี่ทำให้ "การ triage" (เร็ว, batch) แยกออกจาก "การ review เต็ม" (ลึก, ใช้เวลา) — ตามหลัก separation of concerns

## Steering Events ตัวอย่าง

```json
[
  {
    "event": "Scan tracker for launches in next 6 weeks",
    "description": "Weekly radar — default horizon, all surfaces"
  },
  {
    "event": "Triage: new Agent autonomy feature (PROJ-1234), is this a problem?",
    "description": "Single-ticket triage against calibration table — runs the is-this-a-problem path end-to-end"
  },
  {
    "event": "Re-scan after roadmap reprioritization, horizon: 4 weeks",
    "description": "Off-cadence refresh after a planning meeting shuffled launch dates"
  }
]
```

ตัวที่สองคือ **on-demand single-ticket triage** — เมื่อ PM ping product counsel ว่า "เรื่องนี้มีปัญหาไหม?" — แทนที่จะรอ weekly run

## วิธี Deploy

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export LINEAR_MCP_URL=...
export ATLASSIAN_MCP_URL=...
export ASANA_MCP_URL=...
export GDRIVE_MCP_URL=...
../../scripts/deploy-managed-agent.sh launch-radar
```

ตั้งเฉพาะ MCP URL ที่ทีมใช้จริง — `tracker-reader` ข้าม MCP ที่ไม่ได้ configure

รายละเอียดเต็มในหมวด **08-deployment** หัวข้อ managed-agents

## ต้องปรับอะไรก่อน Deploy

### 1. Tracker Pointer

แก้ `mcp_servers` ใน `agent.yaml` และ `subagents/tracker-reader.yaml` ให้ตรงกับ MCP URL ของ tracker — ถ้าใช้ tool เดียว ลบสอง MCP ที่ไม่ใช้ ถ้า tracker ของทีมไม่อยู่ในรายการ swap MCP ที่ใช้แล้วอัพเดต `tracker-reader` system prompt

### 2. Risk Calibration

`risk-classifier` อ่าน calibration ของผู้ใช้จาก `../../product-legal/CLAUDE.md` (populated โดย `/product-legal:cold-start-interview`)

ถ้ายังไม่ได้รัน cold-start:

- รัน cold-start interview ก่อน
- หรือเขียน `CLAUDE.md` ด้วยมือพร้อมตาราง:
  - **Usually blocks** — ประเภท launch ที่มักจะถูก block
  - **Usually requires work** — ที่มักจะต้องการ legal work
  - **Usually FYI** — ที่ปกติแค่บอกให้รู้

**ถ้าไม่มี calibration** — classifier fallback ไป keyword trigger เท่านั้น = noisy

### 3. Scan Cadence และ Horizon

- **Cadence** ค่าตั้งต้น = weekly / 6 สัปดาห์
- ทีมที่ launch cadence เร็ว — daily หรือ biweekly
- ทีมที่ lead time สั้น — ขยาย horizon

Cadence อยู่ใน scheduler ของทีม (cron, Temporal, Airflow, EventBridge) — ไม่ใช่ในตัว agent

Horizon ส่งผ่าน steering event

### 4. Delivery Channel

memo ลง `./out/` ค่าตั้งต้น — ถ้าต้องการ Slack:

- **(a)** เพิ่ม Slack MCP ใน cookbook + อัพเดต `memo-writer` ให้ post หลังเขียน
- **(b)** ให้ orchestration layer pick up `./out/launch-radar-<date>.md` แล้ว forward

Anthropic แนะนำ **(b)** — เก็บ delivery ออกจาก agent ทำให้ testing ง่าย

### 5. Trigger Keywords

keyword list ใน `launch-watcher` system prompt **มีความเห็น** (COPPA, HIPAA, AI vendor names ฯลฯ):

- ลบ category ที่ไม่เกี่ยวกับสินค้าทีม
- เพิ่ม domain-specific term (FedRAMP, PCI, HITRUST, TCPA, biometrics)
- ปรับ severity threshold ตาม calibration table

### 6. Privilege Header

`memo-writer` prepend work-product header จาก plugin config — ยืนยัน marking กับ GC (general counsel) ก่อน deploy

## Use Case จริง

### สถานการณ์: ทีม product ของบริษัท SaaS มี roadmap 40 launch ใน 6 เดือน

**ก่อน Launch Radar:**
- Product counsel ต้องนั่งใน planning meeting ทุกครั้งเพื่อรู้ว่าอะไรกำลังมา
- หรือไม่ก็ตามไปอ่าน Jira backlog เอง = พลาดเรื่องสำคัญบ่อย
- เมื่อ PM ping ว่า "ฟีเจอร์นี้มี legal issue ไหม?" — counsel ใช้เวลา 1-2 ชั่วโมงต่อ ticket

**กับ Launch Radar:**
- Weekly run ทุกจันทร์เช้า — radar memo ลง `#legal-product` channel
- Counsel เห็น launch ที่ "needs review" ในสัปดาห์นี้พร้อม:
  - Trigger keyword ที่ match (COPPA, AI vendor, biometric)
  - calibration tier (usually blocks / usually requires work)
  - ticket URL กลับไป
- เมื่อ PM ping — single-ticket triage รันใน 1-2 นาที = answer ทันที

**สิ่งที่ counsel ยังต้องทำ:**

- review ทุก "needs review" item — ไม่ trust label
- ดู "FYI" items ด้วย (ไม่ใช่แค่ที่ flag)
- หาก launch ต้องการ memo เต็ม — กระตุ้น `launch-review` ผ่าน handoff

## คำเตือนสำคัญ

### 1. Triage ≠ Legal Review

- "Needs review" = product counsel ควรดู ไม่ใช่ "มีปัญหาแน่นอน"
- "FYI" = ไม่ได้แปลว่า launch นี้ปลอดภัย
- "Skip" = ไม่ได้แปลว่า launch นี้เคลียร์

### 2. Calibration ที่ Stale → Triage ที่ Stale

ถ้า:

- มีสินค้าสายใหม่ที่ calibration ไม่ครอบคลุม
- regulator ใหม่ที่ไม่อยู่ใน list
- ภูมิภาคใหม่ที่เพิ่งขยายไป
- third-party dependency ใหม่

→ ต้องอัพเดต calibration ก่อน radar จะ route ถูกต้อง

### 3. Trigger Keyword List มีความเห็น

ถ้าสินค้าของทีม:

- biometric-heavy — ต้องเพิ่ม BIPA, CCPA, GDPR biometric terms
- FedRAMP-bound — ต้องเพิ่ม FedRAMP, FISMA, NIST 800-53
- เด็ก-related — ต้องเพิ่ม COPPA, parental consent

ถ้า default ไม่ครอบ — retune ก่อน first run

### 4. Ticket ก็เป็น Untrusted Input

PM ใส่อะไรก็ได้ — และ attacker อาจสร้าง ticket — triage **route ตามเนื้อหา** ไม่รับรองความถูกต้อง

## ปรัชญา — สิ่งที่ Cookbook ไม่ให้

จาก README:

> "You don't get a replacement for the product counsel. This agent triages. A lawyer reviews, flags, decides. Every 'needs review' item in the memo is a lead, not a verdict."

หัวใจของ launch-radar คือ **ทำให้ counsel เห็นภาพ** — ไม่ใช่ **ทำการตัดสินใจแทน counsel**

counsel ที่ใช้ tool นี้ดี = counsel ที่เห็น launch ตั้งแต่อยู่ใน roadmap ไม่ใช่ตอน PM โทรมาขอ approval หน้างาน

> **Routing decision ไม่ใช่ legal review** — ทุก radar entry คือ lead ไม่ใช่ verdict
