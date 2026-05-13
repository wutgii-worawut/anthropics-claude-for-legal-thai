---
title: 10 อภิธานศัพท์
nav_order: 11
has_children: true
permalink: /10-glossary/
---

# 10 อภิธานศัพท์ (Glossary)

> "ภาษาเป็นเครื่องมือของกฎหมาย และเป็นเครื่องมือของวิศวกรรม — เมื่อภาษาสองสายมาบรรจบกัน คำอธิบายที่ตรงไปตรงมาเท่านั้นที่ทำให้สื่อสารได้"

หมวดนี้เก็บ **คำศัพท์เฉพาะ** ที่ปรากฏซ้ำตลอดเอกสารชุดนี้ พร้อมคำแปลและคำอธิบายภาษาไทย เป้าหมายคือเป็น "พจนานุกรมหน้าเดียว" ที่ผู้อ่านสามารถเปิดข้ามไปมาได้เมื่อเจอคำที่ไม่คุ้น โดยเฉพาะอย่างยิ่งเมื่อบางคำในระบบกฎหมายของสหรัฐฯ ไม่มีคำเทียบเคียงตรง ๆ ในระบบกฎหมายไทย

## ทำไมต้องมีอภิธานศัพท์แยกหมวด

`claude-for-legal` ถูกเขียนขึ้นในบริบทกฎหมายของ **สหรัฐอเมริกา** (federal + state) คำศัพท์จำนวนมากในเอกสารต้นฉบับ เช่น *Privilege*, *Work Product*, *Disclosure Schedule*, *DSAR*, *FLSA*, *ABA Op. 512* ฯลฯ มีรากจากกฎหมายและธรรมเนียมวิชาชีพอเมริกัน ผู้อ่านไทยที่ไม่คุ้นเคยกับระบบ common law อาจสับสนเมื่อเจอคำเหล่านี้

ขณะเดียวกัน ฝั่ง technical ก็มีศัพท์เฉพาะของ Anthropic เช่น *plugin*, *skill*, *agent*, *MCP*, *managed agent*, *closed-schema handoff*, *cold-start interview* ฯลฯ ซึ่งเป็นภาษากลางของระบบ Claude Code/Cowork — ไม่ใช่ศัพท์มาตรฐานของวงการ software ทั่วไป

หมวดนี้จึงแบ่งเป็น **2 หัวข้อย่อย** เพื่อให้ค้นได้สะดวก

## เนื้อหาในหมวด

### 1. [ศัพท์กฎหมาย (Legal terms)](./legal-terms.html)

รวบรวม ~40-60 คำศัพท์กฎหมายที่ปรากฏใน plugin ต่าง ๆ ครอบคลุม

- **สัญญาและธุรกรรม** — NDA, DPA, M&A, LBO, Disclosure Schedule, Engagement Letter
- **ทรัพย์สินทางปัญญา** — FTO, C&D, DMCA, Trade Secret
- **ข้อมูลส่วนบุคคล** — DSAR, PIA
- **จริยธรรมและการประกอบวิชาชีพ** — ABA Op. 512, Privilege, Work Product, Conflicts Check, Conflict Waiver, Pro Bono
- **กระบวนการในคดี** — Deposition, Privilege Log, Chronology, Claim Chart
- **การคิดแบบทนาย** — IRAC, Socratic Method, Materiality
- **คำเฉพาะของสำนักงานทนาย** — Matter, Diligence, Board Consent, Entity Compliance

แต่ละคำมาพร้อมคำแปลไทย + คำอธิบายสั้น ๆ ว่าคืออะไรและพบในบริบทใดในเอกสารชุดนี้

### 2. [ศัพท์ Technical](./tech-terms.html)

รวบรวม ~25-40 คำศัพท์ technical ที่ใช้ในระบบ Claude/Anthropic ครอบคลุม

- **หน่วยพื้นฐาน** — Plugin, Skill, Agent, Hook, Subagent
- **โปรโตคอลและการเชื่อมระบบ** — MCP, Connector, Marketplace
- **โครงสร้างไฟล์** — Frontmatter, Practice Profile, Cookbook
- **รูปแบบการออกแบบ** — Closed-Schema Handoff, Cold-Start Interview, Three-Tier Security, Privilege Escalation, Verbatim Quote
- **ระดับและสถานะ** — GREEN/YELLOW/RED, Three States of "Not Found"
- **ความตระหนัก (awareness)** — Side Awareness, Jurisdiction Awareness
- **runtime เฉพาะ** — Managed Agent, Slash Command, Steering Event

## วิธีใช้อภิธานศัพท์นี้

1. **อ่านเป็นไฟล์อ้างอิง** — ไม่จำเป็นต้องอ่านรวด เปิดเมื่อเจอคำที่ไม่คุ้น
2. **ใช้กับฟังก์ชันค้นหา** — ใช้ Ctrl/Cmd + F ในเบราว์เซอร์เพื่อค้นทั้งคำไทยและคำอังกฤษ
3. **เปิดคู่กับเอกสารหลัก** — เปิด tab ใหม่ค้างไว้ขณะอ่านหมวด 03 (plugins) หรือ 05 (cookbooks)

> **หมายเหตุ**: คำแปลในที่นี้เป็น **คำแปลเชิงอธิบาย** ไม่ใช่คำแปลทางกฎหมายอย่างเป็นทางการ การใช้คำเทียบเคียงในระบบกฎหมายไทยมีข้อจำกัด เพราะหลายแนวคิด เช่น *Attorney-Client Privilege* หรือ *Work Product Doctrine* มีรากใน common law ที่ไม่มีคู่ขนานในประมวลกฎหมายไทย — ผู้อ่านที่ต้องการใช้คำเหล่านี้ในเอกสารกฎหมายไทย ควรปรึกษาทนายความผู้เชี่ยวชาญ
