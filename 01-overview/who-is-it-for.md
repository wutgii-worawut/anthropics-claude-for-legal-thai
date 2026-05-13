---
title: กลุ่มผู้ใช้เป้าหมาย
parent: 01 ภาพรวม
nav_order: 2
---

# กลุ่มผู้ใช้เป้าหมาย

> "Reference agents, skills, and data connectors for the legal workflows we see most — in-house commercial, privacy, product, corporate, employment, litigation, regulatory, AI governance, IP, and the learning side of the practice (law school clinics and students)."
> — README, anthropics/claude-for-legal

Claude for Legal ออกแบบมาเพื่อ **ผู้ที่ทำงานสายกฎหมายเป็นอาชีพ** — ไม่ใช่ทดแทนทนาย แต่เป็น "ผู้ช่วย" ที่เร่งงานในส่วนที่ทนายต้องทำซ้ำ ๆ ทุกวัน ผู้พัฒนาเล็งกลุ่มผู้ใช้ไว้ **5 กลุ่มหลัก** บวกผู้ใช้รอง (เช่น legal ops)

หน้านี้จะระบุว่าแต่ละกลุ่มควรเริ่มจาก plugin ใด และให้ภาพว่า workflow ของตน จะถูก plugin ช่วยอย่างไร

## ตารางสรุป — ผู้ใช้แต่ละกลุ่มกับ plugin ที่เหมาะสม

ตารางจาก QUICKSTART ของ Anthropic (แปลและขยายความ)

| ท่านคือ... | ติดตั้ง plugin | command แรกที่ควรลอง |
|-----------|---------------|---------------------|
| ทนายความคุ้มครองข้อมูล / DPO | `privacy-legal` | `/privacy-legal:use-case-triage` |
| ทนายความสัญญาเชิงพาณิชย์ | `commercial-legal` | `/commercial-legal:review` |
| ทนายความ M&A / corporate | `corporate-legal` | `/corporate-legal:diligence-issue-extraction` |
| ทนายความแรงงาน / HR counsel | `employment-legal` | `/employment-legal:wage-hour-qa` |
| Product counsel | `product-legal` | `/product-legal:is-this-a-problem` |
| ทนายความ IP / patent agent | `ip-legal` | `/ip-legal:clearance` |
| Litigator (ในบริษัทหรือสำนัก) | `litigation-legal` | `/litigation-legal:matter-intake` |
| Regulatory / compliance counsel | `regulatory-legal` | `/regulatory-legal:reg-feed-watcher` |
| AI governance lead | `ai-governance-legal` | `/ai-governance-legal:use-case-triage` |
| Clinic supervisor (มหาวิทยาลัย) | `legal-clinic` | `/legal-clinic:cold-start-interview` |
| นักศึกษากฎหมาย | `law-student` | `/law-student:cold-start-interview` |
| Legal ops / กำลังหา skill ใหม่ ๆ | `legal-builder-hub` | `/legal-builder-hub:registry-browser` |

## กลุ่มที่ 1 — In-House Counsel (ที่ปรึกษากฎหมายในบริษัท)

### ใครคือกลุ่มนี้

ทนายความที่ทำงานในนิติบุคคล (ไม่ใช่สำนักงานกฎหมาย) — มีตำแหน่งเช่น General Counsel (GC), Deputy GC, Senior Counsel, Legal Counsel, Compliance Officer หน้าที่หลักคือ **บริหารความเสี่ยงทางกฎหมายของบริษัท** ไม่ใช่การฟ้องคดีในนามลูกค้า

### Pain point ที่ Claude for Legal แก้

| ปัญหาเดิม | Plugin ที่ช่วย |
|-----------|---------------|
| ทบทวน vendor MSA / NDA หลายร้อยฉบับต่อปี | `commercial-legal` |
| ติดตาม renewal date และ cancel-by ไม่ทัน เสียค่าต่ออายุที่ไม่จำเป็น | `commercial-legal` (renewal-watcher) |
| รับคำถามจาก product team ว่า "นี่เป็นปัญหามั้ย?" หลายครั้งต่อวัน | `product-legal` (is-this-a-problem) |
| ตอบ DSAR (data subject access request) ภายในเวลาตามกฎหมาย | `privacy-legal` (dsar-response) |
| ทำ PIA / DPIA ก่อนเปิด feature ใหม่ | `privacy-legal` (pia-generation) |
| ติดตาม regulatory feed และคิดผลกระทบกับนโยบายภายใน | `regulatory-legal` |
| ทำ AI governance policy ก่อน feature ใหม่ launch | `ai-governance-legal` |

### Workflow ตัวอย่าง

> วันจันทร์เช้า 9 โมง — `renewal-watcher` agent ส่ง Slack message มาให้ทีม legal ว่า
>
> > 🔴 **3 contracts ใกล้ cancel-by ภายใน 14 วัน**:
> > 1. Acme Corp SaaS — cancel-by 2026-05-25 — annual $120,000
> > 2. Beta Hosting — cancel-by 2026-05-22 — annual $48,000 — uncapped renewal pricing
> > 3. Gamma Tools — cancel-by 2026-05-23 — annual $15,000
>
> GC คลิกเข้าไปดู contract Acme Corp ใน Ironclad, ตัดสินใจไม่ต่อ, สั่งทีมส่ง notice — เสร็จในครึ่งชั่วโมง

## กลุ่มที่ 2 — Law Firm (สำนักงานกฎหมาย)

### ใครคือกลุ่มนี้

ทนายความใน **law firm** ทั้ง partner, of counsel, senior associate, junior associate งานหลักคือ **ให้บริการลูกค้าหลายราย** — เน้น billable hour, drafting, การฟ้องคดี

### Pain point ที่ Claude for Legal แก้

| ปัญหาเดิม | Plugin ที่ช่วย |
|-----------|---------------|
| Due diligence ใน M&A — ต้องอ่านเอกสารใน VDR หลายพันฉบับ | `corporate-legal` (tabular-review) |
| สร้าง claim chart ใน patent litigation | `litigation-legal` (claim-chart) |
| เตรียม deposition outline | `litigation-legal` (deposition-prep) |
| ทำ privilege log review | `litigation-legal` (privilege-log-review) |
| Draft brief section ตาม house style | `litigation-legal` (brief-section-drafter) |
| ค้น case law ผ่าน Westlaw / Practical Law | `cocounsel-legal` (Thomson Reuters partner) |

### Workflow ตัวอย่าง

> Senior associate ใน M&A practice — ลูกค้าจะซื้อบริษัท target ขนาดกลาง มี VDR เป็น Box folder บรรจุเอกสาร 800 ฉบับ
>
> 1. รัน `/corporate-legal:tabular-review` ชี้ไปที่ Box folder
> 2. Skill สร้าง Excel multi-sheet — 1 แถวต่อ 1 เอกสาร, column สำหรับ issue (change-of-control, IP assignment, indemnity cap), ทุก cell มี citation กลับไปยังเอกสารต้นทาง
> 3. รัน `/corporate-legal:diligence-issue-extraction` ดึง issue ที่เกินเกณฑ์ materiality ของลูกค้า
> 4. Partner review Excel, focus ที่ red issue เท่านั้น — ประหยัดเวลา associate ไป 80%

## กลุ่มที่ 3 — Solo Practitioner (ทนายความเดี่ยว)

### ใครคือกลุ่มนี้

ทนายความที่เปิดสำนักงานคนเดียวหรือกลุ่มเล็ก 2–5 คน รับงานหลากหลาย — สัญญา, การฟ้องคดี, ที่ปรึกษาธุรกิจ ไม่มี paralegal เต็มเวลา ไม่มี knowledge management team

### Pain point ที่ Claude for Legal แก้

| ปัญหาเดิม | Plugin ที่ช่วย |
|-----------|---------------|
| ไม่มี playbook กลาง — review สัญญาทุกครั้งต้องคิดใหม่ | `commercial-legal` cold-start สร้าง playbook ให้ |
| Matter intake ไม่เป็นระบบ — ลืม conflict check | `litigation-legal` (matter-intake) |
| Demand letter ต้องเขียนเอง | `litigation-legal` (demand-draft) |
| ติดตาม deadline หลายคดีไม่ครบ | `litigation-legal` (portfolio-status) |
| ไม่มี research subscription | ใช้ CourtListener (ฟรี) ผ่าน MCP |

### Workflow ตัวอย่าง

> ทนายความเดี่ยวรับคดี employment dispute — ลูกค้าถูกเลิกจ้างและสงสัยว่าเป็น wrongful termination
>
> 1. รัน `/litigation-legal:matter-intake` — บันทึก facts, parties, conflict check
> 2. รัน `/litigation-legal:demand-intake` — เตรียม leverage analysis
> 3. รัน `/litigation-legal:demand-draft` — ได้ demand letter พร้อม FRE 408 gate ก่อนส่ง
> 4. ส่ง demand, log การส่งใน matter history
> 5. รัน `/employment-legal:termination-review` (ติดตั้งคู่กัน) — ตรวจ jurisdictional flag

## กลุ่มที่ 4 — Legal Clinic (คลินิกกฎหมายมหาวิทยาลัย)

### ใครคือกลุ่มนี้

อาจารย์ผู้ดูแลคลินิกกฎหมายในคณะนิติศาสตร์ — สอนนักศึกษาผ่านคดีจริงของลูกค้าที่ไม่สามารถจ้างทนายเองได้ (pro bono clinic)

### ความท้าทายเฉพาะกลุ่มนี้

- ต้องสมดุลระหว่าง **การสอน** (ปล่อยให้ student คิด) กับ **การให้บริการลูกค้าจริง** (คุณภาพต้องได้)
- ต้องเป็นไปตาม **ABA Formal Opinion 512** ว่าด้วยการใช้ AI ในการประกอบวิชาชีพ — ต้องมี competence, confidentiality, supervision
- นักศึกษาเปลี่ยนทุก semester — handoff ต้องชัดเจน

### Plugin ที่ออกแบบเฉพาะ

`legal-clinic` ฝัง **pedagogy posture** ไว้ในทุก skill — มี 3 mode:

| Posture | ความหมาย | ใช้เมื่อ |
|---------|---------|---------|
| `assist` | ทำส่วนใหญ่ให้ student แล้วเสริม | ต้องส่งงานเร็ว, deadline ใกล้ |
| `guide` | ถามคำถาม student แล้วช่วย refine | สอนตามปกติ |
| `teach` | ไม่ทำให้เลย, ถามอย่างเดียวให้คิด | beginning of semester |

### Workflow ตัวอย่าง

> นักศึกษากฎหมายปี 3 ใน immigration clinic รับคดีใหม่ — ลูกค้าขอ asylum
>
> 1. รัน `/legal-clinic:client-intake` (posture: `guide`) — student ถูกถามให้ดึงข้อมูล facts, นำเข้า cross-area issue spot (privacy concern? family law angle?)
> 2. รัน `/legal-clinic:research-start` — ได้ roadmap ว่าควรอ่าน statute ใด, case law area ใด, search term ใน Westlaw
> 3. รัน `/legal-clinic:memo` (posture: `teach`) — IRAC scaffold เปิดอยู่ student ต้องเขียนเอง, AI จะ flag ว่ามี research gap ตรงไหน
> 4. ส่งให้ supervisor (อาจารย์) ผ่าน `/legal-clinic:supervisor-review-queue`

## กลุ่มที่ 5 — Law Student (นักศึกษากฎหมาย)

### ใครคือกลุ่มนี้

นักศึกษาคณะนิติศาสตร์ — ไม่ใช่ทนายความ ไม่มีลูกค้า เป้าหมายคือ **เรียนรู้** และ **สอบผ่าน bar exam**

### หลักการสำคัญ: "Learning mode, not answer mode"

> "It never writes the answer for you."
> — README, law-student plugin

`law-student` plugin ออกแบบให้ **ไม่ตอบคำถามแทนนักศึกษา** — แต่ขัดเกลาวิธีคิด ทุก skill มีหลักการนี้ฝังไว้

### Skill เด่น

| Skill | ทำอะไร |
|-------|--------|
| `socratic-drill` | ถามคำถามแบบ Socratic — student ตอบ, AI push back ไม่ให้คำตอบตรง ๆ |
| `case-brief` | brief case ตาม format ของ student (IRAC, Reasoning ก่อน Rule, etc.) |
| `outline-builder` | สร้าง outline จาก class notes — student ต้องชี้แต่ละ section เอง |
| `irac-practice` | grade IRAC essay ของ student — feedback บน structure, issue spotting |
| `cold-call-prep` | ทำนายคำถามอาจารย์ จะถามอะไรวันพรุ่งนี้ |
| `bar-prep-questions` | MBE หรือ essay practice เฉพาะ subject ที่อ่อน |
| `flashcards` | Leitner-style spaced repetition |

### Workflow ตัวอย่าง

> นักศึกษาปี 1 เตรียม cold call วิชา Contracts เรื่อง consideration
>
> 1. รัน `/law-student:cold-call-prep contract-law consideration` — AI ทำนายคำถามอาจารย์ 8 ข้อ
> 2. รัน `/law-student:socratic-drill` — AI ถาม "what is consideration?" student ตอบ "bargained-for exchange" AI push back: "Hamer v. Sidway — the nephew was paid to NOT smoke, drink, gamble. What did the uncle gain?"
> 3. รัน `/law-student:flashcards` — สร้าง flashcards จาก casebook reading

## กลุ่มย่อย — Legal Ops, Engineer ที่สร้าง legal skill

### Legal Ops

ใช้ `legal-builder-hub` เพื่อ

- ค้น community skill ที่มีอยู่
- ติดตั้ง skill จาก registry (LegalOps Consulting `lpm-skills`, Lawvable)
- รัน QA framework ก่อน install เพื่อความปลอดภัย

### Engineer / Developer

ใช้

- `legal-builder-hub:skills-qa` — รัน QA กับ skill ตัวเองก่อน publish
- `managed-agent-cookbooks/` — ดู template สำหรับ deploy ผ่าน Managed Agents API
- `scripts/orchestrate.py` — reference implementation ของ orchestrator พร้อม security model

## คำเตือนสำหรับทุกกลุ่ม

> **ผลผลิตทุกชิ้นจาก plugin คือร่างสำหรับทนายตรวจสอบ**
> ไม่ใช่ legal advice ไม่ใช่ legal conclusion ไม่ใช่สิ่งทดแทนทนาย

ระบบมี guardrail ฝังไว้ — source attribution, jurisdiction assumption, gate ก่อนการกระทำที่กลับไม่ได้ (filed, sent, signed) — เพื่อ **เร่งการตรวจสอบ** ของทนาย ไม่ใช่ทดแทน

## เอกสารถัดไป

- [สถาปัตยกรรมระบบ](./architecture.html) — โครงสร้างภายในเป็นอย่างไร?

← กลับไป [01 ภาพรวม](./)
