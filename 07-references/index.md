---
title: 07 References
nav_order: 8
has_children: true
permalink: /07-references/
---

# 07 References — common templates ที่ใช้ข้าม plugin

> *"Shared by all Claude for Legal plugins. The first plugin you set up writes this; the rest read it."*
> — `references/company-profile-template.md` บรรทัดที่ 3

หมวดนี้คือคำอธิบายของไฟล์ใน **`references/` ระดับ root** ของ repository `anthropics/claude-for-legal` — ไฟล์ template ที่ **ทุก plugin ใช้ร่วมกัน** ไม่ใช่ plugin-specific

ในขณะที่หมวด `02-folders` มี [`references/` ของแต่ละ plugin](../02-folders/plugin-anatomy.html) (เช่น `commercial-legal/references/playbook-default.md`) — `references/` **ระดับ root** เก็บเฉพาะ template ที่ **ข้าม plugin** ได้

## ความต่างของ `references/` ระดับ root vs ระดับ plugin

| Property | `references/` (root) | `<plugin>/references/` |
|---|---|---|
| **Scope** | shared ข้ามทุก plugin | เฉพาะ plugin นั้น |
| **Owner** | Anthropic / repo maintainer | plugin author |
| **Examples** | `company-profile-template.md`, `dashboard-template.md` | `playbook-default.md`, `summary-style.md`, `claim-chart-template.md` |
| **อ่านโดย** | Skill ของหลาย plugin | Skill ของ plugin เดียว |
| **ที่อยู่หลังติดตั้ง** | `~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md` (per plugin) | `~/.claude/plugins/config/claude-for-legal/<plugin>/references/` |

จุดสำคัญที่ไม่ค่อยมีคนสังเกต: **`company-profile-template.md` ถูก copy ไปเป็น `CLAUDE.md` ของแต่ละ plugin หลังตอน setup** — ดังนั้น file template นี้คือ **schema ของ shared context** ไม่ใช่ "ไฟล์ที่ skill อ่านตรง ๆ"

## ไฟล์ในหมวดนี้

| ลำดับ | ไฟล์ | สิ่งที่อธิบาย |
|---|---|---|
| 1 | [company-profile](company-profile.html) | template สำหรับเก็บ shared firm/company context — practice setting, jurisdictions, risk posture, key people |
| 2 | [dashboard](dashboard.html) | dashboard structure guideline สำหรับ tabular review, renewal tracker, entity compliance ใน Cowork/Claude Code/Excel |

## ทำไม template เหล่านี้ถึงสำคัญ

Claude for Legal เป็น **default-free design** — ตอนติดตั้ง plugin แรก ระบบจะ "interview" ผู้ใช้แล้วเขียน facts ใส่ `CLAUDE.md` ของ plugin (จาก `company-profile-template.md`) เพื่อให้ skill อ่าน

> จาก `CONTRIBUTING.md` ใน repo ต้นทาง:
> *"Plugins are default-free: they ask the firm/practice for the relevant defaults at setup time and write them to the practice profile. Skills then read those defaults at runtime."*

ผลคือ:

1. **Skill ไม่ hardcode jurisdiction** — อ่าน `Jurisdictions we operate in:` จาก profile แทน
2. **Skill ไม่ assume risk appetite** — อ่าน `Overall risk appetite:` แทน  
3. **Skill รู้ว่าจะ escalate ไปหาใคร** — อ่าน `Escalation chain:` แทน

ถ้าไม่มี template ชุดนี้ — แต่ละ skill จะมี implicit assumption แตกต่างกัน → ผลคือ output ไม่สม่ำเสมอข้าม skill

## หลักการ "first plugin writes, rest read"

```text
                          ┌────────────────────────────┐
                          │ user install plugin แรก     │
                          │ (เช่น commercial-legal)     │
                          └────────────┬───────────────┘
                                       │ run /setup
                                       ▼
                          ┌────────────────────────────┐
                          │ interview user             │
                          │ (industry? jurisdictions?  │
                          │  risk appetite? GC?)       │
                          └────────────┬───────────────┘
                                       │ เขียนคำตอบลง
                                       ▼
              ~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md
                          (จาก template ของ company-profile-template.md)
                                       │
                                       ▼
       user install plugin ตัวที่สอง (เช่น privacy-legal)
                                       │ run /setup
                                       ▼
                          ┌────────────────────────────┐
                          │ ตรวจว่ามี CLAUDE.md ของ      │
                          │ plugin อื่นอยู่หรือยัง       │
                          │ ถ้ามี → ดู                  │
                          │ ถ้าไม่ → interview ใหม่      │
                          └────────────────────────────┘
```

ดังนั้น `company-profile-template.md` ใน repo คือ **canonical source of truth** — รูปทรงที่ทุก plugin คาดหวัง

## วิธีอ่านที่แนะนำ

- ถ้าเป็น **legal ops / GC** → อ่าน [company-profile](company-profile.html) ก่อน เพราะเป็นเรื่องที่คุณจะต้องกรอกในรอบ setup
- ถ้าเป็น **dev ที่อยาก customize dashboard** → อ่าน [dashboard](dashboard.html) เพื่อรู้ structure ที่ guardrail บังคับ
- ถ้าเป็น **plugin author** → อ่านทั้งสอง — รู้ว่า skill ของคุณคาดหวัง context อะไรจาก profile + dashboard output ต้องตรงตามไหน

## ที่ว่างของ section นี้

`references/` ระดับ root มีไฟล์น้อย (ปัจจุบัน 2 ไฟล์) เพราะ Anthropic ตั้งใจ keep cross-plugin template ให้ **น้อยและจำเป็น** — ส่วนใหญ่ของ "knowledge" อยู่ใน plugin-specific `references/` แทน

ถ้าในอนาคต Anthropic เพิ่ม template ใหม่ใน `references/` ระดับ root จะเป็น signal ที่บอกว่า "ทุก plugin ต้องรู้เรื่องนี้" — ดังนั้นการเช็คโฟลเดอร์นี้เป็นระยะคือ healthy practice สำหรับ ops engineer
