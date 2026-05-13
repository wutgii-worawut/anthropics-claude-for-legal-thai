---
title: ไฟล์ระดับ root
parent: 02 โครงสร้างโฟลเดอร์
nav_order: 1
---

# ไฟล์ระดับ root ของ repo

หน้านี้อธิบายไฟล์ทุกไฟล์ที่อยู่ระดับบนสุดของ `claude-for-legal/` — ไฟล์เหล่านี้คือ **คู่มือ + ทะเบียน plugin + license + กติกาชุมชน** ไม่ได้เป็น code ของ plugin ใด ๆ แต่จำเป็นต้องอ่านก่อนใช้งานหรือ contribute

## ภาพรวมไฟล์ root

```text
claude-for-legal/
├── .claude-plugin/
│   └── marketplace.json    ← ทะเบียน plugin (อธิบายในหน้า marketplace.md)
├── .gitignore
├── .github/
│   └── workflows/          ← CI scripts (lint-tool-scope, validate)
├── README.md               ← คู่มือหลัก ~52KB
├── QUICKSTART.md           ← ติดตั้งใน 60 วินาที ~5KB
├── CONNECTORS.md           ← วิธีเพิ่ม MCP connector
├── CONTRIBUTING.md         ← หลักการออกแบบสำหรับ PR
├── LICENSE                 ← Apache-2.0
└── CODE_OF_CONDUCT.md      ← มาตรฐานชุมชน
```

## README.md — คู่มือหลัก

| ฟิลด์ | ค่า |
|---|---|
| ขนาด | ~52 KB (ใหญ่ที่สุดในระดับ root) |
| ทำหน้าที่อะไร | คู่มืออ้างอิงฉบับเต็ม — list ทุก plugin, ทุก agent ชื่อ-เรียกใช้งาน, ทุก managed-agent cookbook, ทุก MCP connector |
| ใครจะอ่าน | ผู้ใช้ที่ผ่าน QUICKSTART มาแล้ว และต้องการรู้ว่ามี skill อะไรให้เรียกบ้าง, ใช้กับ research tool ตัวไหนได้ |
| โครงสร้างย่อย | "Getting started", "Agents" (ตารางใหญ่ที่ list ทุก agent), "Plugins" (รายละเอียดแต่ละ plugin), "Managed Agent cookbooks", "Connectors", "Safety", "Architecture" |

**สำคัญ**: README เปิดด้วย disclaimer สำคัญที่ทุกผู้ใช้ต้องเข้าใจ:

> *"Every output from these plugins is a draft for attorney review — not legal advice, not a legal conclusion, not a substitute for a lawyer."*

นี่ไม่ใช่ legal product สำเร็จรูป — เป็น **เครื่องมือช่วยทนาย**

## QUICKSTART.md — ติดตั้งใน 60 วินาที

| ฟิลด์ | ค่า |
|---|---|
| ขนาด | ~5 KB |
| ทำหน้าที่อะไร | สเต็ปสั้น ๆ 6 ข้อ — จาก install Claude Desktop ไปจนถึง run cold-start interview |
| ใครจะอ่าน | ผู้ใช้ครั้งแรก (ทั้ง Claude Code และ Claude Cowork) |

หัวข้อหลัก:

1. Install in Claude Cowork (เวอร์ชัน desktop)
2. Install in Claude Code (CLI)
3. **Install user-scoped, not project-scoped** ← สำคัญ; user มักเลือกผิด
4. ตารางเลือก plugin ตามอาชีพ (privacy lawyer → `privacy-legal`, ฯลฯ)
5. What you're installing (อธิบาย practice profile + `~/.claude/plugins/config/...`)
6. Stuck? — troubleshooting 5 ปัญหายอดฮิต

**Tip**: ถ้าจะแนะนำเพื่อนทดลอง ส่ง QUICKSTART พอ — ไม่ต้องส่ง README

## CONNECTORS.md — วิธีเพิ่ม MCP connector

| ฟิลด์ | ค่า |
|---|---|
| ขนาด | ~4.4 KB |
| ทำหน้าที่อะไร | guideline สำหรับผู้พัฒนา MCP server ที่ต้องการให้ Anthropic merge connector ของตนเข้า default `.mcp.json` |
| ใครจะอ่าน | vendor ของ legal database (Westlaw, CourtListener, Ironclad, iManage ฯลฯ) ที่ต้องการ integrate |

หัวข้อหลัก:

- "What makes a good legal MCP connector" — checklist 5 ข้อ (HTTPS, read-heavy, provenance, no instruction-like content, graceful degradation)
- "How to submit" — เปิด PR เพิ่ม URL เข้า `.mcp.json` ของ plugin ที่เกี่ยวข้อง
- "Current connectors" — ตารางว่า connector ใดถูกใช้ใน plugin ใดบ้าง

## CONTRIBUTING.md — หลักการออกแบบสำหรับ PR

| ฟิลด์ | ค่า |
|---|---|
| ขนาด | ~4.4 KB |
| ทำหน้าที่อะไร | บอกหลักการ (ไม่ใช่ style guide) ที่ผู้ contribute ต้องเข้าใจ — โดยเฉพาะหลัก *"SKILL.md encodes the right behavior; CLAUDE.md guardrails are the net"* |
| ใครจะอ่าน | ผู้ contribute ที่จะ PR skill ใหม่หรือแก้ skill เดิม |

ประเด็นสำคัญที่ต้องรู้:

> *"If a skill's correct output depends on a CLAUDE.md guardrail catching a mistake the SKILL.md would have made, that's a design smell."*

→ SKILL.md ต้องบอกตรง ๆ ว่าทำอะไร อย่าพึ่ง CLAUDE.md ช่วยกัน mistake

CONTRIBUTING ยังครอบคลุม:

- CLA (Contributor License Agreement) — ต้อง sign ครั้งเดียวต่อนึง contributor
- "Sources are tagged by tier" — ทุก citation ต้องบอกที่มา
- "User-stated legal facts are not facts" — ต้อง verify เสมอ

## LICENSE — Apache-2.0

| ฟิลด์ | ค่า |
|---|---|
| ขนาด | ~11 KB |
| ทำหน้าที่อะไร | สัญญาอนุญาตให้ใช้ — ผู้ใช้ทำอะไรได้บ้าง |
| ใครจะอ่าน | ทนายของบริษัทที่จะรับ repo นี้เข้าระบบ |

**Apache-2.0** = permissive license สามารถใช้เชิงพาณิชย์, แก้ไข, แจกจ่ายได้ ตราบที่:

- เก็บ copyright + license notice
- ระบุการเปลี่ยนแปลงที่ทำ
- ไม่ใช้ trademark ของ Anthropic โดยไม่ได้รับอนุญาต

## CODE_OF_CONDUCT.md — มาตรฐานชุมชน

| ฟิลด์ | ค่า |
|---|---|
| ขนาด | ~1.8 KB |
| ทำหน้าที่อะไร | กฎพฤติกรรมในการ contribute / open issue / discuss |
| ใครจะอ่าน | ทุกคนที่จะมีปฏิสัมพันธ์กับชุมชนนี้ |

อิงจาก Contributor Covenant — มาตรฐานสากลที่ open-source project นิยมใช้

## .claude-plugin/marketplace.json — ทะเบียน plugin

| ฟิลด์ | ค่า |
|---|---|
| ขนาด | ~4 KB |
| ทำหน้าที่อะไร | JSON registry ที่บอก Claude Code ว่า repo นี้มี plugin อะไรบ้าง, แต่ละตัวอยู่ path ไหน |
| ใครจะอ่าน | Claude Code (อ่านอัตโนมัติเมื่อ `/plugin marketplace add`) |

ดูรายละเอียดในหน้า [marketplace](marketplace.html) — สำคัญพอที่จะมีหน้าแยก

## .github/ — CI / Workflows

```text
.github/
└── workflows/
    └── (validate.yml, lint-tool-scope.yml, test-cookbooks.yml ฯลฯ)
```

ไม่ใช่สิ่งที่ end-user จะอ่าน แต่บอกได้ว่า Anthropic ใช้ script อะไร enforce คุณภาพ:

- `scripts/validate.py` — ตรวจ `plugin.json`, `SKILL.md` frontmatter
- `scripts/lint-tool-scope.py` — ตรวจว่า skill ขอ tool เกินที่จำเป็นไหม
- `scripts/test-cookbooks.sh` — รัน managed-agent cookbook test
- `scripts/deploy-managed-agent.sh` — deploy cookbook → Managed Agents API
- `scripts/orchestrate.py` — reference event-bus สำหรับ cross-agent handoff

## .gitignore

ขนาดเล็ก (64 bytes) — เพราะ repo นี้ตั้งใจ commit เกือบทุกอย่าง รวมถึง template `_README.md` ในโฟลเดอร์ว่างเปล่า เพื่อให้ user clone มาแล้วเห็นโครงสร้างได้เลย

## สรุปลำดับการอ่านที่แนะนำ

| ถ้าท่านเป็น... | อ่านตามลำดับ |
|---|---|
| ผู้ใช้ครั้งแรก | `QUICKSTART.md` → `README.md` (เฉพาะส่วน plugin ที่ใช้) |
| ทนายที่กลัวกฎหมาย | `README.md` ส่วน "Safety" + `LICENSE` → `CODE_OF_CONDUCT.md` |
| ผู้พัฒนา MCP server | `CONNECTORS.md` → `README.md` ส่วน "Connectors" |
| Contributor (PR) | `CONTRIBUTING.md` → `CODE_OF_CONDUCT.md` → relevant plugin's `README.md` |
| IT / DevOps | `.claude-plugin/marketplace.json` → `scripts/` → `.github/workflows/` |
