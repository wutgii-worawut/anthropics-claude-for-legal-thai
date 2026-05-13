---
title: workspace folders เฉพาะ plugin
parent: 02 โครงสร้างโฟลเดอร์
nav_order: 7
---

# Workspace folders เฉพาะ plugin

> *"This folder holds the portfolio. Two layers: `_log.yaml` — the ledger… `[slug]/` — per-matter detail."*
> — `litigation-legal/matters/_README.md`

นอกจากโครงสร้างมาตรฐาน (`skills/`, `agents/`, `hooks/`, `references/`, `CLAUDE.md`) บาง plugin มี **workspace folders** เพิ่มเติม — โฟลเดอร์ที่ skill ใน plugin นั้นใช้เก็บ work product ของ user

หน้านี้ list ทุก workspace folder ที่พบใน repo + บทบาทของแต่ละโฟลเดอร์

## ภาพรวม

```text
litigation-legal/
├── matters/                 ← portfolio of all matters
├── inbound/                 ← incoming demand letters / subpoenas
├── demand-letters/          ← outbound demand letters
└── oc-status/               ← weekly OC status drafts

legal-clinic/
├── client-comms/            ← per-case communication logs
└── handoffs/                ← end-of-semester case handoffs

employment-legal/
└── data/                    ← (placeholder; gitkeep only)
```

## ทำไม plugin ถึงต้องมี workspace folders?

เพราะ skill ใน plugin เหล่านี้ต้อง **เก็บ state ของงานข้าม session** — ไม่ใช่ตอบคำถามจบในที่:

- Litigation: portfolio ของหลาย matter ที่ดำเนินไปหลายปี
- Legal clinic: case ที่ถูก hand over จากนักศึกษารุ่นหนึ่งไปอีกรุ่น
- Employment: data ที่ใช้ runtime (เช่น leave-tracker)

Plugin เช่น `privacy-legal` ที่งานส่วนใหญ่จบใน session เดียว (review DPA, generate PIA) ไม่ต้องมี workspace folder

## `litigation-legal/matters/` — portfolio ของ legal matters

```text
matters/
├── _log.yaml                  # ledger — หนึ่ง row ต่อ matter
├── _README.md                 # คู่มือ
└── [matter-slug]/
    ├── matter.md              # narrative intake + theory + posture
    └── history.md             # append-only event log
```

### Slug convention

```text
acme-v-us-2026
ftc-inquiry-2026
employment-smith-2026
```

ปีอยู่ท้าย → slug ไม่ชน แม้มี matter เดียวกันปีหลัง

### ใครเขียนอะไร

| File | เขียนโดย skill | แก้โดยตรงได้? |
|---|---|---|
| `_log.yaml` | `/matter-intake`, `/matter-update`, `/matter-close` | ได้ แต่ต้องสะท้อนใน history.md |
| `matter.md` | `/matter-intake` ตอน intake; ต่อโดย `/matter-close` | ได้ สำหรับ theory / posture |
| `history.md` | `/matter-intake` seed; `/matter-update`, `/matter-close` append | append-only เท่านั้น |

### Closed matters

> *"Stay here. Don't delete. `/portfolio-status` filters them from active rollups by default; `/portfolio-status --all` includes them. Closed matters are the training set for portfolio judgment."*

→ **ไม่ลบ** — เก็บไว้ เพราะเป็นข้อมูล train สำหรับ portfolio judgment

### Corrections

> *"If a past history entry was wrong, don't edit it. Append a new entry that references and corrects it. The record of the correction is as important as the correction itself."*

→ **append-only philosophy** เหมือน Oracle's Nothing-is-Deleted

## `litigation-legal/inbound/` — incoming legal correspondence

```text
inbound/
├── _README.md
└── [slug]/
    ├── incoming.pdf              # หรือ .eml / .docx — ต้นฉบับ
    ├── triage.md                 # analysis: scope, merit, options
    └── response-v1.docx          # response draft (v2, v3 as iterated)
```

### Slug pattern

`[type]-[sender-short]-[yyyy-mm]`:

- `demand-rec-acme-2026-04` (demand letter received)
- `subpoena-smith-v-us-2026-04` (third-party subpoena)
- `regulator-ftc-inquiry-2026-04`
- `preservation-vendor-2026-04`

### Workflow

| Type | Command | Outputs |
|---|---|---|
| Demand letter received | `/litigation-legal:demand-received [path]` | triage.md + optional response draft |
| Subpoena served | `/litigation-legal:subpoena-triage [path]` | triage.md + objections memo |
| Regulator inquiry | *future skill* | |

### ความสัมพันธ์กับ matters

| Scenario | สิ่งที่เกิด |
|---|---|
| Inbound + เกี่ยวกับ matter ที่มีอยู่ | link via `related_matters` ใน `_log.yaml`; file อยู่ใน `inbound/` |
| Inbound + ควรเป็น matter | สร้าง matter; `matter.md` cross-link กลับ `inbound/[slug]/` |
| Inbound + handle จบ ไม่เป็น matter | อยู่ใน `inbound/` เป็น record |

## `litigation-legal/demand-letters/` — outbound demand letters

```text
demand-letters/
├── _README.md
└── [slug]/
    ├── intake.md                  # context, strategy, leverage, privilege filters
    ├── draft-v1.docx              # letter (v2, v3)
    └── checklist.md               # post-send: delivery, copies, deadlines
```

### Slug pattern

`[type]-[counterparty]-[yyyy-mm]`:

- `payment-acme-2026-04`
- `ceasedesist-competitor-x-2026-04`
- `breach-supplier-2026-04`
- `separation-smith-2026-04`
- `preservation-vendor-2026-04`

### Workflow

1. `/litigation-legal:demand-intake [title]` → adaptive intake → `intake.md`
2. `/litigation-legal:demand-draft [slug]` → FRE 408 / privilege checklist → draft + `checklist.md` + offer to create matter

### ทำไมแยกจาก `matters/`

| เหตุผล | คำอธิบาย |
|---|---|
| ไม่ทุก demand รุ่งเป็น matter | small-dollar payment demand ไม่ต้องมี log row |
| Workflow เหมือนกัน | intake → draft → send → checklist |
| Cross-link | matter ที่เกิดจาก demand → `matter.md` link กลับมา; drafting history อยู่กับ letter |

### Versioning

> *"Never overwrite a sent draft. If a letter was sent and needs revision, start `draft-v2.docx`. The history of versions is itself useful record."*

→ versioning เป็น append-only ผ่าน v1, v2, v3 ไม่ใช่ overwrite

## `litigation-legal/oc-status/` — weekly OC status drafts

```text
oc-status/
├── _README.md
└── [YYYY-MM-DD]/
    ├── _summary.md                  # what ran, what was skipped and why
    ├── [slug-1].md                  # email draft per matter
    ├── [slug-2].md
    └── ...
```

### Cadence

Weekly (Monday AM) เมื่อ schedule — register schedule with `/litigation-legal:oc-status --setup-schedule`

หรือ ad-hoc:

- `/litigation-legal:oc-status` (default filter)
- `/litigation-legal:oc-status --slug=[slug]` (one matter)

### Integration

> *"When the Gmail MCP is authenticated, Gmail drafts are also created in the user's inbox. The markdown files are the persistent record; Gmail drafts are the action layer."*

→ ตัว markdown file = source of truth; Gmail draft = action surface

### Housekeeping

> *"Old dated folders accumulate. Nothing needs them after OC has responded and matter history is updated. Feel free to delete older than 30 days."*

→ ปฏิเสธ append-only ในกรณีนี้เพราะข้อมูลสำคัญ "ย้ายไปอยู่ที่อื่นแล้ว" (history.md)

## `legal-clinic/client-comms/` — per-case communication logs

```text
client-comms/
├── _README.md
└── [case-id]/
    └── log.md                     # append-only running log
```

### Format ของ log entry

```markdown
## [YYYY-MM-DD HH:MM] — [in / out] — [medium]

**Who (student):** [name]
**Who (client side):** [client name]
**Duration / length:** [10 min call | 3-paragraph email | 2-page letter]

**Summary:**
[2-4 sentences. Substance + tone.]

**Action items:**
- [Item the student owes the client]
- [Item the client owes the student]

**Follow-up due:** [date if applicable]

**Notes:**
[language used, family dynamic observed, client anxiety level]
```

### ทำไมต้องมี

| เหตุผล | คำอธิบาย |
|---|---|
| **Malpractice defense** | "we communicated X on date Y" — ต้องมี record |
| **Continuity at handoff** | นักศึกษาคนต่อมาอ่าน log → รู้ client's story โดยไม่ต้องสัมภาษณ์ใหม่ |
| **Pattern visibility** | "five voicemails unreturned over six weeks" = supervision flag |
| **Retention obligation** | clinic ของ law school มี file-retention duty |

### NOT contained

- Substantive case analysis (อยู่ใน intake / memo / status files)
- Document drafts (separate case folders)
- Privileged attorney-only notes (clinic's internal case notes)

→ **comms log = factual record of contact; ไม่ใช่ legal work product**

## `legal-clinic/handoffs/` — end-of-semester case handoffs

```text
handoffs/
├── _README.md
└── [YYYY-semester]/                # e.g., 2026-spring
    ├── _summary.md                  # cross-case rollup
    ├── [case-id].md                 # one per active case
    └── ...
```

### Semester folder convention

- `2026-spring`
- `2026-summer`
- `2026-fall`

### Memo content

| Field | คำอธิบาย |
|---|---|
| Case summary | facts, practice area, posture |
| Outgoing student | ชื่อ + ความสัมพันธ์กับ client |
| Pending deadlines | ดึงจาก `deadlines.yaml` |
| Open issues / decisions | ที่ค้างต้องตัดสิน |
| Communications history | ดึงจาก `client-comms/[case-id]/log.md` |
| Documents drafted / filed | pointer ไปไฟล์ |
| What incoming student needs | first steps |
| Professor's flags | special notes (if any) |

### Workflow

1. `/legal-clinic:semester-handoff` — รันโดย professor (หรือ departing students) ~1-2 สัปดาห์ก่อนสิ้น semester
2. Output per-case memo + summary
3. Incoming cohort run `/legal-clinic:ramp` — surface handoff memos สำหรับ case ที่ assign

### Retention

> *"Handoff memos stay on disk. Historical handoffs are useful for the clinic's own record of case transitions and for students looking at how a case evolved across semesters."*

## `employment-legal/data/` — runtime data placeholder

```text
data/
└── .gitkeep                       # empty placeholder
```

ปัจจุบันโฟลเดอร์ว่าง (มีแค่ `.gitkeep`) — เป็น **placeholder** สำหรับ runtime data:

- Leave tracker เก็บ state of open leaves
- Investigation records
- Worker classification cache

ไฟล์จริงถูก gitignore — เพราะมี **PII** (personally identifiable information) เช่นชื่อพนักงาน, ID, leave reason

## สรุปตารางทั้งหมด

| Plugin | Folder | Purpose | Append-only? |
|---|---|---|---|
| litigation-legal | `matters/` | Portfolio of all matters | ✓ (closed matters stay) |
| litigation-legal | `inbound/` | Incoming demand/subpoena/regulator | ✓ |
| litigation-legal | `demand-letters/` | Outbound demand letters | ✓ (v1, v2, v3) |
| litigation-legal | `oc-status/` | Weekly OC status drafts | ✗ (delete >30 days OK) |
| legal-clinic | `client-comms/` | Per-case comm log | ✓ |
| legal-clinic | `handoffs/` | Semester-end handoff memos | ✓ |
| employment-legal | `data/` | Runtime PII data | gitignored |

## หลักการเบื้องหลัง workspace folders

### 1. Separation of concerns

แต่ละ folder ทำหน้าที่เดียว — `matters/` ≠ `inbound/` ≠ `demand-letters/` แม้บางครั้งเรื่องเดียวกันจะข้ามมา 3 folder

### 2. Append-only history

ส่วนใหญ่ — เพราะ legal practice ต้องเก็บ record ของ:

- การติดต่อ (malpractice defense)
- การตัดสินใจ (audit trail)
- การส่ง document (delivery proof)

### 3. Slug convention ที่อ่านได้

`[type]-[counterparty]-[yyyy-mm]` ทำให้:

- เรียงตามเวลาได้
- ค้นหาตาม counterparty ได้
- ไม่ชน slug

### 4. `_README.md` ทุก folder

ทุก workspace folder มี `_README.md` ที่อธิบาย:

- ความหมายของ folder
- slug convention
- workflow (skill ใดสร้าง / append / read)
- relationship กับ folder อื่น
- retention policy

→ **ระบบ self-documenting** — user เปิด folder → อ่าน `_README.md` → เข้าใจทันที

### 5. Cross-link ระหว่าง folder

`matters/[slug]/matter.md` อาจ link ไปยัง `demand-letters/[demand-slug]/` หรือ `inbound/[demand-slug]/`

→ data graph ระหว่าง folder ไม่ใช่ siloed tree

## สรุป

Workspace folders ของ Claude for Legal:

- มีเฉพาะ plugin ที่ skill ต้อง persist state ข้าม session
- **`litigation-legal`** มี 4 folder (matters / inbound / demand-letters / oc-status) — portfolio-heavy
- **`legal-clinic`** มี 2 folder (client-comms / handoffs) — communication + transition
- **`employment-legal`** มี 1 folder (data/) — runtime data (gitignored)
- ใช้ slug convention ที่อ่านได้ + `_README.md` ทุก folder
- หลัก append-only ส่วนใหญ่ (versioning ผ่าน v1, v2, v3)
- Cross-link ระหว่าง folder ทำให้ data graph ครอบคลุม

---

จบหมวด 02 โครงสร้างโฟลเดอร์ — ผู้อ่านที่มาถึงตรงนี้ควรเข้าใจทุก folder ใน `claude-for-legal/` แล้ว และพร้อมที่จะลงลึกที่แต่ละ plugin ในหมวด **03** ต่อไป
