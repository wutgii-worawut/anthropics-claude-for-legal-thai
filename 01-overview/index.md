---
title: 01 ภาพรวม
nav_order: 2
has_children: true
permalink: /01-overview/
---

# 01 ภาพรวม Claude for Legal

> "Same system prompt, same skills — you choose where it runs."
> — Anthropic, README ของ `claude-for-legal`

หมวดนี้คือจุดเริ่มต้นสำหรับผู้ที่ยังไม่เคยรู้จัก **Claude for Legal** มาก่อน เนื้อหาจะพาท่านเข้าใจตั้งแต่ระดับแนวคิด (ว่า "plugin", "skill", "agent" คืออะไรในบริบทของ Claude) ไปจนถึงสถาปัตยกรรมเชิงเทคนิคและหลักการออกแบบที่ Anthropic ฝังไว้ในระบบ

## วัตถุประสงค์ของหมวดนี้

หมวด **01 ภาพรวม** ทำหน้าที่เป็น "ปฐมบท" — เน้นให้ผู้อ่านได้ภาพใหญ่ก่อนลงรายละเอียดของแต่ละ plugin ในหมวดถัดไป ผู้อ่านที่จบหมวดนี้จะตอบคำถามต่อไปนี้ได้

| คำถาม | เอกสารที่อ่าน |
|------|--------------|
| Claude for Legal คืออะไร? เป็น product หรือ codebase? | [Claude for Legal คืออะไร](./what-is-it.html) |
| Plugin, skill, marketplace มีความหมายว่าอย่างไรในระบบนี้? | [Claude for Legal คืออะไร](./what-is-it.html) |
| โครงการนี้เหมาะกับทนายความประเภทใด? | [กลุ่มผู้ใช้เป้าหมาย](./who-is-it-for.html) |
| โครงสร้างภายในเป็นอย่างไร? plugin → skill → agent → cookbook ต่อกันอย่างไร? | [สถาปัตยกรรมระบบ](./architecture.html) |
| มีหลักการอะไรที่ Anthropic ใช้ออกแบบระบบนี้? | [หลักการออกแบบ](./principles.html) |
| ติดตั้งและเริ่มใช้งานได้อย่างไร? | [Quickstart — เริ่มต้นใช้งาน](./quickstart.html) |

## เนื้อหาในหมวด

### 1. [Claude for Legal คืออะไร](./what-is-it.html)

อธิบายว่า Claude for Legal ไม่ใช่ "AI กฎหมาย" ตัวใหม่ แต่เป็น **collection ของ plugin** ที่บรรจุ skill, agent, MCP connector, และ practice profile สำหรับงานทนายความสายต่าง ๆ ทุกอย่างเป็น markdown และ JSON — ไม่มี build step นิยามศัพท์สำคัญ: plugin, marketplace, skill, agent, cookbook, MCP, hook, practice profile

### 2. [กลุ่มผู้ใช้เป้าหมาย](./who-is-it-for.html)

ระบุ 5 กลุ่มหลัก: in-house counsel, law firm associate/partner, solo practitioner, legal clinic supervisor, law student พร้อมตัวอย่างว่าแต่ละกลุ่มควรเริ่มจาก plugin ใด

### 3. [สถาปัตยกรรมระบบ](./architecture.html)

แผนผัง (mermaid diagram) อธิบาย 3 invocation surfaces — Claude Cowork, Claude Code, Managed Agents API — และความสัมพันธ์ระหว่าง plugin → skill → agent → managed-agent-cookbook

### 4. [หลักการออกแบบ](./principles.html)

6 หลักการที่ Anthropic ใช้: **single source two runtimes**, **three-layer quality**, **closed-schema handoffs**, **default-free / cold-start interview**, **citation discipline**, และ **ABA Op. 512 + privilege-conscious**

### 5. [Quickstart — เริ่มต้นใช้งาน](./quickstart.html)

ขั้นตอนติดตั้ง 60 วินาที สำหรับ Claude Cowork และ Claude Code พร้อมตัวอย่าง command จริง

## ภาพรวมสั้น ๆ ก่อนเริ่มอ่าน

```
claude-for-legal/
├── 12 practice-area plugins  (commercial, privacy, corporate, ...)
├── 5  managed-agent cookbooks (renewal-watcher, docket-watcher, ...)
├── 16+ MCP connectors        (Slack, Drive, Ironclad, Westlaw, ...)
└── 151 skills                 (~45,000+ บรรทัด markdown)
```

ทุก plugin มีโครงสร้างเหมือนกัน — เริ่มด้วย `cold-start-interview` ที่จะ "สัมภาษณ์" ผู้ใช้เพื่อเรียนรู้ playbook ของสำนักงาน แล้วบันทึกลง `CLAUDE.md` ซึ่งเป็น **practice profile** ที่ทุก skill อ่านได้

> **คำเตือนสำคัญจาก Anthropic** (แปลและสรุป):
> ผลผลิตทุกชิ้นจาก plugin เหล่านี้ คือ **ร่างเอกสาร (draft) สำหรับให้ทนายตรวจสอบ** — ไม่ใช่คำปรึกษาทางกฎหมาย ไม่ใช่ข้อสรุปทางกฎหมาย และไม่ใช่สิ่งทดแทนทนายความ guardrail ที่ฝังไว้ในระบบ (source attribution, jurisdiction assumption, privilege marker, gate ก่อนการกระทำที่กลับไม่ได้) มีไว้เพื่อ **เร่งการตรวจสอบ** ของทนาย ไม่ใช่ทดแทนการตรวจสอบ

อ่านไฟล์แรกที่ [Claude for Legal คืออะไร](./what-is-it.html) เพื่อเริ่มต้น
