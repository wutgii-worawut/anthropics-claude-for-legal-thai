---
title: Quickstart — เริ่มต้นใช้งาน
parent: 01 ภาพรวม
nav_order: 5
---

# Quickstart — เริ่มต้นใช้งาน

> "60 seconds. This gets you to using your plugins."
> — QUICKSTART.md, anthropics/claude-for-legal

หน้านี้สอนติดตั้งและใช้งาน Claude for Legal ภายใน **60 วินาที** (ส่วนติดตั้ง) บวก **10–20 นาที** (ส่วน cold-start interview ของ plugin แรก) ครอบคลุมทั้ง 2 surface หลัก — **Claude Cowork** (desktop app) และ **Claude Code** (terminal CLI) — พร้อมตัวอย่างคำสั่งจริงและสิ่งที่ต้องระวัง

## ก่อนเริ่ม — เลือก surface ที่เหมาะ

| ท่านมี... | ใช้ surface ใด |
|----------|---------------|
| Mac/Windows + ไม่ค่อยใช้ terminal | **Claude Cowork** (desktop app) |
| Mac/Linux/Windows + ทำงานใน terminal เป็นปกติ | **Claude Code** (CLI) |
| ทีม engineer ในบริษัท ต้องการ deploy ใน CI/CD | **Managed Agents API** (advanced) |

ผู้เริ่มต้นแนะนำ **Claude Cowork** เพราะ UI ง่ายและไม่ต้องตั้งค่า MCP เอง — Cowork จัดการให้

## ทาง A — ติดตั้งใน Claude Cowork

### Step 1: ติดตั้ง Claude Desktop

ดาวน์โหลดที่ [claude.com/download](https://claude.com/download)

ติดตั้งตามขั้นตอนปกติ (next next finish) สำหรับ Mac, Windows

### Step 2: ขอ access Claude Cowork

Claude Cowork เป็น **feature** ใน Claude Desktop ไม่ใช่ app แยก — แต่อาจอยู่ใน beta หรือ requires subscription tier ที่กำหนด ตรวจสอบสถานะของ account ที่ใช้

### Step 3: เปิด Cowork tab → Customize → Browse plugins

ใน Claude Desktop

1. คลิก **Cowork** ใน sidebar
2. คลิก **Customize**
3. คลิก **Browse plugins**
4. หา marketplace ชื่อ `claude-for-legal` (หรือ upload zipped plugin folder เอง)

### Step 4: install plugin ที่ต้องการ

เลือก plugin ตามอาชีพ (อ้างอิงตารางใน [กลุ่มผู้ใช้เป้าหมาย](./who-is-it-for.html))

ตัวอย่าง — ทนายความสัญญาเชิงพาณิชย์ติดตั้ง `commercial-legal`

### Step 5: รัน cold-start

ในหน้าต่าง chat พิมพ์

```
/commercial-legal:cold-start-interview
```

แล้วทำตามขั้นตอนสัมภาษณ์ — เตรียม

- **3–10 signed MSA ที่ผ่านมา** (PDF หรือ Word)
- **playbook ที่บริษัทใช้อยู่** (ถ้ามี)
- **escalation matrix** (ใครต้อง approve อะไร, threshold เท่าไร)
- **house style guide** (ถ้ามี)

ใช้เวลา 10–20 นาที (quick-start mode สามารถลดเหลือ 2 นาที + กลับมา refine ทีหลัง)

## ทาง B — ติดตั้งใน Claude Code

### Step 1: เปิด Claude Code

ใน terminal พิมพ์ `claude` (สมมุติว่าติดตั้ง Claude Code ไว้แล้ว — ดูที่ [claude.com/product/claude-code](https://claude.com/product/claude-code))

### Step 2: clone repo (หรือ download zip)

```bash
git clone https://github.com/anthropics/claude-for-legal.git ~/code/claude-for-legal
```

หรือ download zip จาก GitHub แล้ว unzip

### Step 3: เพิ่ม marketplace

ใน Claude Code session พิมพ์

```
/plugin marketplace add /Users/you/code/claude-for-legal
```

(แทน path ให้ตรงกับเครื่อง — Tip จาก QUICKSTART: พิมพ์ `/plugin marketplace add ` แล้ว **ลาก folder เข้ามาใน terminal** path จะถูกเติมให้)

### Step 4: install plugin

```
/plugin install commercial-legal@claude-for-legal
```

### ⚠️ Step 5: **Restart Claude Code (สำคัญมาก)**

ปิด terminal แล้วเปิดใหม่ — **ข้ามขั้นตอนนี้ไม่ได้** — plugin จะยังไม่ "live" จนกว่าจะ restart

> สาเหตุที่พบบ่อยที่สุดที่ผู้ใช้บอก "command not found" คือไม่ restart

### Step 6: run cold-start

```
/commercial-legal:cold-start-interview
```

ขั้นตอนเหมือนใน Cowork — เตรียม seed document, ตอบคำถาม, ระบบบันทึก profile

### Step 7: เชื่อม research tool

หลัง cold-start — เชื่อม **CourtListener** (free) หรือ research MCP อื่น

ใน Claude Code, MCP ของ plugin อยู่ใน `.mcp.json` ของ plugin แล้ว — ระบบจะ prompt ให้ authorize เมื่อ skill แรกที่ต้องใช้

หากไม่เชื่อม → ทุก citation จะติด tag `[verify]` (ดูหลัก Citation Discipline ใน [หลักการออกแบบ](./principles.html))

## คำเตือนสำคัญ — User Scope vs Project Scope

> "When you run `/plugin install`, you may be asked whether to install for this project only or for all projects (user scope). **Pick user scope.**"
> — QUICKSTART.md

### ทำไมต้อง User Scope

| Scope | ความหมาย | ปัญหา |
|-------|---------|------|
| **Project scope** | plugin ทำงานเฉพาะ folder ปัจจุบัน | **อ่านไฟล์ใน Downloads, Documents, Dropbox ไม่ได้** — งานทนายส่วนใหญ่ต้อง access ไฟล์ข้าม folder |
| **User scope** ✅ | plugin ทำงานจากทุก folder | plugin ไม่ได้สิทธิ์เพิ่ม — ยังต้องชี้ไฟล์เอง — แต่ใช้งานได้จริง |

หากเผลอติดตั้งแบบ project scope แล้ว — แก้

```
/plugin uninstall commercial-legal
cd ~                                # กลับไปที่ home directory
/plugin install commercial-legal@claude-for-legal
# เลือก user scope ตอนถาม
```

## ตัวอย่าง — Workflow แรกหลังติดตั้ง

สมมติติดตั้ง `commercial-legal` แล้ว ทำ cold-start เสร็จ — มี vendor MSA จาก Acme Corp ให้ review

### ขั้นที่ 1: รัน review

```
/commercial-legal:review acme-msa-2026.pdf
```

(หรือลากไฟล์เข้าใน chat แล้วพิมพ์ `/commercial-legal:review`)

### ขั้นที่ 2: Claude ทำงานตาม flow

1. โหลด practice profile จาก `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`
2. ตรวจ contract type — detect ว่าเป็น MSA → route ไปยัง `vendor-agreement-review` sub-skill
3. สแกน clause ทีละข้อเทียบกับ playbook
4. หา deviation
5. (หาก Ironclad connected) ค้น prior agreement กับ Acme Corp
6. สร้าง deviation table + redline
7. (หาก red issue) เรียก `escalation-flagger` หาผู้ approve ที่ถูกต้อง
8. สร้าง stakeholder summary

### ขั้นที่ 3: output

```markdown
# Acme Corp MSA Review
**[AI-ASSISTED DRAFT — requires attorney review]**
*Privileged — Attorney Work Product*

## Summary
- 3 RED deviations from playbook
- 2 YELLOW (negotiable per playbook)
- 12 GREEN (standard, acceptable)

## RED — Escalation Required

| Clause | Issue | Playbook | Acme position | Approver |
|--------|-------|---------|---------------|----------|
| §7.2 Liability | Uncapped indirect damages | 12mo fees, no carve-out | Uncapped IP-indemnity | GC |
| §11 IP | Assignment to Acme | Mutual license, no assignment | Full assignment | CTO + GC |
| §3.4 MFN | Most-favored pricing | We do not offer MFN | MFN required | CFO |

## Redlines
[tracked changes block — open in Word]

## Reviewer Notes
- Citations: clauses cited from contract [verified]; case law from training data [verify]
- Jurisdiction: assumes Delaware governing law (per §15)
- Prior agreements: 2 prior MSAs with Acme found in Ironclad [Ironclad]
  - 2024-03: 12mo cap accepted
  - 2025-08: MFN rejected

## Next steps
- Send to GC for §7.2, §11 approval before counterparty response
- Ready to send redlines? [Y/N]
```

### ขั้นที่ 4: action

- review output ทั้งหมด
- **ตรวจ citation ที่ติด `[verify]` ด้วยตัวเอง**
- หาก approve → ตอบ Y ระบบจะ format redline เป็น .docx tracked changes
- หาก reject → แก้ไข, สั่ง re-run

## Plugin แนะนำสำหรับเริ่มต้น

| ลำดับ | Plugin | ทำไม |
|------|--------|------|
| 1 | `commercial-legal` หรือ `privacy-legal` | workflow ชัดเจน, ผลเห็นเร็ว |
| 2 | `legal-builder-hub` | เปิดประตูไป community skill |
| 3 | plugin ที่ตรงกับงานหลักของท่าน | (อ้างอิง [กลุ่มผู้ใช้เป้าหมาย](./who-is-it-for.html)) |

หลีกเลี่ยงการติดตั้งทุก plugin พร้อมกัน — เริ่ม 1–2 ตัว, ทำ cold-start, ลองใช้จริง, แล้วค่อยขยาย

## ปัญหาที่พบบ่อย (Stuck?)

อ้างอิงจาก QUICKSTART.md

| อาการ | สาเหตุ | วิธีแก้ |
|------|--------|--------|
| "Command not found" หลัง install | ลืม restart Claude Code | ปิด-เปิด Claude Code ใหม่ |
| "Run setup first" | ยังไม่ได้รัน cold-start | รัน `/<plugin>:cold-start-interview` |
| Citation ติด `[verify]` ทุกตัว | ไม่ได้เชื่อม research tool | เชื่อม CourtListener (Settings → Connectors) |
| "I can't read [file]" | plugin เป็น project scope, ไฟล์อยู่นอก folder | uninstall แล้ว install ใหม่ในแบบ user scope |
| Plugin ไม่ทำสิ่งที่ต้องการ | อาจมี community skill ดีกว่า | รัน `/legal-builder-hub:related-skills-surfacer` |

## ทาง C — Managed Agents Deploy (Advanced)

หากท่านเป็น engineer ที่ต้องการ deploy agent ใน production

```bash
# 1. ตั้งค่า API key
export ANTHROPIC_API_KEY=sk-ant-...

# 2. clone repo
git clone https://github.com/anthropics/claude-for-legal.git
cd claude-for-legal

# 3. deploy agent ตัวที่ต้องการ
scripts/deploy-managed-agent.sh renewal-watcher
scripts/deploy-managed-agent.sh docket-watcher
scripts/deploy-managed-agent.sh diligence-grid
```

script จะ

1. อ่าน `managed-agent-cookbooks/<agent>/agent.yaml`
2. resolve file reference (system prompt จาก plugin)
3. upload skill เป็น text blob ไป API
4. สร้าง leaf-worker subagents
5. POST agent manifest ไป `/v1/agents` endpoint

ผูกกับ orchestrator ของท่าน (reference implementation อยู่ใน `scripts/orchestrate.py`)

> **Research Preview**: subagent delegation (`callable_agents`) ยัง preview, รองรับ depth-1 เท่านั้น

## Update Plugin

เมื่อ Anthropic update plugin

```
/plugin update
```

จะตรวจสอบทุก plugin ที่ install และ update ที่จำเป็น **practice profile ของท่านจะอยู่รอด** plugin update (เพราะอยู่ใน config path ไม่ใช่ plugin path)

## หลังจาก Quickstart แล้วทำอะไรต่อ

1. อ่านหมวด **02 Plugins** เพื่อทำความเข้าใจแต่ละ plugin (กำลังเขียน)
2. อ่าน **03 Skills** เพื่อดูตัวอย่าง skill ที่น่าสนใจ (กำลังเขียน)
3. ทดลอง customize practice profile — แก้ไข `CLAUDE.md` ของ plugin ตรง ๆ
4. หาก plugin ไม่ตรงความต้องการ — ลองหา community skill ผ่าน `legal-builder-hub`
5. หากอยากสร้าง skill เอง — อ่าน CONTRIBUTING.md ของ repo ต้นฉบับ

## คำสรุป

> **Claude for Legal ไม่ใช่ "ของวิเศษ"** — ระบบจะทำงานดีเฉพาะเมื่อ
>
> 1. ทำ cold-start interview อย่างจริงจัง (seed document เยอะ ๆ ดีกว่า)
> 2. เชื่อม research tool (อย่างน้อย CourtListener)
> 3. ทนายตรวจสอบทุก output ก่อนนำไปใช้
>
> ระบบช่วย **เร่งงาน 80%** ที่ทำซ้ำได้ — เพื่อให้ทนายได้เอาเวลาไปคิดกับ **20%** ที่ต้องใช้ดุลพินิจจริง

← กลับไป [01 ภาพรวม](./)
