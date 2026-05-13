---
title: Claude for Legal คืออะไร
parent: 01 ภาพรวม
nav_order: 1
---

# Claude for Legal คืออะไร

## คำตอบสั้นที่สุด

**Claude for Legal คือชุด plugin อ้างอิง (reference plugins) สำหรับงานกฎหมาย** ที่ Anthropic เผยแพร่ผ่าน repository ชื่อ [`anthropics/claude-for-legal`](https://github.com/anthropics/claude-for-legal) ภายใต้สัญญาอนุญาต Apache 2.0

ในเชิงโครงสร้าง — ไม่ใช่ application ที่รันได้ทันที แต่เป็น **library ของไฟล์ markdown และ JSON** ที่ผู้ใช้นำไป "ลง" บน Claude (ผ่าน Cowork, Claude Code, หรือ Managed Agents API) เพื่อให้ Claude ทำหน้าที่เป็น "ผู้ช่วยทนายความ" เฉพาะสายงานได้ — สำนวนการสัญญา (commercial), การควบรวมกิจการ (M&A), ความเป็นส่วนตัวของข้อมูล (privacy), การฟ้องคดี (litigation), ทรัพย์สินทางปัญญา (IP) และอื่น ๆ

## คำตอบยาวขึ้น — แยกตามนิยามของศัพท์แต่ละคำ

ก่อนเข้าใจ Claude for Legal ต้องเข้าใจศัพท์ใน ecosystem ของ Claude เสียก่อน

### Plugin (ปลั๊กอิน)

**Plugin** ในที่นี้คือ **"bundle เฉพาะสายงาน"** — directory เดียวที่บรรจุทุกอย่างที่ Claude ต้องใช้เพื่อทำงานในสายนั้น

โครงสร้างภายในแต่ละ plugin

```
<plugin>/
├── .claude-plugin/plugin.json     # metadata: ชื่อ, รุ่น, คำอธิบาย
├── .mcp.json                      # รายการ MCP server ที่ใช้
├── CLAUDE.md                      # template practice profile
├── README.md                      # คู่มือผู้ใช้ของ plugin
├── skills/                        # skill ทุก skill อยู่ในนี้
│   └── <skill-name>/SKILL.md
├── agents/                        # scheduled agent (ถ้ามี)
└── hooks/                         # pre/post-tool hooks (ถ้ามี)
```

ใน repository นี้มี plugin ทั้งหมด **13 ตัว** (12 ของ Anthropic เอง + 1 ของ Thomson Reuters)

| Plugin | สายงาน | ภาษาไทย |
|--------|--------|---------|
| `commercial-legal` | สัญญาเชิงพาณิชย์ | งานนิติกรรมเชิงธุรกิจ — ทบทวน MSA, NDA, SaaS, ติดตามวันต่ออายุ |
| `corporate-legal` | M&A และนิติกรรมบริษัท | ตรวจสอบสถานะกิจการ (due diligence), เอกสาร closing, มติคณะกรรมการ |
| `privacy-legal` | กฎหมายคุ้มครองข้อมูล | DPA review, DSAR response, PIA generation |
| `product-legal` | กฎหมายผลิตภัณฑ์ | launch review, marketing claims, "is this a problem?" |
| `employment-legal` | กฎหมายแรงงาน | จ้างใหม่, เลิกจ้าง, การจำแนกลูกจ้าง, การลา, สอบสวนภายใน |
| `regulatory-legal` | กฎหมายควบคุม | ติดตาม regulatory feed, diff นโยบาย, NPRM comment |
| `ai-governance-legal` | ธรรมาภิบาล AI | AI use-case triage, AIA, vendor AI review |
| `ip-legal` | ทรัพย์สินทางปัญญา | trademark clearance, FTO, DMCA, OSS compliance |
| `litigation-legal` | งานคดีและการฟ้องร้อง | matter intake, claim chart, deposition prep |
| `legal-clinic` | คลินิกกฎหมายมหาวิทยาลัย | นักศึกษากฎหมาย, advisor, ABA Op. 512 |
| `law-student` | นักศึกษากฎหมาย | Socratic drill, IRAC, bar prep, flashcards |
| `legal-builder-hub` | hub สำหรับ community skill | trust-gated install, QA framework |
| `cocounsel-legal` | partner plugin (Thomson Reuters) | Westlaw Deep Research |

### Marketplace (ตลาด plugin)

**Marketplace** คือไฟล์ JSON หนึ่งไฟล์ ([`.claude-plugin/marketplace.json`](https://github.com/anthropics/claude-for-legal/blob/main/.claude-plugin/marketplace.json)) ซึ่งเป็น **catalog** บอกว่าใน repository นี้มี plugin อะไรบ้าง ชื่ออะไร อยู่ใน path ใด

เมื่อผู้ใช้รัน `/plugin marketplace add <path-to-repo>` ใน Claude Code, Claude จะอ่านไฟล์ marketplace.json นี้แล้วจัดทำรายการ plugin ที่ติดตั้งได้ ผู้ใช้ติดตั้งทีละตัวด้วย `/plugin install <plugin-name>@claude-for-legal`

ตัวอย่างจาก marketplace.json จริง

```json
{
  "name": "claude-for-legal",
  "owner": { "name": "Anthropic" },
  "plugins": [
    {
      "name": "commercial-legal",
      "source": "./commercial-legal",
      "description": "Reviews vendor agreements, NDAs, and SaaS subscriptions..."
    },
    ...
  ]
}
```

### Skill (สกิล)

**Skill** คือ **"workflow เดี่ยว"** ที่ Claude เรียกใช้ได้ — เป็นไฟล์ markdown หนึ่งไฟล์ที่บรรยายขั้นตอนการทำงานอย่างเป็นลำดับ พร้อม frontmatter (YAML) ระบุ metadata

โครงสร้างไฟล์ skill

```markdown
---
name: vendor-agreement-review
description: >
  Review a vendor MSA against the practice playbook
  and produce a redline memo. Used when an MSA is uploaded
  or referenced.
argument-hint: "[file-path]"
user-invocable: true
---

# /commercial-legal:vendor-agreement-review

## Purpose
...

## Instructions
1. โหลด playbook จาก CLAUDE.md
2. ระบุประเภทของสัญญา (MSA, NDA, SaaS)
3. สแกนข้อสัญญาทีละข้อ เทียบกับ playbook
...
```

ผู้ใช้เรียกใช้ skill ผ่าน slash command: `/commercial-legal:review`, `/privacy-legal:dsar-response`, `/litigation-legal:claim-chart` เป็นต้น

ใน repository นี้มี skill ทั้งหมดประมาณ **151 skills** กระจายอยู่ใน 13 plugin

### Agent (เอเจนต์)

**Agent** ใน Claude for Legal คือ **scheduled workflow** — ไม่เหมือน skill ที่ผู้ใช้ต้องสั่ง agent ทำงานตามตารางเวลา (cron-like) อ่าน practice profile แล้วเขียนรายงานหรือส่งข้อความเอง

ตัวอย่าง

| Agent | ทำอะไร | ความถี่ |
|-------|--------|--------|
| `renewal-watcher` | สแกน register สัญญา หาที่ใกล้ครบกำหนด cancel-by | รายสัปดาห์ |
| `docket-watcher` | ติดตาม court docket หาคำสั่งใหม่และ deadline | รายวัน |
| `deal-debrief` | สรุปสัญญาที่ลงนามใหม่ที่มี deviation จาก playbook | รายสัปดาห์ |
| `launch-watcher` | ดูตารางเปิดตัวผลิตภัณฑ์ที่ต้องการ legal review | รายสัปดาห์ |
| `reg-change-monitor` | ตรวจสอบ feed กฎหมาย หากระทบนโยบายของบริษัท | รายวัน |

Agent มีทั้งหมด **10 ตัว** ใน repository นี้

### Cookbook (managed-agent cookbook)

**Cookbook** คือ **template สำหรับ deploy agent ผ่าน Managed Agents API** ของ Anthropic — ใน directory `managed-agent-cookbooks/` มี 5 ตัว ซึ่งสะท้อน agent ตัวสำคัญ ๆ ของ plugin

แต่ละ cookbook มีโครงสร้าง

```
managed-agent-cookbooks/renewal-watcher/
├── agent.yaml              # manifest สำหรับ API deploy
├── subagents/              # leaf-worker subagents
│   ├── repo-reader.yaml
│   ├── deadline-calculator.yaml
│   └── alert-writer.yaml
├── steering-examples.json
└── README.md
```

Cookbook คือสิ่งที่ทำให้ "agent เดียวกัน" รันได้ทั้งใน Cowork (สำหรับผู้ใช้คนเดียว) และในระบบหลังบ้านของบริษัท (เป็น Managed Agent ที่รันใน cron job ของตัวเอง)

### MCP (Model Context Protocol)

**MCP** คือ protocol มาตรฐานที่ Anthropic เผยแพร่ ให้ Claude ติดต่อกับระบบภายนอก — เปรียบเหมือน USB-C ของ LLM tool use

Claude for Legal มี MCP connector ที่ "ต่อ" Claude เข้ากับ

- **ทั่วไป**: Slack, Google Drive, Box
- **CLM**: Ironclad, DocuSign CLM
- **DMS**: iManage
- **E-discovery**: Everlaw
- **คดีความ**: CourtListener (federal docket), Trellis (state docket), Aurora
- **งานวิจัยกฎหมาย**: Descrybe, CoCounsel (Westlaw), Solve Intelligence
- **ตามรอย Launch**: Linear, Jira, Asana

แต่ละ plugin ระบุ MCP ที่ใช้ใน `.mcp.json` ของ plugin นั้น เมื่อ MCP ใดไม่ได้ตั้งค่า skill จะ degrade อย่างนุ่มนวล — ทำงานต่อได้แต่ skip ส่วนที่ต้องใช้ connector นั้น

### Hook

**Hook** คือ pre/post tool interception ที่ plugin บางตัวใช้ — เช่น redact PII ก่อนส่งข้อมูลไปยัง MCP, หรือเพิ่ม source tag ลงในผลลัพธ์ที่ได้กลับมา

มี plugin ไม่กี่ตัวที่ใช้ hook ส่วนใหญ่ทำงานได้โดยไม่ต้องมี hook

### Practice Profile (CLAUDE.md)

**Practice Profile** เป็นหัวใจของ Claude for Legal — ไฟล์ markdown ใน path

```
~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md
```

ที่บันทึก **playbook ของสำนักงานผู้ใช้** เป็นภาษาคน (plain English) — ไม่ใช่ YAML config — เช่น

```markdown
## Sales-side playbook
- Liability cap: 12 months fees, no carve-outs
- Indemnity: mutual, IP from us, data from them
- Termination: 30 days notice for convenience

## Escalation Matrix
| Issue | Threshold | Approver |
|-------|-----------|----------|
| Uncapped liability | any | GC |
| IP assignment to vendor | any | CTO + GC |
```

ทุก skill ใน plugin **อ่าน CLAUDE.md ก่อนทำงานเสมอ** — เป็นกลไก personalization หลักของระบบ ผู้ใช้แก้ไขไฟล์นี้ได้ตรง ๆ หรือสั่ง re-run `cold-start-interview` ใหม่

## Two Runtimes, One Source

Anthropic ออกแบบให้ทุก skill และ agent ใช้ได้ **2 ทาง จาก source เดียว**

1. **ลงเป็น plugin** ใน Claude Cowork หรือ Claude Code — ผู้ใช้คนเดียว, interactive
2. **deploy ผ่าน Managed Agents API** — รันหลังบ้านของบริษัท, scheduled, headless

ทั้ง 2 ทางใช้ system prompt และ skill ชุดเดียวกัน — ต่างกันแค่ "ที่รัน" หลักการนี้ทำให้สำนักงานสามารถใช้งานแบบ pilot ผ่าน Cowork ก่อน แล้ว scale ขึ้นเป็น production ได้โดยไม่ต้องเขียน prompt ใหม่

## เอกสารถัดไป

- [กลุ่มผู้ใช้เป้าหมาย](./who-is-it-for.html) — Claude for Legal เหมาะกับใคร?
- [สถาปัตยกรรมระบบ](./architecture.html) — โครงสร้างภายในเป็นอย่างไร?

← กลับไป [01 ภาพรวม](./)
