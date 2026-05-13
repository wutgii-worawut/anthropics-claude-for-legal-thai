---
title: 05 Cookbooks
nav_order: 6
has_children: true
permalink: /05-cookbooks/
---

# 05 Managed Agent Cookbooks — สูตรอาหารสำหรับ Agent ที่ทำงานเอง

> "Every agent in this repo ships two ways: as a Claude Code plugin you install today, and as a Claude Managed Agent template your platform team deploys behind your own workflow engine. Same agent, same skills — pick your surface."
> — `managed-agent-cookbooks/README.md`

## Managed Agent คืออะไร

ตลอด `claude-for-legal` plugin ที่เห็นในหมวด `03-plugins` ทำงานในแบบ **interactive** — ผู้ใช้เปิด Claude Code, พิมพ์คำสั่ง, agent ทำงาน, ผลลัพธ์แสดงในหน้าต่างเดียวกัน

แต่งานบางอย่างไม่ควรรอให้ผู้ใช้เปิด Claude — เช่น:

- **เฝ้าศาล (court docket)** — สำนวนใหม่อาจเข้าตอนกลางคืน ต้องประมวลทันที
- **เฝ้ากฎใหม่ (regulatory feed)** — Federal Register ออกประกาศใหม่ทุกวันทำการ
- **เฝ้าวันต่อสัญญา (renewal deadlines)** — สัญญาบางฉบับต้องบอกเลิกล่วงหน้า 90 วัน ห้ามลืม

สำหรับงานแบบนี้ Anthropic มี **Managed Agent** — agent ที่:

1. **รันบนคลาวด์ของ Anthropic** (ไม่ใช่บนเครื่องผู้ใช้)
2. **ถูกกระตุ้นโดย event** (จาก scheduler, webhook, หรือ message queue) ไม่ใช่จากผู้ใช้พิมพ์
3. **ทำงานในแบบ headless** — ไม่มีหน้าจอตอบโต้ ผลลัพธ์เป็นไฟล์หรือ message ที่ส่งต่อ
4. **ถูกตั้งค่าผ่าน API endpoint** `POST /v1/agents` ของ Anthropic

ตัว Managed Agent เองเป็น **product** ของ Anthropic (ยังเป็น research preview ในตอนนี้) แต่การจะ deploy agent ใด ๆ ก็ตามต้องมี **manifest** ที่บอกว่า:

- system prompt ของ agent คืออะไร
- ใช้ skill ตัวไหนได้
- เรียก MCP server ใด
- delegate งานไป sub-agent ใด

## ทำไมแยกเป็น cookbooks

ภายในโฟลเดอร์ `managed-agent-cookbooks/` มี **5 ตัวอย่าง agent** ที่ Anthropic เขียนไว้ให้ทีมกฎหมายเอาไป "ปรุงต่อ" — ทุกตัวใช้ **system prompt และ skill เดียวกับ plugin ในหมวด 03** แต่บรรจุใหม่ในรูปแบบ manifest สำหรับ Managed Agent API

ผู้เขียน README ของหมวดนี้ใช้คำว่า "**cookbook, not product**" — เป็นสูตรอาหาร ไม่ใช่อาหารพร้อมเสิร์ฟ:

> "These are cookbooks, not products. They are starting points. Adapt them to your document management system, your contract repository, your Slack workspace, your notification routing, your review cadence. They will not work out of the box without that adaptation, and they are not supposed to."

แปลว่า — Anthropic ตั้งใจให้:

- **ใช้ทันทีไม่ได้** — เพราะแต่ละทีมมีระบบเก็บเอกสารต่างกัน, ช่อง Slack ต่างกัน, ตารางเวลาต่างกัน
- **ต้อง customize** — ชี้ MCP ไปที่ระบบจริงของทีม, ปรับเกณฑ์ความสำคัญ (materiality threshold), ตั้งตารางการรัน
- **เป็นจุดเริ่มต้น** — ทีมไม่ต้องเริ่มจากศูนย์ มีโครงที่ผ่านการคิดเรื่อง security และ workflow มาแล้ว

## Single Source of Truth — Same Agent, Two Surfaces

หลักการสำคัญของ cookbooks คือ **"same agent, same skills — pick your surface"**

แต่ละ cookbook directory **ไม่ได้คัดลอก** system prompt หรือ skill มาเก็บไว้ใหม่ — แต่ใช้ syntax พิเศษใน `agent.yaml` เพื่ออ้างไปยัง plugin ตัวจริงในหมวด `03-plugins`:

```yaml
system:
  file: ../../litigation-legal/agents/docket-watcher.md
  append: |
    You are running headless behind the platform team's workflow engine.
    ...

skills:
  - { from_plugin: ../../litigation-legal }
```

`deploy-managed-agent.sh` script จะอ่าน `agent.yaml`, inline เนื้อหา system prompt จาก plugin, อัปโหลด skill ทุกตัว แล้วเรียก `POST /v1/agents` พร้อม config ที่ resolved แล้ว

ผลลัพธ์: agent ตัวเดียวกัน ทำงานในสองที่:

- **Claude Code plugin** — สำหรับใช้แบบ interactive ในขณะนั่งทำงาน
- **Managed Agent cookbook** — สำหรับรันเป็นงาน scheduled หรือ event-driven

## ความปลอดภัย — 3-Tier Worker Model

cookbook ทุกตัวใช้ **โมเดลความปลอดภัย 3 ชั้น** — เพราะเอกสารทางกฎหมาย (สัญญา, สำนวน, คำให้การ, ข้อร้องเรียน, จดหมาย) ทั้งหมดถือเป็น **untrusted input** — มีโอกาสที่จะมีข้อความฝัง prompt injection เพื่อหลอก agent

| ชั้น | สัมผัสเอกสาร? | Tools ที่ใช้ได้ | MCP |
|---|---|---|---|
| **Readers** (อ่าน) | ใช่ — read-only | `Read`, `Grep` เท่านั้น | อ่านอย่างเดียว |
| **Analyzers** (วิเคราะห์) | ไม่ — เห็นเฉพาะ JSON ที่ผ่านการตรวจสอบ | `Read`, `Grep`, `Glob`, `Agent` | อ่านอย่างเดียว (เฉพาะ verification) |
| **Writers** (เขียน) | ไม่ | `Read`, `Write`, `Edit` | ไม่มี |

**Readers** เป็นด่านที่อ่านเอกสารจริง — แต่จำกัดให้ใช้แค่ `Read`/`Grep` ไม่มี Write ไม่มี MCP ไม่มี network ผลลัพธ์ที่ส่งออกต้องเป็น **JSON ที่ผ่าน schema validation** และจำกัดความยาว

**Analyzers** รับ JSON ที่ Reader ส่งมา ทำการวิเคราะห์ตามกฎที่ผู้ใช้ตั้งไว้ ไม่เห็นเอกสารดิบ

**Writers** เป็นชั้นเดียวที่มี `Write` แต่ไม่เคยเห็นเอกสารดิบ ผลิตผลลัพธ์สุดท้ายเป็นไฟล์

หลักการ: **Write ห้ามอยู่ในมือเดียวกับ Read เอกสารดิบ** — ป้องกัน prompt injection ที่อาจสั่งให้เขียนสิ่งที่ผู้ใช้ไม่ต้องการ

## Closed-Schema Handoff ผ่าน Orchestrator

agent ใน cookbook **ไม่เรียกกันโดยตรง** — เมื่อ `launch-radar` พบ launch ที่ต้องการ legal review เต็มรูปแบบ มันไม่เรียก `launch-review` skill เอง แต่ส่ง **`handoff_request`** ใน output

มี `scripts/orchestrate.py` (หรือ event bus ของทีม) คอย:

1. รับ `handoff_request` จาก agent ตัวแรก
2. ตรวจสอบว่า target อยู่ใน **allowlist** หรือไม่
3. ตรวจสอบว่า payload ตรงกับ **schema** ที่กำหนดไว้
4. กระตุ้น agent ตัวที่ 2 ผ่าน new steering event

นี่คือการแยก **routing** ออกจาก **handling** — ไม่ให้ agent ตัวใดตัวหนึ่งมีอำนาจเรียกอะไรก็ได้ตามใจชอบ

> **Research preview limit:** Managed Agent ในตอนนี้รองรับ delegation แค่ **1 ระดับ** — orchestrator เรียก worker ได้ แต่ worker เรียก subagent ต่อไม่ได้

## รายการ Cookbook ทั้ง 5

| Cookbook | Plugin ต้นทาง | เฝ้าอะไร | Workers (Bold = ตัวเดียวที่มี Write) |
|---|---|---|---|
| [reg-monitor](reg-monitor.html) | regulatory-legal | Federal Register, agency RSS, TR | feed-reader · materiality-filter · **digest-writer** |
| [renewal-watcher](renewal-watcher.html) | commercial-legal | คลังสัญญา (Ironclad) — วันต่อสัญญา / cancel-by | repo-reader · deadline-calculator · **alert-writer** |
| [diligence-grid](diligence-grid.html) | corporate-legal | Virtual Data Room (Box, Datasite, iManage) | doc-reader · extractor · normalizer · **grid-writer** |
| [launch-radar](launch-radar.html) | product-legal | ระบบติดตามผลิตภัณฑ์ (Jira, Linear, Asana) | tracker-reader · risk-classifier · **memo-writer** |
| [docket-watcher](docket-watcher.html) | litigation-legal | Court dockets (Trellis, CourtListener) | docket-reader · deadline-mapper · **tracker-writer** |

## สิ่งที่ Cookbook ให้คุณและไม่ให้

### คุณได้:

- โครง **manifest** ที่ใช้งานได้จริง
- **reference architecture** ที่ผ่านการคิดเรื่องความปลอดภัย
- **skill** ที่พิสูจน์แล้วใน plugin ของ Claude Code
- **steering-event examples** สำหรับทดสอบ

### คุณไม่ได้:

- agent ที่ใช้ใน production ได้ทันที — ต้อง wire MCP กับระบบจริง, ตั้ง cadence, ตั้ง notification routing, ปรับ prompt และทำ evaluation ของตัวเอง
- **การทดแทนทนายความ** — agent เหล่านี้ monitor, extract และ draft เท่านั้น ทนายเป็นคนตัดสิน

## ก่อน Deploy — Work Product และความลับทางวิชาชีพ

ทุก agent ใน cookbook สร้าง **attorney work product** ในงานจริง — ผลลัพธ์ของ agent อยู่ภายใต้ **work-product privilege** (ความลับทางวิชาชีพ) ตามกฎหมายอเมริกัน

ทุก manifest มี append ที่สั่งให้ agent นำ **work-product header** จากการตั้งค่าของ plugin มาใส่ที่ต้นไฟล์ — ทีมต้องยืนยัน header นี้กับทีมกฎหมายภายในก่อน deploy เพราะภาษา header แตกต่างกันตาม:

- เขตอำนาจศาล (jurisdiction)
- บทบาทผู้ใช้ (lawyer vs non-lawyer)
- ประเภทของเอกสาร

หาก deployment ของคุณประมวลข้อมูลที่ **ไม่ควรเก็บ** — ทบทวน **data retention settings** ของ Anthropic และของทีมเองก่อนเปิดใช้งาน

## วิธี Deploy

แต่ละ cookbook มีคำสั่ง deploy ที่เหมือนกัน:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export <MCP_URL_1>=...
export <MCP_URL_2>=...
../../scripts/deploy-managed-agent.sh <cookbook-slug>
```

script จะ:

1. อ่าน `agent.yaml`
2. Resolve `system.file` → inline ข้อความ system prompt
3. Resolve `skills.from_plugin` → อัปโหลด skill ทุกตัวภายใต้ plugin นั้น
4. Resolve `callable_agents.manifest` → สร้าง sub-agent ทุกตัวก่อน
5. `POST /v1/agents` พร้อม config ที่ resolved แล้ว

รายละเอียดเพิ่มเติมดูในหมวด **08-deployment** หัวข้อ managed-agents

---

> **อ่านต่อ:** เลือก cookbook ที่ตรงกับงานทีมกฎหมายของคุณ — แต่ละไฟล์อธิบาย workflow, security model, และจุดที่ต้อง customize ก่อน deploy
