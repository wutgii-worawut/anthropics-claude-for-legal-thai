---
title: 03 Plugins
nav_order: 4
has_children: true
permalink: /03-plugins/
---

# 03 Plugins — รวมทุก plugin ใน `claude-for-legal`

> "Same system prompt, same skills — you choose where it runs."
> — Anthropic, README ของ `claude-for-legal`

`claude-for-legal` คือ **plugin marketplace** สำหรับงานกฎหมาย — 12 plugin หลักที่ Anthropic ดูแลภายใน repository (in-tree) + 1 external plugin (Thomson Reuters CoCounsel) ที่ลงทะเบียนใน `external_plugins/` รวมทั้งสิ้น **13 plugin** ครอบคลุม practice area ของวงการกฎหมายเกือบทั้งหมด

แต่ละ plugin คือโฟลเดอร์ที่มี:

- `.claude-plugin/plugin.json` — manifest (ชื่อ, version, description, author)
- `skills/<name>/SKILL.md` — task doctrine แต่ละหน่วยงาน (1 โฟลเดอร์ = 1 skill)
- `agents/<name>.md` — scheduled / data-triggered agents (ถ้ามี)
- `hooks/hooks.json` — pre/post tool hooks (มี placeholder ทุก plugin)
- `references/` — playbook, currency-watch, template (ถ้ามี)
- `.mcp.json` — รายชื่อ MCP server ที่ plugin ใช้
- `CLAUDE.md` — practice profile template ที่ลูกค้าจะใช้ตอน cold-start interview
- `README.md` — แนะนำ plugin ใน source

## วิธีอ่านหมวดนี้

| # | Plugin | สำหรับ | สถานะ |
|---|--------|--------|------|
| 1 | [commercial-legal](./commercial-legal.html) | สัญญาเชิงพาณิชย์ (NDA, vendor, SaaS) — review, escalation, renewal tracking | in-tree |
| 2 | [privacy-legal](./privacy-legal.html) | กฎหมายความเป็นส่วนตัว — PIA, DPA, DSAR, policy monitor | in-tree |
| 3 | [product-legal](./product-legal.html) | product counsel — launch review, marketing claims, "is this a problem?" | in-tree |
| 4 | [corporate-legal](./corporate-legal.html) | M&A diligence, board minutes, disclosure schedules, entity compliance | in-tree |
| 5 | [employment-legal](./employment-legal.html) | จ้างงาน เลิกจ้าง wage & hour leave investigations | in-tree |
| 6 | [regulatory-legal](./regulatory-legal.html) | เฝ้าฟีดกฎใหม่ diff กับ policy library NPRM comment tracker | in-tree |
| 7 | [ai-governance-legal](./ai-governance-legal.html) | AI registry, AIA, AI vendor terms, EU AI Act | in-tree |
| 8 | [ip-legal](./ip-legal.html) | trademark clearance, FTO, C&D, DMCA, open source compliance | in-tree |
| 9 | [litigation-legal](./litigation-legal.html) | claim charts, chronology, depo prep, privilege log, discovery | in-tree |
| 10 | [law-student](./law-student.html) | Socratic drilling, case briefs, IRAC, bar prep | in-tree |
| 11 | [legal-clinic](./legal-clinic.html) | student onboarding, intake, malpractice-aware deadline tracking (ABA Op. 512) | in-tree |
| 12 | [legal-builder-hub](./legal-builder-hub.html) | ติดตั้ง community skill พร้อม security review gate | in-tree |
| 13 | [cocounsel (Thomson Reuters)](./external/cocounsel.html) | external plugin — เชื่อม CoCounsel agent stack | external |

## การจัดกลุ่มอ่าน (suggested reading path)

หมวดนี้แยกเอกสารตาม plugin ตัวต่อตัว แต่หากต้องการอ่านเป็น "กลุ่ม" ตามลักษณะงาน เราแนะนำลำดับนี้

### กลุ่ม 1 — Practice-facing in-house (เริ่มที่นี่ ถ้าเป็นทนายในบริษัท)

1. **commercial-legal** — งาน contract review ที่พบมากที่สุด
2. **privacy-legal** — PIA, DPA, DSAR สำหรับทีม privacy/compliance
3. **product-legal** — product counsel ที่ต้อง review launch
4. **corporate-legal** — corporate secretary, M&A diligence

### กลุ่ม 2 — Specialty practice

5. **employment-legal** — HR counsel
6. **regulatory-legal** — regulatory affairs
7. **ai-governance-legal** — AI governance officer
8. **ip-legal** — IP counsel
9. **litigation-legal** — litigation team

### กลุ่ม 3 — Education + builder

10. **law-student** — นักศึกษา / บัณฑิต
11. **legal-clinic** — clinic supervisor + นักศึกษาในคลินิก
12. **legal-builder-hub** — ผู้ที่อยากเพิ่ม community skill เข้ามาเอง

### External

13. **cocounsel** — สำหรับองค์กรที่ใช้ Thomson Reuters CoCounsel อยู่แล้ว

## โครงสร้างของเอกสารแต่ละ plugin

เพื่อให้อ่านสอดคล้องกัน เอกสารของทุก plugin จะมี 10 หัวข้อตามลำดับนี้

1. **บทนำ** — plugin ทำอะไร เหมาะกับใคร
2. **`.claude-plugin/plugin.json`** — manifest พร้อมอธิบายทุกฟิลด์
3. **โครงสร้างโฟลเดอร์** — tree diagram
4. **Skills ทั้งหมด** — ตาราง name / path / คำสั่ง / สำหรับ
5. **Agents** — ตาราง agent name / description / trigger (ถ้ามี)
6. **Hooks** — pre/post tool hook ใน `hooks/hooks.json`
7. **References** — playbook, currency-watch, template (ถ้ามี)
8. **Connectors (MCP)** — รายชื่อ MCP server ใน `.mcp.json`
9. **Workflow ตัวอย่าง** — 1–2 case การใช้งานจริง
10. **ข้อควรระวัง** — เหตุที่ทุก output ต้องผ่าน attorney review

## เกณฑ์ "in-tree" vs "external"

- **in-tree** = อยู่ในโฟลเดอร์ของ repo โดยตรง — Anthropic ดูแลทั้ง code review, security review, version bump
- **external** = ลงทะเบียนใน `external_plugins/<vendor>/` — Anthropic ยืนยันแค่ว่า plugin มีจริง vendor (เช่น Thomson Reuters) เป็นผู้ดูแล code

## ภาษาในตาราง skill — รหัสคำสั่ง

ในตาราง skill ของแต่ละ plugin เราจะใช้รูปแบบคำสั่งของ Claude Code ดังนี้

| รูปแบบ | ตัวอย่าง | ความหมาย |
|--------|---------|---------|
| `/<plugin>:<skill>` | `/commercial-legal:nda-review` | เรียก skill โดยตรง (user invocable) |
| `(reference)` | — | skill ที่โหลดจาก skill อื่น (มี `user-invocable: false`) |
| `(scheduled)` | — | scheduled agent ที่รันตามเวลา |
| `(triggered)` | — | data-triggered agent (รันเมื่อมีสัญญาณบางอย่าง) |

## คำเตือนสำคัญ (จากต้นทาง)

> "These plugins do not represent Anthropic's legal positions. They are tools that help lawyers analyze issues. ... The attorney using the plugin — not the plugin, and not Anthropic — is responsible for the legal positions taken in their work product."
> — `claude-for-legal/README.md` §1.1

แปลไทย: **plugin เหล่านี้ไม่ใช่ความเห็นทางกฎหมายของ Anthropic** เป็นเพียงเครื่องมือช่วยทนายความ output ทุกชิ้นเป็น "เครื่องมือช่วยคิด" — ทนายความที่ใช้ (ไม่ใช่ plugin และไม่ใช่ Anthropic) คือผู้รับผิดชอบสุดท้าย

---

> หมายเหตุสำหรับผู้ดูแลเอกสาร: ไฟล์ `index.md` นี้เป็น **section landing** ที่อ่านได้แบบเป็นทางเข้าของหมวด 03 ทั้งหมด ห้าม agent ที่เขียน plugin ใดเป็นรายตัวมาแก้ index นี้ — แก้เฉพาะไฟล์ลูกของตน
