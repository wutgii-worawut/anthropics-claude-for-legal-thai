---
title: Adding Skills
parent: 09 Development
nav_order: 1
---

# วิธีเพิ่ม Skill ใหม่

> "New skill → add it under `<plugin>/skills/<skill-name>/SKILL.md` with the frontmatter the existing skills use. Keep the description under 1024 characters — it's the trigger signal."
> — `README.md`

หมวดนี้สอนวิธี **เพิ่ม skill ใหม่** เข้า plugin ที่มีอยู่ — ตั้งแต่ folder structure, format ของ `SKILL.md`, naming convention, ไปจนถึงตัวอย่าง skill ที่ดีและไม่ดี

## โครงสร้างพื้นฐาน

Skill ทุกตัวอยู่ที่:

```
<plugin>/skills/<skill-name>/SKILL.md
```

ตัวอย่าง:

```
commercial-legal/
└── skills/
    ├── review/
    │   ├── SKILL.md
    │   └── references/                 # ถ้าจำเป็น
    │       ├── playbook-template.md
    │       └── deviation-codes.md
    ├── amendment-history/
    │   └── SKILL.md
    └── renewal-tracker/
        ├── SKILL.md
        └── references/
            └── deadline-schema.json
```

หลักการ: **หนึ่ง skill = หนึ่ง folder** ภายในมี `SKILL.md` หนึ่งไฟล์เป็น entry point + ไฟล์เสริมตามจำเป็น

## SKILL.md format

### ส่วน frontmatter (YAML)

```yaml
---
name: <skill-id>
description: >
  One-sentence description for triggering — รวมประโยคที่ user น่าจะใช้.
  เช่น "Use when user says X, Y, or Z."
  (ต้องน้อยกว่า 1024 ตัวอักษร)
argument-hint: "[optional — สิ่งที่ user ใส่ใน <args>]"
user-invocable: true        # default true; false = skill ใช้ภายในเท่านั้น
---
```

| Field | บังคับ? | ความหมาย |
|-------|---------|----------|
| `name` | ใช่ | identifier ของ skill — invokable เป็น `/<plugin>:<name>` |
| `description` | ใช่ | **trigger signal** สำหรับ Claude — ต้องบอกว่าใช้ตอนไหน, ≤ 1024 chars |
| `argument-hint` | ไม่ | แนะนำว่า user ใส่อะไรหลัง slash command |
| `user-invocable` | ไม่ | `true` = invokable เป็น slash command, `false` = internal helper |

### ส่วน body (markdown)

หลังจาก frontmatter ปิดด้วย `---`:

```markdown
# Invocation Name

(เนื้อหา skill ทีละขั้น — instructions ที่บอกโมเดลว่าจะทำอะไร)

## Inputs
...

## Steps
1. Load practice profile from CLAUDE.md
2. ...

## Output format
...

## Decline pathway
...
```

> ข้อสังเกต: `SKILL.md` เขียนเหมือน "คู่มือพนักงานใหม่" — เป็น natural language ไม่ใช่ code โมเดลอ่านและ follow ตามนั้น

## Naming convention

| สิ่งที่ตั้งชื่อ | กฎ |
|---|---|
| Skill folder name (`<skill-name>`) | lowercase, hyphenated, ใช้กริยา-นาม เช่น `review`, `cold-start-interview`, `amendment-history` |
| `name:` ใน frontmatter | ต้องตรงกับ folder name |
| Slash command | กลายเป็น `/<plugin>:<skill-name>` อัตโนมัติ |
| Reference files | ใน `references/` subfolder, ตั้งชื่อตามเนื้อหา เช่น `playbook-template.md` |

ตัวอย่างที่ดี:

```
clearance/            # /ip-legal:clearance
worker-classification/ # /employment-legal:worker-classification
matter-intake/        # /litigation-legal:matter-intake
```

ตัวอย่างที่ไม่ดี:

```
ReviewVendorAgreement/  # camelCase — ผิด
review_vendor/          # underscore — ผิด
v2-review/              # version ใน name — ผิด (ใช้ version ใน plugin.json แทน)
```

## เลือก folder structure

### Skill เดี่ยวที่ไม่ซับซ้อน

```
<plugin>/skills/my-skill/
└── SKILL.md
```

### Skill ที่มี reference data

```
<plugin>/skills/my-skill/
├── SKILL.md
└── references/
    ├── checklist.md
    ├── jurisdiction-map.json
    └── plausibility-bands.md
```

`references/` คือ **static data** ที่ skill อ่าน — เช่น checklist, schema, lookup table — ไม่ใช่ instruction

### Skill ที่มี sub-skill (nested)

```
<plugin>/skills/review/
├── SKILL.md                          # router skill (user-invocable)
├── vendor-agreement-review/
│   └── SKILL.md                       # internal sub-skill (user-invocable: false)
├── nda-review/
│   └── SKILL.md                       # internal sub-skill
└── saas-msa-review/
    └── SKILL.md                       # internal sub-skill
```

Pattern นี้ใช้ใน `commercial-legal:review` — skill `review` เป็น router ที่ตรวจประเภทเอกสารแล้ว delegate ไป sub-skill ที่เหมาะสม

## Skill description (load-bearing!)

`description:` คือ **trigger signal** — เป็นข้อความที่ Claude อ่านเพื่อตัดสินใจว่าจะเรียก skill นี้ไหม จาก `<system>` prompt ที่ Claude ได้รับ

**Rule of thumb**: เขียนเหมือนกำลังบอกผู้ช่วยใหม่ว่าควรใช้ tool นี้เมื่อไร

### ตัวอย่างที่ดี

```yaml
description: >
  Reviews a vendor agreement, NDA, or SaaS subscription against your playbook
  and produces a redline memo. Use when user says "review this contract",
  "redline this NDA", "check this MSA", or sends a contract file.
  Triggers automatically when a .docx or .pdf with contract-shaped content
  is attached. Routes internally to vendor-agreement-review, nda-review,
  or saas-msa-review based on document type.
```

ทำไมดี:
- บอกชัดว่า skill ทำอะไร
- ระบุ trigger phrase หลายแบบ
- บอก auto-trigger condition
- บอก internal routing

### ตัวอย่างที่ไม่ดี

```yaml
description: "Reviews contracts."
```

ทำไมไม่ดี:
- สั้นเกินไป — Claude ไม่รู้จะใช้เมื่อไร
- ไม่ระบุ trigger phrase
- ไม่บอก scope (ประเภทไหน?)
- ไม่บอก output (memo? markup?)

## ตัวอย่าง skill ที่ดี — `worker-classification`

```yaml
---
name: worker-classification
description: >
  Classifies a proposed engagement (contractor vs employee) against the
  controlling state test. Use when user describes a new engagement and asks
  "is this a contractor or employee?", "1099 or W-2?", "can we treat X as
  independent contractor?", or pastes engagement terms. Handles ABC test
  (CA, MA, NJ), 20-factor test (IRS), economic-reality test (federal).
  Output: classification + risk flag + cite to controlling test.
argument-hint: "[brief description of the engagement or paste terms]"
user-invocable: true
---

# Worker Classification

## Inputs
- Engagement terms (paste, file, or link to Drive doc)
- State of engagement (if not in practice profile)

## Process

1. **Load practice profile** from `CLAUDE.md` — get jurisdictions in scope,
   escalation contact, house risk posture.

2. **Identify controlling test** for the jurisdiction:
   - California, Massachusetts, New Jersey → ABC test
   - Federal tax (1099 issue) → IRS 20-factor test
   - FLSA (wage/hour) → economic-reality test
   - Multi-state engagement → run all applicable tests

3. **Walk each factor** of the controlling test:
   - For ABC: A (free from control), B (outside usual business),
     C (independently established trade)
   - Cite the engagement term that supports/contradicts each factor.

4. **Classify**: contractor / employee / mixed signals.

5. **Flag risk** if classification is borderline OR if engagement
   crosses CA/MA/NJ AND ABC factor B fails.

## Output format

```markdown
# Worker Classification — [Engagement Name]

**Classification**: [Contractor / Employee / Borderline — escalate]

**Controlling test**: [ABC test (CA) / IRS 20-factor / etc.]

## Analysis

### Factor A: Free from control
- ✓ Engagement allows worker to set own hours [cite term §3.2]
- ✗ Engagement requires daily standup [cite term §4.1]
...

## Risk flag
[Y/N — and why]

## Recommendation
[Engage as contractor / Convert to employee / Escalate to [contact]]
```

## Decline pathway

If engagement crosses ≥ 2 jurisdictions with conflicting tests AND
practice profile doesn't specify primary jurisdiction → DO NOT classify.
Output: "Cannot classify without primary-jurisdiction designation;
escalate to [escalation contact]."
```

### ทำไม skill นี้ดี

1. **Doctrine in skill** — ABC test, 20-factor test เขียนชัดใน skill ไม่อาศัย model knowledge
2. **Loads practice profile** — รู้ jurisdiction และ escalation contact
3. **Cites engagement terms** — provenance ติดกับ factual finding ไม่ใช่ paragraph
4. **Risk flag with specific trigger** — บอกชัดว่าเมื่อไรต้อง flag
5. **Decline pathway is a scaffold** — ไม่ใช่ escape hatch, มี condition ชัดเจน

## ตัวอย่าง skill ที่ไม่ดี

```yaml
---
name: helper
description: "Helps with legal stuff."
user-invocable: true
---

# Helper

Answer the user's question about legal matters. Be helpful.
Cite cases when relevant.
```

### ทำไมไม่ดี

1. **Description ไม่ชัด** — Claude ไม่รู้จะ trigger เมื่อไร
2. **Scope กว้างเกินไป** — "legal stuff" คืออะไร? — ไม่มี boundary
3. **ไม่อ่าน practice profile** — ทำงานเหมือน general-purpose chatbot
4. **ไม่มี citation discipline** — "Cite cases when relevant" — เมื่อไรคือ relevant?
5. **ไม่มี decline pathway** — Claude จะตอบทุกอย่าง รวมถึงคำถามที่ตอบไม่ได้
6. **ขาด output format** — ผลลัพธ์จะไม่ consistent

## Workflow การเพิ่ม skill

1. **อ่าน plugin's `CLAUDE.md` ก่อน** — เข้าใจ practice profile structure, integration, decision posture
2. **สร้าง folder** — `<plugin>/skills/<my-skill>/`
3. **เขียน `SKILL.md`** — frontmatter + body
4. **เพิ่ม references** ถ้าจำเป็น — checklist, schema, lookup table
5. **รัน `python scripts/validate.py`** — ตรวจ structural invariant
6. **ทดสอบ skill** — install plugin local, รัน skill, ดู output
7. **Bump version** ใน `<plugin>/.claude-plugin/plugin.json`:
   - Patch (1.0.x) — behavior addition
   - Minor (1.x.0) — new skill / new required input
8. **Commit + PR**

## หลัก "Put doctrine in skill"

จาก `CONTRIBUTING.md`:

> *"If a skill's mode covers patents, cover design patents. If it covers overtime, cover the regular-rate formula. Not a pointer to 'and also think about' — the actual checklist."*

หลีกเลี่ยง:

```markdown
## Step 3
Apply the controlling test. Also think about state-specific variations.
```

ทำแบบนี้แทน:

```markdown
## Step 3 — Apply controlling test

If state is California:
- ABC test (Lab. Code §2750.3)
- Exemptions: Borello list (Dynamex carve-outs)
- AB5 (effective 2020), AB2257 (amended 2020)

If state is Massachusetts:
- ABC test (G.L. c. 149 §148B)
- All three prongs must be met

If state is New Jersey:
- ABC test (N.J.S.A. 43:21-19(i)(6))
...
```

**Doctrine = checklist ที่ skill carry เอง**, ไม่ใช่ pointer ไปยัง guardrail

## Attach provenance tags to numbers, not paragraphs

จาก `CONTRIBUTING.md`:

> *"`[model calculation — verify against the notice clause]` next to the date; `[verify — consult wage-and-hour counsel before asserting or paying]` on the line the back-pay number appears. Tags on surrounding prose get lost; tags on the load-bearing digit do not."*

ทำแบบนี้:

```markdown
Cancel-by date: 2026-06-15 [model calculation — verify against §11.2 notice clause]

Back-pay estimate: $45,250 [verify — consult wage-and-hour counsel before asserting or paying]
```

ไม่ใช่:

```markdown
The notice deadline appears to be in June. Some back-pay may be owed.
[verify all calculations]
```

## Gate header เป็น default-on

จาก `CONTRIBUTING.md`:

> *"If there is an exemption, phrase the heading as the gate and narrow the exemption in a sub-bullet, not the other way around. A load-bearing parenthetical is a bug waiting to be reintroduced by the next edit."*

ทำแบบนี้:

```markdown
## Gate: DO NOT send demand letter

This skill outputs a draft only — never sends.
- Exception: if user explicitly says "send via [channel]"
  AND user has confirmed the final draft in this session
  THEN render send instructions (still not auto-send)
```

ไม่ใช่:

```markdown
## Send the demand letter

(unless this is a draft session, in which case stop after rendering)
```

## ตรวจสุดท้ายก่อน push

| ✓ | คำถาม |
|---|------|
| [ ] | `name:` ตรงกับ folder name? |
| [ ] | `description:` < 1024 chars และมี trigger phrase? |
| [ ] | Skill load practice profile ก่อนทำงาน? |
| [ ] | Doctrine อยู่ใน skill ไม่ pointer? |
| [ ] | Provenance tag ติดกับ number ไม่ paragraph? |
| [ ] | Decline pathway เป็น scaffold ไม่ escape hatch? |
| [ ] | Gate header เป็น default-on? |
| [ ] | `validate.py` ผ่าน? |
| [ ] | Version bumped? |

ขั้นต่อไป — รัน [validation](validation.html) เพื่อตรวจให้แน่ใจ
