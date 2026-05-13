---
title: law-student (นักศึกษากฎหมาย)
parent: 03 Plugins
nav_order: 10
---

# law-student — Plugin สำหรับนักศึกษากฎหมาย

> "Drills Socratically, briefs cases, builds outlines, runs bar prep sessions tuned to your jurisdiction, grades IRAC practice, and plans the study schedule — without ever writing it for you."
> — `law-student/.claude-plugin/plugin.json`

## 1. บทนำ

`law-student` คือ plugin สำหรับ **นักศึกษากฎหมาย** (law student) ที่ต้องการ "ครูฝึก" ส่วนตัว — ผู้คอยตั้งคำถามแบบ Socratic ตรวจ IRAC ช่วยสร้าง outline ของวิชา และจัดตารางอ่านสำหรับสอบเนติบัณฑิต (bar exam) ตาม jurisdiction ที่ตั้งเป้าไว้

จุดเด่นที่แยก plugin นี้ออกจาก plugin อื่น ๆ ใน `claude-for-legal`:

| มิติ | plugin งานจริง (e.g., commercial-legal) | `law-student` |
|---|---|---|
| ผลลัพธ์ | draft, memo, review ที่ทนายเอาไปใช้ได้ | **STUDY NOTES — NOT LEGAL ADVICE** |
| โหมด AI | drafting / authoring | **scaffolding** (ตั้งคำถาม ไม่เขียนแทน) |
| โหมดผู้ใช้ | ทนายความที่ใช้งานจริง | นักศึกษา 1L / 2L / 3L / LLM / กำลังเตรียมสอบเนติ |
| Header | `PRIVILEGED & CONFIDENTIAL — ATTORNEY WORK PRODUCT` | `STUDY NOTES — NOT LEGAL ADVICE` |

### ปรัชญา "refuse to write-for-students"

เอกสาร `law-student/CLAUDE.md` ระบุชัดว่า output ของ plugin นี้ **ไม่ใช่งานทนายความ ไม่ใช่ legal work product** เป็น **study material** เท่านั้น เหตุผลหลัก 2 ข้อ:

1. **ไม่มีทนายความที่ "directed" งานนี้** การติด header `ATTORNEY WORK PRODUCT` ลงไปจะเป็นการ "misstate" — ลวงให้คิดว่ามีการคุ้มครองทาง privilege ทั้งที่ไม่มี
2. **doctrine "attorney work product"** มาจาก FRCP 26(b)(3) ของสหรัฐฯ — เกิน jurisdiction ออกไป (EU, เยอรมนี, ฝรั่งเศส, UK) ไม่มี doctrine เทียบเท่า การติด header แบบนั้นจะให้ false assurance ที่อันตรายกว่าไม่ติด

> "A student preparing for a non-US bar should never apply a US work-product header to their notes and assume it means anything. `STUDY NOTES — NOT LEGAL ADVICE` is the honest label regardless of jurisdiction."
> — `law-student/CLAUDE.md` § Outputs

หลัก scaffolding-not-authoring แสดงออกใน skill ทุกตัว เช่น `socratic-drill` ที่ระบุไว้แบบ no-compromise:

> "This skill does not give answers. It asks questions. If you want answers, there's a different tool."
> — `skills/socratic-drill/SKILL.md`

แม้นักศึกษาจะ "stuck" หลายรอบ skill ยังจะไม่บอกเฉลย — เพียงแต่ narrow คำถามให้ลึกขึ้น และหากนักศึกษาตอบไม่ได้จริง ๆ จะส่งกลับไปอ่าน casebook ไม่ใช่บอกคำตอบ

## 2. `.claude-plugin/plugin.json`

```json
{
  "name": "law-student",
  "version": "1.0.2",
  "description": "Drills Socratically, briefs cases, builds outlines, runs bar prep sessions tuned to your jurisdiction, grades IRAC practice, and plans the study schedule — without ever writing it for you.",
  "author": {
    "name": "Anthropic"
  }
}
```

| field | ค่า | หมายเหตุ |
|---|---|---|
| `name` | `law-student` | ใช้เรียก skill — `/law-student:socratic-drill` |
| `version` | `1.0.2` | semantic versioning |
| `description` | ดูข้างต้น | หางประโยค "without ever writing it for you" คือคำประกาศปรัชญาที่ฝัง manifest ไว้ |
| `author.name` | Anthropic | first-party — Anthropic เป็นผู้ดูแล |

## 3. โครงสร้างโฟลเดอร์

```text
law-student/
├── .claude-plugin/
│   └── plugin.json
├── .gitignore
├── .mcp.json                          ← Slack, Google Drive, CourtListener, Descrybe
├── CLAUDE.md                          ← practice profile template (34.5K — ใหญ่)
├── README.md
├── hooks/
│   └── hooks.json                     ← {} (placeholder ว่าง)
└── skills/                            ← 13 skill
    ├── bar-prep-questions/
    ├── case-brief/
    ├── cold-call-prep/
    ├── cold-start-interview/
    ├── customize/
    ├── exam-forecast/
    ├── flashcards/
    ├── irac-practice/
    ├── legal-writing/
    ├── outline-builder/
    ├── session/
    ├── socratic-drill/
    └── study-plan/
```

**สังเกต**: ไม่มีโฟลเดอร์ `agents/` (ไม่มี scheduled / data-triggered agent) ไม่มี `references/` (ไม่ใช้ playbook ภายนอก) — ทุก doctrine อยู่ใน SKILL.md ของแต่ละ skill เอง

## 4. Skills ทั้งหมด

| # | skill | คำสั่ง | สำหรับ |
|---|------|------|------|
| 1 | `cold-start-interview` | `/law-student:cold-start-interview` | สัมภาษณ์ครั้งแรก — ปี, school, target bar, learning style (drill-me vs explain-to-me), past materials |
| 2 | `socratic-drill` | `/law-student:socratic-drill [topic]` | ครูฝึก Socratic — ตั้งคำถาม push back ไม่ให้คำตอบ |
| 3 | `case-brief` | `/law-student:case-brief [case]` | brief case ตามรูปแบบที่ตั้งไว้ — drill-me mode ให้นักศึกษาระบุ holding ก่อน |
| 4 | `outline-builder` | `/law-student:outline-builder` | สร้าง / ต่อ outline — scaffold เฉย ๆ ไม่เขียน outline ให้ทั้งหมด |
| 5 | `irac-practice` | `/law-student:irac-practice [file]` | ตรวจ IRAC essay — structure, issue-spotting, rule accuracy แต่ **ไม่เขียนใหม่ ไม่แสดง model answer** |
| 6 | `legal-writing` | `/law-student:legal-writing [file]` | structural feedback — memo / brief / paper / exam essay — never rewrites |
| 7 | `cold-call-prep` | `/law-student:cold-call-prep` | คาดเดาคำถามที่อาจารย์จะถามใน class แล้วซ้อม Socratic |
| 8 | `bar-prep-questions` | `/law-student:bar-prep-questions` | คำถาม MBE / essay สำหรับเตรียมสอบ bar — ปรับตาม jurisdiction และ weak areas |
| 9 | `flashcards` | `/law-student:flashcards` | สร้าง / drill flashcards — Leitner-style buckets |
| 10 | `exam-forecast` | `/law-student:exam-forecast` | วิเคราะห์ past exam ของอาจารย์คนเดิม — หา pattern, recurring issue-spot traps |
| 11 | `study-plan` | `/law-student:study-plan` | จัดตารางอ่าน bar prep ระยะยาว |
| 12 | `session` | `/law-student:session` | run N-question focused session — track performance อัพเดต study plan |
| 13 | `customize` | `/law-student:customize` | แก้ practice profile ทีละจุด — ไม่ต้อง re-run cold-start ทั้งหมด |

### หลักการเฉพาะ skill ที่สำคัญ

**`socratic-drill` — "Real-matter check"** — ก่อนเริ่ม drill ทุกครั้ง skill จะตรวจว่าคำถามที่นักศึกษาถามเป็นเรื่องสมมุติ (hypothetical) หรือเรื่องจริงในชีวิต (real situation) — ถ้ามี real name, address, deadline เป็นวัน, dollar amount จริง — skill จะ **หยุดทันที** แล้วบอกให้ไปหาทนายจริง (legal aid, school clinic, bar referral service)

> "This sounds like a real situation, not a hypothetical. I can't give you legal advice, and you can't give it either — you're not a lawyer yet."

**`socratic-drill` — Narrow carve-out (rule contradiction)** — กฎ "ไม่ให้คำตอบ" มีข้อยกเว้น 1 ข้อ: เมื่อนักศึกษาท่องกฎที่ขัดกับ **notes / outline / case brief ของตนเอง** ที่อัพโหลดไว้ skill จะยกข้อความจาก material ของนักศึกษามาชนกับสิ่งที่นักศึกษาเพิ่งพูด:

> "That doesn't match your own notes at [file / outline section] — you wrote [exact quote]. Which is right?"

นี่ไม่ใช่การเฉลย แต่เป็นการ **สอนให้ verify จาก source ของตัวเอง** — skill ที่ transfer ไปสู่การสอบจริง

## 5. Agents / Hooks

| ประเภท | สถานะ |
|---|---|
| Agents | **ไม่มี** — ไม่มีโฟลเดอร์ `agents/` |
| Hooks (`hooks/hooks.json`) | `{"hooks": {}}` — placeholder ว่าง |

ไม่มี scheduled / data-triggered behavior — `law-student` ทำงานแบบ user-initiated เท่านั้น

## 6. References / Workspace dirs

| ทรัพยากร | สถานะ |
|---|---|
| `references/` | **ไม่มี** — ทุก doctrine ฝังใน SKILL.md |
| MCP (`.mcp.json`) | Slack, Google Drive, CourtListener, Descrybe |

**CourtListener + Descrybe** — สำหรับ verify case citation ตอน `case-brief` หรือ `irac-practice`
**Google Drive** — เก็บ outline, flashcard, study material
**Slack** — ใช้ใน group study (ไม่ใช่ core)

หมายเหตุ: skill จะใส่ tag `[model knowledge — verify]` หรือ `[CourtListener]` ตาม **provenance** ที่เกิดจริงในการทำงาน ไม่ใช่ตามที่อยากให้คิด

## 7. Workflow ตัวอย่าง

### Case 1: นักศึกษา 1L เตรียมสอบ contracts

```text
Day 1 — Setup
1. /law-student:cold-start-interview
   → ระบุ: 1L, target bar = California, Drill-me style,
            weak subjects = contracts (consideration, statute of frauds),
            อัพโหลด outline เก่า + past exams ของศ. Smith
2. ระบบเขียน profile ที่
   ~/.claude/plugins/config/claude-for-legal/law-student/CLAUDE.md

Day 2-30 — Daily drill
3. /law-student:socratic-drill contracts
   → "A promises to pay B $100 if B quits smoking. B quits.
      Is this an enforceable contract? Why or why not?"
   → นักศึกษาตอบ → ระบบ push back สามรอบ → ยืนยันเมื่อถูกต้องและ
      ใช้ "consideration" ได้ถูกหลัก ไม่ใช่แค่บอกชื่อ doctrine

Day 31 — ก่อนสอบ midterm
4. /law-student:cold-call-prep
   → อ่าน syllabus + past cases ของศ. Smith
   → คาดเดา cold-call ที่อาจารย์น่าจะถาม week นี้
   → drill ทุกข้อ flag อันที่ไม่มั่นใจ
5. /law-student:irac-practice essay-draft.md
   → ตรวจโครงสร้าง — flag issue ที่ตกหล่น flag analysis ที่ตื้น
   → ไม่เขียนใหม่ให้ ไม่แสดง model answer
```

### Case 2: 3L กำลังเตรียมสอบ bar

```text
Phase 1 — สร้างแผน
1. /law-student:study-plan
   → ระบุ: bar date = July 2026, target = California bar
   → แผน 12 สัปดาห์: weighted ตาม MBE weak subjects

Phase 2 — Daily session
2. /law-student:session
   → 20 ข้อ MBE focused บน week 3 = real property
   → ระบบ track miss pattern เก็บกลับมา drill อีกใน week 5

Phase 3 — Practice essay
3. /law-student:bar-prep-questions
   → 1 essay คำถามแบบ California bar
   → นักศึกษาตอบเอง / ใช้ /law-student:irac-practice ตรวจ
   → flag จุดที่ analysis ตื้น — นักศึกษาเขียนใหม่เอง

Throughout — Adjust
4. /law-student:customize → ปรับ weak subject ตามผล session
```

## 8. ข้อควรระวัง

### 1. โอนไป real-client work ไม่ได้

> "If you're navigating a real legal issue, see a lawyer."
> — `law-student/CLAUDE.md` § Who's using this

ถ้าเป็น **clinical practice** (real client) — ใช้ `legal-clinic` ไม่ใช่ `law-student`
ถ้าเป็น **บุคคลที่ไม่ใช่ law student ที่กำลังจะฟ้องใคร** — `law-student` ไม่ใช่เครื่องมือที่ถูกต้อง

### 2. ใช้ output เป็นงานส่งอาจารย์ไม่ได้

ทุก output ติด `STUDY NOTES — NOT LEGAL ADVICE` — แต่ก่อนเอาไปส่งเป็น graded work ต้อง **เช็ค honor code ของ school + AI policy ของอาจารย์** เพราะหลายโรงเรียนถือว่าใช้ AI เขียน graded essay = academic dishonesty

### 3. Citation ทุกตัวยังไม่ verify

แม้ skill เชื่อม CourtListener / Descrybe ก็ตาม — **ทุก citation ที่ skill ปล่อยออกมา** ที่ไม่ได้มาจาก research tool ในรอบ session นี้จะติด `[model knowledge — verify]` นักศึกษาต้องเช็คกับ casebook / bar prep service จริง ก่อนเชื่อ

### 4. `socratic-drill` จะไม่บอกเฉลย — แม้ตอบไม่ได้

ถ้านักศึกษาตอบไม่ได้หลายรอบ → skill จะส่งกลับไปอ่าน casebook ไม่ใช่บอกคำตอบ — เพราะการบอกคำตอบบน take-home exam = ให้คะแนนแทนนักศึกษา ซึ่งเส้นนี้ skill ไม่ข้าม

> "Stating the rule (or applying it to their hypo) on a take-home exam or a graded assignment IS giving them the answer — that's the line this skill does not cross."

### 5. Jurisdiction ของกฎหมาย bar ต่างประเทศ

`law-student` ออกแบบมาเป็น US-centric — ถ้านักศึกษากำลังเตรียม **bar นอกสหรัฐฯ** (England & Wales, Scotland, Australia, Canada) skill จะใช้ US framework เป็นโครง แต่ติด `[US framework — verify against [jurisdiction] law]` กับทุก doctrine ที่ใช้ — ต้องตรวจกับ local supplement เอง

### 6. ไม่มี privilege protection

output ไม่มี attorney-client privilege ไม่มี work product protection — **อย่า paste real client facts** ของคลินิกลงไป (ใช้ `legal-clinic` plugin) อย่าใช้ทำงานจริงให้ลูกค้าใด ๆ

### 7. Verification log สะสมตามตัว plugin

ถ้านักศึกษาเช็ค citation แล้วถูก / ผิด — บันทึกที่
`~/.claude/plugins/config/claude-for-legal/law-student/verification-log.md`
เพื่อให้ครั้งหน้าไม่ต้อง verify ซ้ำ (อยู่ใน CLAUDE.md § Verification log)

### 8. Practice profile = หัวใจ

ทุก skill จะอ่าน `~/.claude/plugins/config/claude-for-legal/law-student/CLAUDE.md` ก่อนทุกครั้ง — ถ้าไฟล์ยังมี `[PLACEHOLDER]` skill จะ **หยุดทำงาน** (ยกเว้น `cold-start-interview` เอง) — ดังนั้น ขั้นตอนแรกหลัง install คือ run cold-start

---

> "You don't learn law by reading. You learn it by being wrong about it, noticing you're wrong, and fixing it. This skill makes you wrong on purpose, in a safe place, so the exam doesn't."
> — `skills/socratic-drill/SKILL.md` § Purpose
