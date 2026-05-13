---
title: 02 โครงสร้างโฟลเดอร์
nav_order: 3
has_children: true
permalink: /02-folders/
---

# 02 โครงสร้างโฟลเดอร์ของ Claude for Legal

> "Each plugin learns your playbook through a setup interview, writes it to a practice profile file…"
> — `QUICKSTART.md`

หมวดนี้คือ **แผนที่ละเอียด** ของ repository `anthropics/claude-for-legal` ทุกโฟลเดอร์ ทุกไฟล์ระดับ root ทุก subdirectory ภายใน plugin — มีความหมายอะไร ทำหน้าที่อะไร และใครเป็นผู้ใช้งานหลัก

ถ้าหมวด `01-overview` คือคำตอบของคำถาม *"Claude for Legal คืออะไร"* หมวดนี้คือคำตอบของคำถาม *"พอ clone repo มาแล้วเปิดดู ทำไมโฟลเดอร์มันเยอะขนาดนี้?"*

## ทำไมต้องเข้าใจโครงสร้างก่อนใช้งาน

Anthropic ออกแบบ repo นี้ให้เป็น **plugin marketplace** — ไม่ใช่โปรเจกต์ Python หรือ Node app ที่มี `src/` กับ `tests/` ตามแบบมาตรฐาน แต่เป็นชุดของ **12 plugins + 5 cookbooks** ที่แต่ละตัวเป็น "งานสำเร็จรูป" ที่ติดตั้งแยกได้ ภายในแต่ละ plugin มีสถาปัตยกรรม layered: `skills/`, `agents/`, `hooks/`, `references/`, plus practice-level `CLAUDE.md` ที่ skill ทุกตัวอ่านได้

ทำความเข้าใจโฟลเดอร์ก่อน → จะทำให้:

1. **อ่าน source code ของ plugin ได้รู้เรื่อง** — เพราะรู้ว่า `SKILL.md` อยู่ตรงไหน, hook คือไฟล์อะไร, agent ต่างจาก skill อย่างไร
2. **เขียน plugin ของตัวเองได้** — โดยใช้โครงเดียวกัน (Anthropic กำหนด convention ผ่าน `legal-builder-hub`)
3. **debug ได้** เมื่อ skill หา reference file ไม่เจอ หรือ matter workspace ไม่ทำงาน

## ภาพรวมระดับ root

```text
claude-for-legal/
├── .claude-plugin/
│   └── marketplace.json          # ทะเบียน plugin ทั้ง 13 ตัว (ดู marketplace.md)
├── README.md                     # คู่มือหลัก
├── QUICKSTART.md                 # 60-วินาทีติดตั้ง
├── CONNECTORS.md                 # วิธีเพิ่ม MCP connector ใหม่
├── CONTRIBUTING.md               # หลักการออกแบบ + วิธี contribute
├── LICENSE                       # Apache-2.0
├── CODE_OF_CONDUCT.md            # มาตรฐานชุมชน
├── .gitignore
├── commercial-legal/             # plugin: review NDA / vendor agreement
├── privacy-legal/                # plugin: PIA / DPA / DSAR
├── product-legal/                # plugin: launch review
├── corporate-legal/              # plugin: M&A diligence
├── employment-legal/             # plugin: hire / fire / leave
├── regulatory-legal/             # plugin: rule diff / comment
├── ai-governance-legal/          # plugin: AI use-case triage
├── litigation-legal/             # plugin: claim chart / depo prep
├── ip-legal/                     # plugin: trademark / DMCA / OSS
├── law-student/                  # plugin: Socratic drill / outline
├── legal-clinic/                 # plugin: clinic intake / handoff
├── legal-builder-hub/            # plugin: install / disable skills
├── external_plugins/
│   └── cocounsel-legal/          # plugin จากผู้พัฒนาภายนอก (Thomson Reuters)
├── managed-agent-cookbooks/      # 5 cookbook สำหรับ Managed Agents API
│   ├── diligence-grid/
│   ├── docket-watcher/
│   ├── launch-radar/
│   ├── reg-monitor/
│   └── renewal-watcher/
├── references/                   # template ใช้ร่วมข้าม plugin
└── scripts/                      # deploy / validate / lint
```

จุดสำคัญ: **ทุก plugin มีโครงสร้างคล้ายกัน** แต่ไม่เหมือนกันเป๊ะ ๆ — ขึ้นอยู่กับว่า plugin นั้นต้องการ `agents/`, `hooks/`, หรือ workspace folder พิเศษ (เช่น `matters/`, `inbound/`) ไหม

## ไฟล์ในหมวดนี้

| ลำดับ | ไฟล์ | สิ่งที่อธิบาย |
|---|---|---|
| 1 | [root-files](root-files.html) | ไฟล์ระดับ root ทุกไฟล์ — README, QUICKSTART, CONNECTORS, CONTRIBUTING, LICENSE, CODE_OF_CONDUCT, `.claude-plugin/marketplace.json` |
| 2 | [marketplace](marketplace.html) | `.claude-plugin/marketplace.json` — schema, in-tree vs external plugins |
| 3 | [plugin-anatomy](plugin-anatomy.html) | โครงสร้างมาตรฐานของ plugin (`.claude-plugin/plugin.json`, `skills/`, `agents/`, `hooks/`, `references/`, `CLAUDE.md`) |
| 4 | [skills-folder](skills-folder.html) | โฟลเดอร์ `skills/` — `SKILL.md` format, naming convention, subfolder, `references/` |
| 5 | [agents-and-hooks](agents-and-hooks.html) | โฟลเดอร์ `agents/` และ `hooks/` — scheduled worker + guardrail |
| 6 | [cookbooks-folder](cookbooks-folder.html) | โฟลเดอร์ `managed-agent-cookbooks/` — `agent.yaml`, leaf-worker pattern |
| 7 | [special-folders](special-folders.html) | โฟลเดอร์พิเศษเฉพาะบาง plugin: `matters/`, `inbound/`, `demand-letters/`, `oc-status/`, `client-comms/`, `handoffs/`, `data/` |

## วิธีอ่านที่แนะนำ

- ถ้าเพิ่งเปิด repo ครั้งแรก → อ่าน [root-files](root-files.html) ก่อนเพื่อหา README/QUICKSTART
- ถ้าจะเขียน plugin ใหม่ → อ่าน [plugin-anatomy](plugin-anatomy.html) → [skills-folder](skills-folder.html) → [agents-and-hooks](agents-and-hooks.html) ตามลำดับ
- ถ้าสนใจ Managed Agents API → ข้ามไป [cookbooks-folder](cookbooks-folder.html)
- ถ้าสงสัยว่า `matters/[slug]/matter.md` คือไฟล์อะไร → [special-folders](special-folders.html)

## หลักการที่อยู่เบื้องหลังการแบ่งโฟลเดอร์

จาก `CONTRIBUTING.md`:

> *"`<plugin>/skills/<skill>/SKILL.md` — what this specific skill does, step by step. The narrow, task-specific scaffold. `<plugin>/CLAUDE.md` — the shared guardrails and the practice profile."*

นั่นคือเหตุผลที่:

- **`skills/` กับ `CLAUDE.md` ต้องแยกกัน** — เพราะ skill คือ "ทำอะไร" และ `CLAUDE.md` คือ "อย่าทำผิดในรูปแบบไหน" (safety net)
- **`agents/` แยกจาก `skills/`** — agent คือ scheduled worker ที่ทำงานเอง ไม่ใช่ skill ที่ user เรียก
- **`hooks/` แยกจากทุกอย่าง** — hook คือ pre/post tool gate ที่ run โดยอัตโนมัติเพื่อบล็อก action เสี่ยง
- **`references/` แยก** — เพราะเป็น static data (เช่น checklist, schema, plausibility band) ที่ skill อ่าน ไม่ใช่ instruction

เมื่อเข้าใจหลักการนี้แล้ว ทุกโฟลเดอร์ใน repo จะกลายเป็น "เหตุผล" ไม่ใช่ "ที่เก็บของ"
