---
title: company-profile (โปรไฟล์องค์กร)
parent: 07 References
nav_order: 1
---

# company-profile-template.md — shared context ที่ทุก skill อ่าน

> *"Shared by all Claude for Legal plugins. The first plugin you set up writes this; the rest read it. Edit directly or re-run any plugin's `/setup` to update."*
> — header ของ template

`references/company-profile-template.md` คือ **canonical template** ที่ทุก plugin ใน Claude for Legal ใช้เป็นแม่แบบสำหรับ "ข้อเท็จจริงเกี่ยวกับองค์กร" ที่ skill ทุกตัวอ่านเป็น context

ไฟล์นี้ไม่ใช่แค่ "ข้อมูลบริษัท" แต่เป็น **default-free design contract** — Claude for Legal เลือกไม่ hardcode สมมุติฐานใด ๆ ใน skill แทนที่จะถามผู้ใช้ตอน setup แล้วเก็บคำตอบเป็น profile เพื่อให้ skill อ่านได้ทุกครั้ง

## ทำไมต้องมี profile

skills แต่ละตัว (เช่น `nda-review`, `pia-draft`, `launch-review`) ต้องการ **shared context** เดียวกันเสมอ:

- บริษัทอยู่ใน jurisdiction อะไร — เพื่อรู้ว่าควรอ้างกฎหมายไหน
- เป็น in-house, law firm, หรือ government — เพื่อรู้ว่าควร framing output แบบไหน
- risk appetite เป็นแบบไหน — เพื่อรู้ว่าควรเข้มงวดแค่ไหน
- ใครคือ GC / escalation chain — เพื่อรู้ว่าควร flag ใครเมื่อเจอ issue

ถ้าแต่ละ skill ต้อง re-ask คำถามเหล่านี้ทุกครั้ง = ผู้ใช้เบื่อ + คำตอบไม่สอดคล้องกันระหว่าง skill

**Solution**: เก็บ shared context ใน profile (ที่ skill ทุกตัวอ่าน) — interview ครั้งเดียว ใช้ทุก plugin

## Template structure

จาก `references/company-profile-template.md` ของ Anthropic:

```markdown
# Company Profile

*Shared by all Claude for Legal plugins. The first plugin you set up writes this; the rest read it.
Edit directly or re-run any plugin's `/setup` to update.*

**Practice setting:** [Solo/small firm | Midsize/large firm | In-house | Government/legal aid/clinic]
**Name:** [Company or firm name]
**Industry:** [What the company does / the firm's primary practice areas]
**What we sell / deliver:** [Products, services, who to — or "N/A, law firm"]
**Size:** [Employee count / lawyers / relevant headcount]

## Geographic and regulatory footprint

**Jurisdictions we operate in:** [e.g., US (CA, NY, TX), UK, EU (DE, FR), AU, SG]
**Primary jurisdiction:** [Where the bulk of work happens]
**Regulators we're subject to:** [SEC, FTC, ICO, EDPB, ASIC, OAIC, etc. — only what applies]
**Open regulatory matters:** [or none]

## Risk posture

**Overall risk appetite:** [Conservative / middle / aggressive]
**What keeps us up at night:** [The thing that would be a very bad day]
**The question leadership always asks:** [or not known yet]

## Key people

**GC / Head of Legal:** [Name]
**Escalation chain:** [Name → Name → Name, or "set per plugin"]

---

*Per-plugin practice profiles (playbooks, review frameworks, house style, matter workspaces)
live alongside this file in each plugin's folder. This file holds the facts that are true
regardless of which plugin you're using.*
```

## โครงสร้างเป็น 4 sections

| Section | ข้อมูล | ใครอ่าน |
|---|---|---|
| **Identity** | Practice setting, Name, Industry, What we sell, Size | ทุก skill (default framing) |
| **Geographic and regulatory footprint** | Jurisdictions, Primary jurisdiction, Regulators, Open regulatory matters | `regulatory-legal/rule-diff`, `privacy-legal/dsar`, `commercial-legal/dpa-review` |
| **Risk posture** | Risk appetite, What keeps us up, The question leadership asks | `commercial-legal/nda-review` (เลือก fallback position), `product-legal/launch-review` |
| **Key people** | GC, Escalation chain | ทุก plugin (เมื่อต้อง flag issue) |

## ตัวอย่างการกรอก — บริษัทไทยใน SET

```markdown
# Company Profile

**Practice setting:** In-house
**Name:** บริษัท ABC จำกัด (มหาชน) — ABC Public Company Limited
**Industry:** Retail banking and consumer finance
**What we sell / deliver:** Personal loans, credit cards, digital wallet services to retail customers in Thailand
**Size:** ~12,000 employees, 35-person legal team

## Geographic and regulatory footprint

**Jurisdictions we operate in:** Thailand (primary), Lao PDR (subsidiary), Singapore (treasury entity)
**Primary jurisdiction:** Thailand
**Regulators we're subject to:** Bank of Thailand (BOT), Securities and Exchange Commission Thailand (SEC), Personal Data Protection Committee (PDPC), Office of the Anti-Money Laundering (AMLO)
**Open regulatory matters:** PDPA compliance review (ongoing since 2025-Q4), BOT data resiliency examination scheduled 2026-Q3

## Risk posture

**Overall risk appetite:** Conservative — listed entity, retail customers, BOT-regulated
**What keeps us up at night:** Personal data breach + PDPA enforcement action (administrative fine up to 5M THB + civil liability), AML breach with cross-border element
**The question leadership always asks:** "Will this trigger BOT notification or PDPC disclosure?"

## Key people

**GC / Head of Legal:** [Name] — Chief Legal & Compliance Officer
**Escalation chain:** Associate GC → Deputy GC → CLCO → CEO (for issues >50M THB or regulator-facing)
```

ตัวอย่างนี้แสดง:

- **Jurisdictions** ใช้ระดับ regulator ของไทยที่ skill ของ `privacy-legal` หรือ `regulatory-legal` ต้องรู้ (PDPA → PDPC, AML → AMLO)
- **Risk appetite** = "Conservative" → skill `nda-review` จะ default ไปหา redline ที่ปกป้อง บริษัท มากกว่าปล่อย counterparty
- **What keeps us up at night** = ข้อความ free text ที่ skill เอาไป interpret เป็น "priority issue" — เช่น PDPA breach + AML cross-border → `privacy-legal` ให้ความสำคัญ
- **Escalation chain** = สาย escalate ที่ skill จะ flag เมื่อเจอ blocking issue (เช่น "ส่งเรื่องนี้ไปที่ Associate GC ทันที")

## ทำไมต้องเป็น default-free

จาก `CONTRIBUTING.md` ใน repo ต้นทาง:

> *"Don't bake firm-specific defaults into skills. Skills should ask the practice profile at runtime."*

หลักการนี้สำคัญเพราะ:

1. **Plugin เดียว — รองรับหลาย firm setting** — solo lawyer ใน Bangkok, in-house ของ SET firm, NGO legal clinic ใช้ plugin เดียวกันได้ผ่าน profile ที่ต่างกัน
2. **เปลี่ยน firm context ทีหลังได้** — เมื่อบริษัทขยายไป Singapore แค่แก้ profile, ไม่ต้อง redeploy skill
3. **Audit trail ชัด** — profile เปลี่ยน = log ใน git ของ `~/.claude/plugins/config/...` หรือ commit ของ matter workspace

## ความสัมพันธ์กับ `<plugin>/CLAUDE.md`

ตอน user install plugin แรก → `/setup` interview จะ:

1. อ่าน `references/company-profile-template.md` เป็น schema
2. ถามทุก field ที่ต้องการ (interactively)
3. เขียน answers ลง `~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md` (filling-in template)

ตอน install plugin ตัวต่อไป → `/setup` จะ:

1. ตรวจว่ามี `CLAUDE.md` ของ plugin อื่นอยู่ไหม
2. ถ้ามี → อ่านมาเป็น context, ถามเฉพาะ delta
3. ถ้าไม่มี → เริ่ม interview ใหม่ตั้งแต่ต้น

ดังนั้น **template นี้คือ "single source of truth"** ที่ทุก plugin คาดหวัง — ถ้าใครเขียน plugin ใหม่ ก็ต้องอ่าน template นี้ก่อนเพื่อรู้ว่าจะ assume context อะไรได้

## หลักการที่ encode ในไฟล์นี้

1. **Default-free design** — skill ไม่ hardcode assumption ทุกอย่างมาจาก profile
2. **First plugin writes, rest read** — เขียนครั้งเดียวใช้ทั่ว ecosystem
3. **Open-ended where it matters** — "What keeps us up at night" เป็น free text เพราะ context ของแต่ละองค์กรไม่เหมือนกัน
4. **Structured where it matters** — "Practice setting" มี enum (`Solo/small | Midsize/large | In-house | Government/legal aid/clinic`) เพราะ skill ต้องการ branch
5. **Pointer ไปยัง per-plugin profile** — บอกว่า "playbooks, house style" อยู่คนละที่ ไม่ปะปนกับ shared facts

## ไฟล์นี้ใน lifecycle

```text
[install plugin แรก] → /setup → interview → กรอก template → เป็น CLAUDE.md ของ plugin
                                                                      │
[install plugin ตัวต่อไป] ────────────────────────────────────┘ ↓
                                                              อ่าน CLAUDE.md เก่า + ถามเฉพาะ delta
                                                                      │
[skill ทุกตัวระหว่าง runtime] ──────────────────────────────┐ ↓
                                                              อ่าน CLAUDE.md เพื่อเอา context
                                                                      │
[user แก้ profile] ──────────────────────────────────────┐ ↓
                                                              edit CLAUDE.md ตรง ๆ หรือ /setup ซ้ำ
```

## ข้อสังเกตจากการอ่าน template

- **`Open regulatory matters:` อาจเป็น "none"** — ไม่บังคับให้ใส่ค่า เพราะหลายองค์กรไม่มี matter ค้างอยู่
- **`The question leadership always asks:`** — field นี้ subtle: บังคับให้ user verbalise "GC ของบริษัทมีคำถามมาตรฐานอะไร" — ผลคือ skill จะรู้ว่าควร emphasize ส่วนไหนของ output (เช่น ถ้าคำถามคือ "Will this trigger BOT notification?" — skill `regulatory-legal/rule-diff` จะ priorities BOT-impact ใน output)
- **`Escalation chain` รองรับ "set per plugin"** — บอกว่า escalate chain ของ commercial-legal อาจไม่เหมือน privacy-legal ได้ — ในนั้นจะ override

อ่านคู่กับ [dashboard](dashboard.html) — dashboard template ก็เป็น cross-plugin standard เดียวกันที่ทุก skill ใช้
