---
title: legal-clinic (คลินิกกฎหมายในมหาวิทยาลัย)
parent: 03 Plugins
nav_order: 11
---

# legal-clinic — Plugin สำหรับคลินิกกฎหมายในมหาวิทยาลัย

> "Sets up the clinic, onboards students, runs structured intake, tracks deadlines with malpractice-aware caution, and hands off cases at semester end — built within ABA Formal Op. 512."
> — `legal-clinic/.claude-plugin/plugin.json`

## 1. บทนำ

`legal-clinic` คือ plugin สำหรับ **คลินิกกฎหมายของโรงเรียนกฎหมาย** (law school clinic) — สถานที่ที่นักศึกษาทำงานกฎหมายให้ผู้รับบริการจริง (real clients) ภายใต้การดูแลของอาจารย์ผู้สอน (supervising attorney) — เป็น **environment ที่ผสม "การสอน" + "งานจริง"** จึงต้องการ guardrail ที่ละเอียดกว่า plugin นักศึกษา (`law-student`) และต้องการ workflow ที่ละเอียดกว่า plugin ทนายในบริษัท (`commercial-legal`)

### ความต่างจาก `law-student`

| มิติ | `law-student` | `legal-clinic` |
|---|---|---|
| ผู้ใช้ | นักศึกษา 1L/2L/3L คนเดียว | คลินิก = อาจารย์ + นักศึกษาหลายคน + คดีลูกค้าจริง |
| คดี | hypothetical เท่านั้น | **real client matters** |
| Setup | นักศึกษา run cold-start เอง | **อาจารย์เท่านั้น** ที่ run cold-start ได้ นักศึกษา onboard ผ่าน `/ramp` |
| Header | `STUDY NOTES — NOT LEGAL ADVICE` | `[AI-ASSISTED DRAFT — requires student analysis and attorney review]` |
| Cycle | ตามวิชา | **semester-based** — onboard ต้นเทอม, handoff ปลายเทอม |
| Malpractice exposure | ไม่มี | **มี** (deadline ถ้าพลาด = malpractice ของอาจารย์ที่ supervise) |
| ABA Op. 512 | ไม่เกี่ยว | **built within** — กรอบหลักของ plugin |

### ABA Formal Opinion 512 (2024) — กรอบจรรยาบรรณหลัก

ABA Formal Op. 512 คือความเห็นจรรยาบรรณ (Formal Ethics Opinion) ของ American Bar Association ปี 2024 ที่กำหนดว่าการใช้ AI ในงานกฎหมายต้องอยู่ภายในกรอบของ 4 หลักการ:

1. **Competence (ความเชี่ยวชาญ)** — ทนายต้องเข้าใจขีดจำกัดของเครื่องมือ AI ที่ใช้
2. **Confidentiality (ความลับ)** — ไม่ leak ข้อมูลลูกค้า
3. **Supervision (การกำกับดูแล)** — งานของผู้ที่ไม่ใช่ทนาย (non-lawyer assistant) รวมถึง AI ต้องผ่าน supervising attorney ตรวจ
4. **Client disclosure (การแจ้งลูกค้า)** — ในบางกรณีต้องบอกลูกค้าว่าใช้ AI

`legal-clinic` ฝัง 4 หลักนี้ใน **output safeguards** ที่ทุก skill ปฏิบัติ — ไม่เปิดให้ override:

> "Outputs include: AI-assisted label, Confidence indicators `[UNCERTAIN: ...]`, Verification prompts, Ethical reminders calibrated to task: ABA Formal Opinion 512 (2024) established that AI use in legal practice requires competence, supervision, verification, and in some cases client disclosure."
> — `legal-clinic/CLAUDE.md` § Output safeguards

## 2. `.claude-plugin/plugin.json`

```json
{
  "name": "legal-clinic",
  "version": "1.0.2",
  "description": "Sets up the clinic, onboards students, runs structured intake, tracks deadlines with malpractice-aware caution, and hands off cases at semester end — built within ABA Formal Op. 512.",
  "author": {
    "name": "Anthropic"
  }
}
```

| field | ค่า | หมายเหตุ |
|---|---|---|
| `name` | `legal-clinic` | คำสั่ง — `/legal-clinic:client-intake` |
| `version` | `1.0.2` | semantic versioning |
| `description` | ดูข้างต้น | คำสำคัญ: "**malpractice-aware**" (ระวังความรับผิดทางวิชาชีพ) + "**ABA Formal Op. 512**" |
| `author.name` | Anthropic | first-party |

## 3. โครงสร้างโฟลเดอร์

```text
legal-clinic/
├── .claude-plugin/
│   └── plugin.json
├── .gitignore
├── .mcp.json                            ← Slack, GDrive, CourtListener, Courtroom5, Descrybe
├── CLAUDE.md                            ← 31.6K — practice profile + supervision policy
├── README.md
├── deadlines.yaml                       ← case deadline ledger (malpractice-relevant)
├── client-comms/                        ← per-case communication logs (append-only)
│   └── _README.md
├── handoffs/                            ← per-semester case handoff memos
│   └── _README.md
├── hooks/
│   └── hooks.json                       ← {} placeholder
├── references/
│   └── plausibility-bands/              ← deadline plausibility ranges per state
│       ├── CA.md
│       └── IL.md
└── skills/                              ← 17 skill
    ├── build-guide/                     ← supervisor authors per-practice-area guide
    ├── client-comms-log/                ← log client call/email/letter
    ├── client-intake/                   ← structured intake template
    ├── client-letter/                   ← routine client correspondence
    ├── cold-start-interview/            ← professor setup (Part 0 + ethical preconditions)
    ├── customize/                       ← adjust one section without redo
    ├── deadlines/                       ← add / report / mark complete
    ├── draft/                           ← first draft of common docs
    ├── form-generation/                 ← DEPRECATED → /draft
    ├── memo/                            ← IRAC-scaffolded case memo
    ├── plain-language-letters/          ← DEPRECATED → /client-letter
    ├── ramp/                            ← student semester onboarding
    ├── research-start/                  ← research roadmap (leads, not authority)
    ├── semester-handoff/                ← end-of-semester case memos
    ├── status/                          ← case status by audience
    └── supervisor-review-queue/         ← professor review queue
```

**ข้อสังเกตที่สำคัญ**:

- **`client-comms/`** + **`handoffs/`** — โฟลเดอร์ที่ skill เขียนข้อมูลคดีจริง (append-only) — เป็น **client file** จริง ไม่ใช่ skill metadata
- **`deadlines.yaml`** — ledger ของ deadline ทุกคดี (malpractice-relevant)
- **`references/plausibility-bands/`** — กรอบ "deadline สมเหตุผลไหม" ต่อ state (ปัจจุบัน CA, IL) — ไม่ได้คำนวณแทน นักศึกษา แต่จับ gross error
- ไม่มี `agents/` — ไม่มี scheduled / data-triggered agent

## 4. Skills ทั้งหมด

| # | skill | คำสั่ง | สำหรับใคร | สำหรับอะไร |
|---|------|------|------|------|
| 1 | `cold-start-interview` | `/legal-clinic:cold-start-interview` | **อาจารย์เท่านั้น** | Part 0 + ethical preconditions + supervision model + practice-area templates |
| 2 | `build-guide` | `/legal-clinic:build-guide` | อาจารย์ | สร้าง per-practice-area guide (intake questions, pedagogy posture, review gates) |
| 3 | `ramp` | `/legal-clinic:ramp` | นักศึกษาใหม่ | onboarding ครั้งแรกในเทอม — handbook, tools, practice exercises |
| 4 | `client-intake` | `/legal-clinic:client-intake` | นักศึกษา | structured intake template — cross-area issue spotting |
| 5 | `client-comms-log` | `/legal-clinic:client-comms-log` | นักศึกษา | log การติดต่อลูกค้าทุกครั้ง (call/email/text/letter/in-person) |
| 6 | `research-start` | `/legal-clinic:research-start` | นักศึกษา | research roadmap — **leads, not authority** (citation ทุกตัวยังไม่ verify) |
| 7 | `memo` | `/legal-clinic:memo` | นักศึกษา | IRAC-scaffolded case memo — flag research gap |
| 8 | `draft` | `/legal-clinic:draft` | นักศึกษา | first draft เอกสารทั่วไป — asylum app, eviction answer ฯลฯ |
| 9 | `client-letter` | `/legal-clinic:client-letter` | นักศึกษา | จดหมายถึงลูกค้า (routine) — appointment, update, follow-up |
| 10 | `status` | `/legal-clinic:status` | นักศึกษา | case status — แยก audience (ลูกค้า, อาจารย์, internal) |
| 11 | `deadlines` | `/legal-clinic:deadlines --add/--report` | นักศึกษา + อาจารย์ | track deadline — **malpractice-aware** (warning 14, 7, 3, 1 day) |
| 12 | `supervisor-review-queue` | `/legal-clinic:supervisor-review-queue` | อาจารย์ | queue สำหรับตรวจงานนักศึกษาก่อนส่งจริง |
| 13 | `semester-handoff` | `/legal-clinic:semester-handoff` | อาจารย์ + นักศึกษา | end-of-semester memo สำหรับ cohort ใหม่ |
| 14 | `customize` | `/legal-clinic:customize` | อาจารย์ | แก้ profile ทีละจุด |
| 15 | `plain-language-letters` | (deprecated) | — | redirect → `/client-letter` |
| 16 | `form-generation` | (deprecated) | — | redirect → `/draft` |

### ปรัชญา supervision: 3 model

ตอน cold-start อาจารย์เลือก supervision model 1 ใน 3:

| Model | กลไก | เหมาะกับ |
|---|---|---|
| **Formal review queue** | งาน client-facing / court-bound เข้า queue อาจารย์ approve/edit/return + log | คลินิกใหม่ / นักศึกษามือใหม่ / high-stakes |
| **Configurable flags** | trigger เฉพาะ (court filing, deadline, DV/immigration ฯลฯ) ติดป้าย "CHECK WITH [PROFESSOR]" ไม่มี queue | คลินิกขนาดกลาง / ผสม trust + flexibility |
| **Lighter-touch** | AI-assisted label + verification prompt ปกติ ไม่มี gate เพิ่ม | อาจารย์ supervise ผ่าน case round / one-on-one อยู่แล้ว |

> "This is an open design question — no model is 'right.' Depends on student experience, caseload, and how you already supervise. Change by editing this section."
> — `legal-clinic/CLAUDE.md` § Supervision style

### Pedagogy posture (ใน per-area guide)

ใน `build-guide` อาจารย์เลือก posture ต่อ practice area:

- **`guide`** (default) — skill draft โครงสร้าง นักศึกษากรอกเนื้อหา skill ให้ feedback
- **`assist`** — skill ทำงานเต็ม นักศึกษาตรวจ — สำหรับสถานการณ์ที่ต้องเร่ง
- **`teach`** — skill ขอให้นักศึกษา draft ก่อน ตรวจ feedback แล้วค่อยแสดง model — สูงสุดของ pedagogy

## 5. Agents / Hooks

| ประเภท | สถานะ |
|---|---|
| Agents | **ไม่มี** — ไม่มีโฟลเดอร์ `agents/` |
| Hooks (`hooks/hooks.json`) | `{"hooks": {}}` — placeholder ว่าง |

## 6. References / Workspace dirs

### `references/plausibility-bands/`

แต่ละไฟล์ (เช่น `CA.md`, `IL.md`) เก็บ **กรอบเวลาที่สมเหตุผล** สำหรับ deadline แต่ละประเภทใน state นั้น ใช้กับ `/deadlines --add` เพื่อ catch gross arithmetic error

> "These are rough plausibility ranges, not computations. If a student-entered due date falls outside the range, the `/legal-clinic:deadlines --add` flow flags it for re-check. The skill does NOT compute — it catches gross arithmetic errors in the student's own work."
> — `references/plausibility-bands/CA.md`

### `client-comms/<case-id>/log.md`

ไฟล์ **append-only** ทุกการติดต่อลูกค้า — `/client-comms-log` เป็นผู้เขียน รูปแบบ entry:

```markdown
## [YYYY-MM-DD HH:MM] — [in / out] — [medium]

**Who (student):** [name]
**Who (client side):** [client name]
**Duration / length:** [10 min call | 3-paragraph email]

**Summary:** [2-4 sentences]

**Action items:**
- [Item the student owes the client, with deadline]
- [Item the client owes the student]

**Follow-up due:** [date]

**Notes:** [tone, language, anxiety level ฯลฯ]
```

เหตุที่ต้องมี:

1. **Malpractice defense** — "we communicated X on date Y" ต้องมี record
2. **Continuity at handoff** — เทอมหน้า incoming student อ่าน log เข้าใจคดีโดยไม่ต้อง re-interview
3. **Pattern visibility** — voicemail ที่ลูกค้าไม่ตอบ 5 ครั้งในระยะ 6 สัปดาห์ = supervision flag
4. **Client file-retention** — คลินิกมีหน้าที่เก็บ client file ตามกฎ retention

### `handoffs/<YYYY-semester>/<case-id>.md`

ผลลัพธ์ของ `/legal-clinic:semester-handoff` — เอกสาร handoff ต่อคดี เนื้อหา:

- Case summary (facts, posture)
- Outgoing student + ความสัมพันธ์กับลูกค้า
- Pending deadlines (จาก `deadlines.yaml`)
- Open issues / decisions pending
- Communications history (จาก `client-comms/<case-id>/log.md`)
- Documents drafted / filed
- What incoming student needs to know first
- Professor flags

### `deadlines.yaml`

ledger ของ deadline ทุกคดี — schema ระบุ field สำคัญ:

```yaml
- id: smith-eviction-response-2026-03
  case_id: smith-2026-001
  case_name: Smith v. Property Mgmt
  practice_area: housing
  type: response                          # filing / hearing / SOL / discovery ...
  description: File answer to unlawful detainer
  due_date: 2026-03-15
  source: Court order dated 2026-03-01
  owner_student: Jane Doe
  owner_attorney: Prof. Smith
  status: upcoming                        # upcoming / today / overdue / completed / closed
  created: 2026-03-01
  created_by: Jane Doe
```

หมายเหตุจาก source comment:

> "Missed deadlines in clinic work are malpractice exposure for the supervising attorney. Treat this file as the operational record."

### MCP (`.mcp.json`)

- **CourtListener** — federal docket + opinion
- **Courtroom5** — self-represented litigant workflow (สำคัญสำหรับคลินิกที่บริการ pro se)
- **Descrybe** — case law research
- **Google Drive** — เก็บ matter file
- **Slack** — ใช้ภายในคลินิก

## 7. Workflow ตัวอย่าง

### Workflow 1: Semester onboarding + cohort cycle

```text
Phase A — ต้นเทอม (อาจารย์)
1. /legal-clinic:cold-start-interview
   → ระบุ: clinic name, school, practice areas (immigration + housing),
            supervision model (formal review queue),
            อัพโหลด handbook + local court rules + intake form template
2. /legal-clinic:build-guide immigration
3. /legal-clinic:build-guide housing
   → per-area guide ที่กำหนด intake questions, pedagogy (default = guide),
      review gate (asylum app → require review; routine letter → student-direct)

Phase B — ต้นเทอม (นักศึกษาใหม่)
4. /legal-clinic:ramp
   → 20-min walkthrough: clinic context, commands, fake intake practice,
      verification habits, ABA Op. 512 reminders

Phase C — รับคดี (นักศึกษา)
5. /legal-clinic:client-intake
   → structured intake — cross-area issue spotting
      (eviction case อาจมี domestic violence layer)
6. /legal-clinic:deadlines --add "file answer 2026-03-15"
   → ระบบเช็คกับ plausibility-bands/CA.md → "ภายในกรอบ 30 days" ผ่าน
7. /legal-clinic:client-comms-log
   → log call กับลูกค้าทุกครั้ง
8. /legal-clinic:research-start eviction-grounds
   → research roadmap — citation ติด [model knowledge — verify] ทั้งหมด
9. /legal-clinic:memo
   → IRAC memo — research gap flagged เป็น [review]
   → ระบบติด [AI-ASSISTED DRAFT — requires student analysis and attorney review]
10. /legal-clinic:draft answer-template
    → first draft — student review + edit + queue ให้อาจารย์

Phase D — supervisor review
11. /legal-clinic:supervisor-review-queue
    → อาจารย์เห็น queue งานทั้งหมดที่ flagged require review
    → approve / edit / return พร้อม comment
12. /legal-clinic:client-letter sent-confirmation
    → ส่งจริงเฉพาะหลัง approval

Phase E — ปลายเทอม (อาจารย์ + นักศึกษา)
13. /legal-clinic:semester-handoff
    → memo ต่อคดี → handoffs/2026-spring/<case-id>.md
14. cohort ใหม่ — กลับ Phase B
```

### Workflow 2: เคสที่มี malpractice exposure สูง

```text
1. นักศึกษารับเคส eviction (housing) — โอ้นทั่วไป
2. ตอน /client-intake พบ "client said landlord changed locks"
   → cross-area issue: criminal trespass? domestic violence indicator?
   → ระบบ flag [review] หลายจุด ส่ง queue ให้อาจารย์
3. /deadlines --add — student ใส่ "3 weeks from notice"
   → ระบบเช็ค plausibility-bands → "CA eviction response = 5 days, ไม่ใช่ 3 weeks"
   → ระบบ refuse และให้ student verify กับ CCP § 1167
4. อาจารย์เห็น flag → discuss with student → correct deadline
5. คดีไปถึง court filing → /draft + queue ให้อาจารย์
6. อาจารย์ approve → ส่ง court → /client-letter ส่ง update ลูกค้า
7. /client-comms-log บันทึก call ลูกค้าทุกครั้งตามมาตรฐาน retention
```

## 8. ข้อควรระวัง

### 1. อาจารย์เท่านั้นที่ run cold-start

ตามที่ระบุใน `CLAUDE.md`:

> "Setup must be run by the supervising attorney. Students onboard via `/legal-clinic:ramp`. Clinic clients are not plugin users."

ถ้านักศึกษา run `cold-start-interview` ระบบจะ refuse + redirect ไป `/ramp`

### 2. `[AI-ASSISTED DRAFT]` label คือ canonical header

แทนที่จะใช้ `PRIVILEGED & CONFIDENTIAL — ATTORNEY WORK PRODUCT` แบบ plugin อื่น — `legal-clinic` ใช้:

`[AI-ASSISTED DRAFT — requires student analysis and attorney review]`

เพราะ:

1. งานเป็น **student work ภายใต้ supervised practice** ไม่ใช่ attorney-directed work product
2. ต้องสัญญาณว่ามี supervision step ที่ยังไม่เกิดอย่างชัดเจน

**Strip header** จาก deliverable ภายนอก (จดหมายถึงลูกค้า, court filing) **เฉพาะ** หลังผ่าน supervisor review เท่านั้น

### 3. Non-US jurisdiction = work product label เปลี่ยน

ถ้าคลินิกรับเคสที่แตะ non-US jurisdiction — label เปลี่ยนเป็น:

`CONFIDENTIAL — INTERNAL LEGAL ANALYSIS — NOT A SUBSTITUTE FOR EXTERNAL COUNSEL ADVICE`

เพราะ FRCP 26(b)(3) ไม่มีผลกับ EU/UK/เยอรมนี/ฝรั่งเศส

### 4. Deadline = malpractice exposure

> "Missed deadlines in clinic work are malpractice exposure for the supervising attorney. Treat this file as the operational record."
> — `deadlines.yaml` header

`/deadlines --add` จะเช็ค plausibility band ก่อน accept — student ที่ใส่ผิด (เช่น ใส่ "30 days" สำหรับ CA unlawful detainer ที่จริงคือ 5 days) จะถูก refuse และต้อง verify จากต้นทาง

### 5. Cross-skill severity floor

ถ้า `client-intake` flag เคสว่า 🔴 Blocking (DV indicator + immigration status) — skill ต่อเนื่อง (`memo`, `draft`, `status`) **ห้าม demote** ลง 🟡/🟢 โดยเงียบ ถ้าจะลดต้องระบุเหตุผลที่อาจารย์เห็นได้

### 6. Real client = ห้ามนักศึกษาตอบ "advice"

แม้ skill จะช่วยร่าง — เส้นแบ่ง legal advice อยู่ที่ student ห้าม "ให้คำปรึกษา" ลูกค้าโดยตรง — ต้องผ่านอาจารย์ก่อนเสมอ (Student practice rule ของ jurisdiction จะกำหนด เช่น Cal. Rules of Court 9.42)

### 7. `client-comms/<case-id>/log.md` = append-only

ห้ามแก้ entry เก่า ถ้าผิดให้ entry ใหม่อ้างถึง entry เก่า เพราะ log นี้เป็น **client file part** — rewriting ทำให้ malpractice defense พังและละเมิด client file retention

### 8. `research-start` = leads, not authority

> "/research-start produces leads, not authoritative citations. Every citation is explicitly unverified until the student confirms it."
> — `legal-clinic/CLAUDE.md` § Output safeguards

นักศึกษาต้อง verify citation ทุกตัวก่อนใช้ — เป็นทั้งจรรยาบรรณ และเป็น pedagogical feature (สอนให้ research จริง)

### 9. Practice profile กับ company profile

`legal-clinic` อ่าน 2 ไฟล์ก่อนทุก skill:

- `~/.claude/plugins/config/claude-for-legal/company-profile.md` — clinic + school + state (shared กับทุก plugin)
- `~/.claude/plugins/config/claude-for-legal/legal-clinic/CLAUDE.md` — supervision style + practice areas + supervisor list

ถ้าทั้งสองยังมี `[PLACEHOLDER]` skill หยุดทำงานทันที (ยกเว้น `cold-start-interview`)

### 10. Semester handoff = client continuity = legal duty

> "The incoming student can read the log and know the client's story without re-interviewing."
> — `client-comms/_README.md`

การไม่ทำ handoff = ลูกค้าต้องเล่าเรื่องใหม่ตั้งแต่ต้น = client harm + supervision failure — ดังนั้น `/semester-handoff` เป็น **mandatory** ใน workflow ปลายเทอม

---

> "The clinic loses its entire workforce every semester and rebuilds from scratch. New students need to learn procedures, case management, filing conventions, and practice-area basics before they're useful. This skill is the guided walkthrough."
> — `skills/ramp/SKILL.md` § Purpose
