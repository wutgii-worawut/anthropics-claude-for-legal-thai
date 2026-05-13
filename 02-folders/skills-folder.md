---
title: skills/ — หัวใจของทุก plugin
parent: 02 โครงสร้างโฟลเดอร์
nav_order: 4
---

# โฟลเดอร์ `skills/` — หัวใจของทุก plugin

> *"A skill is the unit of work."*
> — sentinel ของทุก plugin ใน Claude for Legal

โฟลเดอร์ `skills/` คือที่ที่ **คำสั่งที่ user เรียก** ทั้งหมดอยู่ — ทุกครั้งที่ user พิมพ์ `/<plugin>:<skill>` ภายใต้ฮูดคือการอ่าน `SKILL.md` ของ skill นั้น ๆ มาทำงาน

## โครงสร้างทั่วไป

```text
<plugin>/skills/
├── skill-1/
│   ├── SKILL.md                ← required: คำอธิบาย + ขั้นตอน
│   └── references/             ← optional: static data
├── skill-2/
│   └── SKILL.md
└── ...
```

**กฎพื้นฐาน**:

1. หนึ่ง skill = หนึ่ง subdirectory
2. ชื่อโฟลเดอร์ = ชื่อคำสั่ง (kebab-case)
3. ต้องมี `SKILL.md` (case-sensitive — ตัวพิมพ์ใหญ่ทั้งหมด)
4. `references/` มีหรือไม่มีก็ได้

## ตัวอย่าง path ของจริง

```text
commercial-legal/skills/nda-review/SKILL.md
commercial-legal/skills/renewal-tracker/SKILL.md
commercial-legal/skills/renewal-tracker/references/
litigation-legal/skills/claim-chart/SKILL.md
litigation-legal/skills/claim-chart/references/
law-student/skills/case-brief/SKILL.md
legal-clinic/skills/client-intake/SKILL.md
legal-clinic/skills/client-intake/references/
```

User เรียกผ่าน:

```text
/commercial-legal:nda-review
/commercial-legal:renewal-tracker
/litigation-legal:claim-chart
/law-student:case-brief
/legal-clinic:client-intake
```

## `SKILL.md` format

ทุก `SKILL.md` มี 2 ส่วน:

```markdown
---
name: skill-name
description: >
  คำอธิบายว่า skill นี้ทำอะไร เมื่อไหร่ user (หรือ Claude) ควรเรียก
user-invocable: true        # หรือ false ถ้าเป็น helper skill
argument-hint: "[args]"     # optional — hint สำหรับ argument
---

# /skill-name

(เนื้อหา — ขั้นตอน, workflow, edge cases, output format)
```

### YAML frontmatter — required fields

| ฟิลด์ | type | required | คำอธิบาย |
|---|---|---|---|
| `name` | string | yes | ต้องตรงกับชื่อโฟลเดอร์ |
| `description` | string (multi-line) | yes | บอก user / Claude ว่าเมื่อไหร่ใช้ — รวม **trigger phrases** ที่ Claude ใช้ตัดสินใจเลือก skill |
| `user-invocable` | boolean | optional | default `true`; ถ้า `false` = skill นี้เรียกจาก skill อื่นเท่านั้น |
| `argument-hint` | string | optional | hint แสดงเวลา user พิมพ์คำสั่ง |

### ตัวอย่างจริง — `commercial-legal/skills/nda-review/SKILL.md`

```yaml
---
name: nda-review
description: >
  Reference: fast triage of inbound NDAs into GREEN / YELLOW / RED so the team only
  spends lawyer time on the ones that need it. Built for sales and BD to self-serve
  before pinging legal. Loaded by /commercial-legal:review when an NDA is detected.
user-invocable: false
---
```

สังเกตว่า `user-invocable: false` — skill นี้ถูกเรียกจาก `/commercial-legal:review` อัตโนมัติเมื่อพบว่า document เป็น NDA ไม่ใช่ user เรียกเอง

### ตัวอย่าง user-invocable skill — `law-student/skills/case-brief/SKILL.md`

```yaml
---
name: case-brief
description: >
  Brief a case in your preferred format. In drill-me mode, makes the student
  state the holding first. Use when the user says "brief [case]", "what's the
  holding in", "case brief", or pastes a case.
argument-hint: "[case name or citation, or paste the case]"
---
```

สังเกต:

- **trigger phrases** อยู่ใน description: *"Use when the user says 'brief [case]'..."*
- **mode hint** ใน description: *"In drill-me mode, makes the student state the holding first"*
- **argument-hint** บอก user format ที่ใส่ได้

## เนื้อหาภายใน SKILL.md (หลัง frontmatter)

โครงเนื้อหามาตรฐาน (อิงจาก SKILL หลายตัวใน repo):

```markdown
# /skill-name

## Matter context
[ตรวจว่าทำงานในระดับ matter หรือ practice-level]

## Destination check
[ก่อนเขียน output ถามว่าส่งไปไหน — privilege protection]

## Purpose
[เป้าหมายของ skill]

## Load context
[อ่าน CLAUDE.md → ดึง user preference]

## The workflow
[ขั้นตอนหลัก — เป็น checklist หรือ numbered steps]

## Output format
[รูปแบบที่ส่งกลับ user]

## What this skill will not do
[ขอบเขตที่ปฏิเสธ — เช่น "won't write a brief without case text"]

## Edge cases
[จัดการ scenario ที่ผิดปกติ]
```

ไม่ใช่ทุก SKILL ต้องมีทุกหัวข้อนี้ — แต่ pattern นี้มาจาก `CONTRIBUTING.md` ที่ว่า **SKILL.md ต้องบอกตรง ๆ ว่าทำอะไร ไม่พึ่ง CLAUDE.md ช่วย**

## Subfolder convention

### `references/` ใน skill folder

ถ้า skill ต้องใช้ static data → ใส่ใน `references/` ของ skill นั้น:

```text
commercial-legal/skills/renewal-tracker/
├── SKILL.md
└── references/
    └── (date format spec, deadline calculation, etc.)

corporate-legal/skills/tabular-review/
├── SKILL.md
└── references/
    └── ma-diligence-columns.md    ← schema สำหรับ column ที่ extract
```

จริง ๆ แล้ว `managed-agent-cookbooks/diligence-grid/agent.yaml` reference ไฟล์นี้ตรง ๆ:

```yaml
# จาก agent.yaml ของ diligence-grid
Default is the M&A diligence standard column set in
`skills/tabular-review/references/ma-diligence-columns.md`.
```

### `references/` ระดับ plugin (ไม่ใช่ระดับ skill)

ถ้า reference ใช้ข้ามหลาย skill → ใส่ที่ระดับ plugin:

```text
legal-clinic/references/
└── plausibility-bands/    ← ใช้โดยหลาย skill เกี่ยวกับ deadline
```

**กฎ**: reference อยู่ใกล้ที่สุดกับ skill ที่ใช้

## Naming convention

จาก skill ทั้งหมดใน repo:

| Pattern | ตัวอย่าง | บทบาท |
|---|---|---|
| `<noun>-<verb>` | `nda-review`, `case-brief`, `dpa-review` | document review |
| `<verb>-<noun>` | `draft`, `review`, `triage` | action ทั่วไป |
| `<noun>-<noun>` | `claim-chart`, `legal-hold`, `study-plan` | artifact production |
| `<noun>-workspace` | `matter-workspace` | workspace setup |
| `cold-start-interview` | (universal) | setup interview ของทุก plugin |
| `customize` | (universal) | แก้ practice profile |
| `matter-*` | `matter-intake`, `matter-update`, `matter-close` | matter lifecycle |

**Reserved names**:

- `cold-start-interview` — ทุก plugin ต้องมี เพื่อให้ user setup ครั้งแรก
- `customize` — ทุก plugin ต้องมี เพื่อให้ user แก้ profile ภายหลัง

## Skill ที่พบในทุก plugin

จากการสำรวจ folder ทั้ง 12 plugins:

| Skill | ทำหน้าที่ |
|---|---|
| `cold-start-interview` | สัมภาษณ์ user → สร้าง `~/.claude/plugins/config/.../CLAUDE.md` |
| `customize` | ให้ user แก้ profile ที่สร้างไว้แล้ว |
| `matter-workspace` (บาง plugin) | enable / disable matter mode |

## ตัวอย่าง skill count ต่อ plugin

| Plugin | จำนวน skills |
|---|---|
| `litigation-legal` | 21 |
| `commercial-legal` | 14 |
| `corporate-legal` | 14 |
| `employment-legal` | 19 |
| `legal-clinic` | 17 |
| `legal-builder-hub` | 10 |
| `law-student` | 13 |
| `ai-governance-legal` | 10 |
| `ip-legal` | 12 |
| `privacy-legal` | 9 |
| `product-legal` | 8 |
| `regulatory-legal` | 11 |
| `cocounsel-legal` (external) | 1 (`deep-research`) |

**ข้อสังเกต**: external plugin ของ Thomson Reuters มี **skill เดียว** เพราะมันเป็น integration กับ Westlaw API — ไม่ใช่ workflow plugin

## วิธีที่ Claude เลือก skill

เมื่อ user พิมพ์อะไรก็ตาม (ไม่ใช่ `/command` ตรง ๆ) Claude จะ:

1. อ่าน `description` ของทุก SKILL.md ใน plugin ที่ install
2. หา trigger phrase ที่ match กับ user input
3. ถ้ามี match → load SKILL.md → ทำงาน
4. ถ้าไม่มี match → ตอบแบบทั่วไป

ดังนั้น **`description` field สำคัญมาก** — เขียนให้ครอบคลุม trigger phrases ที่ user น่าจะใช้

ตัวอย่าง trigger phrases ใน description:

```text
"Use when the user says 'brief [case]', 'what's the holding in', 'case brief',
or pastes a case."
```

## SKILL.md ขนาดทั่วไป

จากการสุ่ม:

| Skill | ขนาด |
|---|---|
| `commercial-legal/skills/nda-review/SKILL.md` | ~8 KB |
| `litigation-legal/skills/claim-chart/SKILL.md` | ~15 KB |
| `law-student/skills/case-brief/SKILL.md` | ~6 KB |
| `legal-clinic/skills/client-intake/SKILL.md` | ~10 KB |

**ค่าเฉลี่ย**: 5–15 KB — เป็น essay ขนาดกลาง ไม่ใช่ one-liner

## หลักการที่อยู่เบื้องหลัง (จาก CONTRIBUTING.md)

> *"The narrow, task-specific scaffold."*

SKILL.md ออกแบบให้:

- **specific** — ไม่ใช่ generic helper
- **narrow** — โฟกัส 1 งาน
- **scaffold** — เป็นกรอบที่ Claude เติม ไม่ใช่ template ตายตัว

## เปรียบเทียบ skills vs agents

| ลักษณะ | `skills/` | `agents/` |
|---|---|---|
| ใครเรียก | user (โดยตรง) | scheduler / agent system |
| Trigger | user คำสั่ง / phrase match | cron schedule / external event |
| Interactive | ✓ | ✗ (มัก background) |
| Output | conversation | file (เช่น log, report) |
| ตัวอย่าง | `nda-review`, `case-brief` | `renewal-watcher`, `docket-watcher` |

ดูรายละเอียด agent → [agents-and-hooks](agents-and-hooks.html)

## สรุป

โฟลเดอร์ `skills/` คือ:

- **หัวใจของ plugin** — ทุกคำสั่ง user เรียกอยู่ที่นี่
- **หนึ่ง skill = หนึ่ง subdirectory + SKILL.md**
- **YAML frontmatter** กำหนด name, description, invocability
- **trigger phrases ใน description** ทำให้ Claude เลือกถูก
- **`references/`** สำหรับ static data — ใกล้ skill ที่ใช้
- **SKILL.md เนื้อหา** ต้องตรง specific (CONTRIBUTING rule)

หน้าถัดไป → [agents-and-hooks](agents-and-hooks.html) จะอธิบาย `agents/` กับ `hooks/` — สองโฟลเดอร์ที่ "ไม่ user เรียก" แต่สำคัญพอ ๆ กัน
