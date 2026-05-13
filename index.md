---
title: หน้าแรก
layout: home
nav_order: 1
description: "เอกสารภาษาไทยของ Claude for Legal — ครอบคลุมทุกโฟลเดอร์ ทุก plugin ทุก skill"
permalink: /
---

# Claude for Legal — คู่มือภาษาไทย
{: .fs-9 }

เอกสารแปลและอธิบายแบบครบทุกโฟลเดอร์ของ [`anthropics/claude-for-legal`](https://github.com/anthropics/claude-for-legal) — reference implementation ของ AI plugins สำหรับงาน **กฎหมาย** ทุกสาย ตั้งแต่ corporate, litigation, IP, employment, privacy, regulatory, AI governance ไปจนถึง law-student และ legal-clinic
{: .fs-6 .fw-300 }

[เริ่มอ่าน — ภาพรวม](./01-overview/){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[ดู Source บน GitHub](https://github.com/anthropics/claude-for-legal){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## คืออะไร?

**Claude for Legal** เป็น reference implementation ที่ Anthropic ปล่อยเป็น open source สำหรับงาน **กฎหมายในทุกสาย** — ออกแบบเป็น **plugin marketplace** ที่ติดตั้งได้ใน Claude Cowork, Claude Code, หรือ deploy เป็น Managed Agents API

ครอบคลุม 12 plugin หลัก + 1 external plugin (Thomson Reuters CoCounsel) รวม **150 skills** ครอบคลุมงานจริงของทนายความ:

- **Commercial** (สัญญาเชิงพาณิชย์) — NDA, vendor agreement, SaaS subscription, ติดตาม renewal
- **Privacy** — PIA, DPA review, DSAR, นโยบายความเป็นส่วนตัว
- **Product** — review การปล่อยฟีเจอร์, ตรวจสอบ marketing claims
- **Corporate** — M&A diligence, board minutes, disclosure schedules
- **Employment** — hiring, termination, leave, internal investigations
- **Regulatory** — เฝ้าฟีดกฎใหม่, diff กับ policy library
- **AI Governance** — AI registry, impact assessment, AI vendor terms
- **Litigation** — claim charts (patent + civil), chronology, depo prep, privilege log
- **IP** — trademark clearance, FTO, C&D, DMCA, open source compliance
- **Law Student** — Socratic drilling, case briefs, IRAC, bar prep
- **Legal Clinic** — student onboarding, intake, malpractice-aware deadline tracking (ตาม ABA Op. 512)
- **Legal Builder Hub** — ติดตั้ง community skill พร้อม security review gate

## โครงสร้างเอกสารชุดนี้

| หมวด | เนื้อหา |
|------|---------|
| [01 ภาพรวม](./01-overview/) | What/Why/Who — เข้าใจ project ใน 5 นาที |
| [02 โครงสร้างโฟลเดอร์](./02-folders/) | อธิบายทุกโฟลเดอร์ใน repo ต้นทาง |
| [03 Plugins ทั้ง 12 ตัว](./03-plugins/) | แต่ละ practice area แบบละเอียด |
| [04 External Plugins](./04-external-plugins/) | CoCounsel + community |
| [05 Managed Agent Cookbooks](./05-cookbooks/) | 5 cookbook สำหรับ headless deployment |
| [06 Scripts](./06-scripts/) | 5 ไฟล์ใน scripts/ |
| [07 References](./07-references/) | template ในระดับ root |
| [08 การ Deploy](./08-deployment/) | Cowork vs Claude Code vs Managed Agents |
| [09 การ Development](./09-development/) | เพิ่ม skill, validate, security model |
| [10 อภิธานศัพท์](./10-glossary/) | ศัพท์กฎหมาย + เทคนิค ไทย–อังกฤษ |

## หลักการ 5 อย่างที่อยู่เบื้องหลัง

1. **Plugin-based, pure markdown** — ทุก skill คือไฟล์ `SKILL.md` ที่มี YAML frontmatter ไม่มี build, ไม่มี framework
2. **Three-layer quality** — `CLAUDE.md` (practice profile guardrails) → `SKILL.md` (task doctrine) → validator scripts; ไม่มี unit test แต่ **attorney review** คือ verification boundary
3. **Closed-schema handoffs** — การส่งงานระหว่าง agent ใช้ enumerated intent ผ่าน `orchestrate.py`; `lint-tool-scope.py` ป้องกัน privilege escalation
4. **Default-free design** — ต้อง cold-start interview ก่อนเริ่มงาน; output แต่ละชิ้นมี GREEN/YELLOW/RED tag + verbatim quote (ห้ามสรุปลอย ๆ)
5. **MCP-heavy** — 7–15 connector ต่อ plugin (Ironclad, DocuSign, iManage, Box, CourtListener, CoCounsel, Everlaw, Slack, Google Drive, Linear ฯลฯ)

## เอกสารนี้สำหรับใคร

- **ทนายความ / นักกฎหมาย** ที่อยากรู้ว่า AI agent จะมาช่วยงานยังไง ไม่ใช่แค่ "AI ทำงานแทน"
- **Legal Operations / Legal Tech** ที่ต้องการ reference สำหรับวางระบบในองค์กร
- **Solo practitioner** ที่อยากใช้ Claude Code ช่วยงานสำนักงาน
- **อาจารย์/นักศึกษานิติศาสตร์** ที่อยากเข้าใจการสอน Socratic ผ่าน AI โดยไม่ให้ AI เขียนแทน
- **นักพัฒนา** ที่อยากเข้าใจ plugin architecture ของ Claude

---

## ข้อสำคัญทางกฎหมาย (Disclaimer)

เอกสารชุดนี้เป็นการ **แปลและอธิบาย** ของ open source repository ต้นทาง ไม่ได้เป็นคำแนะนำทางกฎหมาย ไม่ได้รับรองโดย Anthropic หรือ Thomson Reuters หรือผู้ให้บริการข้อมูลใด ๆ ที่กล่าวถึง

**Output ทุกชิ้นจาก plugin เหล่านี้คือ DRAFT/LEAD สำหรับทนายความที่ได้รับใบประกอบวิชาชีพ** ตรวจสอบ — ไม่ใช่ "คำแนะนำกฎหมาย" โดยตัวมันเอง การใช้งานต้องอยู่ภายใต้กฎจรรยาบรรณวิชาชีพในเขตอำนาจของผู้ใช้

โค้ดต้นทางอยู่ภายใต้ license ตามที่ระบุใน [LICENSE](https://github.com/anthropics/claude-for-legal/blob/main/LICENSE) ของ repo ต้นทาง

---

*แปล/เรียบเรียงโดย [Apex Oracle](https://github.com/wutgii-worawut/apex-oracle) — ผู้เก็บความรู้ทุกยุค 🌳📚*
