---
title: litigation-legal — งานคดีและการฟ้องร้อง
parent: 03 Plugins
nav_order: 8
---

# litigation-legal — งานคดีและการฟ้องร้อง

## บทนำ

`litigation-legal` คือ plugin ที่ใหญ่ที่สุดและซับซ้อนที่สุดในตระกูล Claude for Legal — ออกแบบมาเพื่อทนายความที่ดูแล "พอร์ตคดี (matter portfolio)" หลายคดีพร้อมกัน ทั้งแบบ in-house counsel, ทนายในสำนักงานกฎหมาย หรือทนายเดี่ยว

หัวใจของ plugin นี้คือ **"thinking partner ไม่ใช่ matter management system"** — ถ้าองค์กรมี LawVu / SimpleLegal / Onit อยู่แล้ว plugin นี้จะไม่เข้ามาแทนที่ แต่จะวางตัวเป็น "ชั้นการให้เหตุผลเชิงโครงสร้าง (structured reasoning layer)" — ทำหน้าที่ตั้งคำถามที่ถูก, สร้าง draft ที่ถูก format, ปักธงประเด็นที่อาจหลุดสายตา

ทุก output ในที่นี้เป็น **"draft สำหรับทนายตรวจทาน"** ไม่ใช่ "ข้อสรุปทางกฎหมาย" — แต่ละ output มี citation ติดอยู่, มี privilege marker ติดอยู่, และทุก action ที่ irreversible (ส่งจดหมาย, ยื่นฟ้อง) ต้องผ่าน gate ยืนยันก่อน

19 skills + 4 workspace dirs + 1 agent + 7 MCP server = plugin ที่บอกได้ว่าเป็น "flagship" ของชุดนี้

## `.claude-plugin/plugin.json`

```json
{
  "name": "litigation-legal",
  "version": "1.0.2",
  "description": "Manages the litigation portfolio — matters, deadlines, holds, demands, outside counsel — and does the work: claim charts (patent and civil), chronologies, depo prep, privilege logs, brief drafting. Adapts to how you work litigation: in-house, firm, or solo.",
  "author": {
    "name": "Anthropic"
  }
}
```

แปล: "จัดการพอร์ตคดี — คดี, deadline, การกักเก็บเอกสาร (hold), จดหมายเรียกร้อง (demand), ทนายภายนอก — และทำงานจริง: ผังองค์ประกอบคดี (claim chart) ทั้งคดีสิทธิบัตรและคดีแพ่ง, ลำดับเหตุการณ์ (chronology), เตรียมการให้ปากคำ (depo prep), บัญชีเอกสารสงวนสิทธิ์ (privilege log), ร่างคำคู่ความ (brief)"

## โครงสร้างโฟลเดอร์

```
litigation-legal/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json                        # 7 MCP server: Slack, GDrive, Everlaw, TopCounsel, CourtListener, Aurora, Trellis
├── .gitignore
├── CLAUDE.md                        # 50 KB — template practice profile (เขียนใหม่โดย /cold-start-interview)
├── README.md                        # 12 KB — คู่มือผู้ใช้
├── agents/
│   └── docket-watcher.md            # scheduled agent ตรวจสารบบความ
├── skills/                          # 19 skill
│   ├── brief-section-drafter/
│   ├── chronology/
│   ├── claim-chart/
│   ├── cold-start-interview/
│   ├── customize/
│   ├── demand-draft/
│   ├── demand-intake/
│   ├── demand-received/
│   ├── deposition-prep/
│   ├── legal-hold/
│   ├── matter-briefing/
│   ├── matter-close/
│   ├── matter-intake/
│   ├── matter-update/
│   ├── matter-workspace/
│   ├── oc-status/
│   ├── portfolio-status/
│   ├── privilege-log-review/
│   └── subpoena-triage/
├── matters/                         # workspace — พอร์ตคดี
│   ├── _README.md
│   └── _log.yaml                    # ledger รายคดี
├── inbound/                         # workspace — เอกสารขาเข้า
│   └── _README.md
├── demand-letters/                  # workspace — จดหมายเรียกร้องขาออก
│   └── _README.md
└── oc-status/                       # workspace — สถานะรายสัปดาห์
    └── _README.md
```

จุดสำคัญที่ต่างจาก plugin อื่น: มี **4 workspace dir** ที่จะถูกเขียนข้อมูลจริงเข้าไปตามเวลา ไม่ใช่แค่ skill อย่างเดียว

## Skills ทั้งหมด (19 skill)

| Skill | argument-hint | หน้าที่ (ไทย) |
|-------|---------------|---------------|
| `cold-start-interview` | `[--redo \| --check-integrations]` | สัมภาษณ์ตอนติดตั้งครั้งแรก — แตกสาขาตามบทบาท (in-house / firm / solo) และฝั่ง (โจทก์/จำเลย) เขียน practice profile |
| `customize` | `[section]` | แก้ profile ทีละส่วนโดยไม่ต้อง re-run cold-start |
| `matter-intake` | `[ชื่อคดี]` | รับคดีใหม่เข้าพอร์ต — ตรวจ conflict, ประเมิน risk, สร้าง `matter.md` + `history.md` + เพิ่มแถวใน `_log.yaml` |
| `matter-update` | `[slug] [เหตุการณ์]` | บันทึก event ลง `history.md` แบบ append-only + อัปเดต field ใน log |
| `matter-briefing` | `[slug]` | สรุปสถานะคดีเชิงลึก — posture ปัจจุบัน, อะไรเปลี่ยน, deadline ถัดไป, risk re-assessment |
| `matter-close` | `[slug]` | ปิดคดี — บันทึก outcome, exposure สุดท้าย, lesson learned (ไม่ลบ) |
| `matter-workspace` | `<new\|list\|switch\|close\|none> [slug]` | จัดการ workspace สำหรับ firm/solo ที่มีหลายลูกความ |
| `portfolio-status` | `[--all \| --risk=high \| --stale]` | ภาพรวมพอร์ต — risk distribution, deadline 14/30/60 วัน, คดีที่ไม่อัปเดต >30 วัน |
| `demand-intake` | `[title] [--full]` | สัมภาษณ์ก่อนร่าง demand letter — ฝ่าย, ข้อเท็จจริง, มูลฐาน, leverage, BATNA, privilege filter |
| `demand-draft` | `[slug] [--version=N]` | ร่างจดหมายจาก intake — gate ตรวจ FRE 408 / privilege / waiver / admission ก่อนปล่อย .docx |
| `demand-received` | `[path] [--slug=...]` | คัดกรองจดหมายเรียกร้องที่รับเข้ามา — ตรวจสอบความน่าเชื่อถือ, cross-check พอร์ต, เสนอตัวเลือกตอบโต้ |
| `subpoena-triage` | `[path] [--slug=...]` | คัดกรองหมายเรียก/หมายเรียกพยานเอกสาร — จัดประเภท (third-party docs/depo, party, CID, grand jury), ขอบเขต/ภาระ/privilege, framework คัดค้าน |
| `legal-hold` | `[slug] [--issue\|--refresh\|--release\|--status]` | ออก/ต่ออายุ/ปล่อย/รายงานสถานะการสั่งห้ามทำลายเอกสาร (legal hold) |
| `chronology` | `[slug] [--format=working\|sof\|witness-...]` | สร้าง/อัปเดตลำดับเหตุการณ์ — ดึงเหตุการณ์ที่มีวันที่ตามทฤษฎีคดี, ติด significance flag 🔴/🟡/⚪ |
| `claim-chart` | `[--patent\|--civil] [--infringement\|--invalidity\|--review] ...` | สร้างผังองค์ประกอบ — claim chart สิทธิบัตร (infringement/invalidity) หรือ element chart คดีแพ่ง พร้อม pin-cite ทุก cell + gap list |
| `deposition-prep` | `[ชื่อพยาน]` | สร้าง outline เตรียมการให้ปากคำ — ดึงเอกสารของพยาน, จัด topic ตามทฤษฎีคดี, ระบุ impeachment material |
| `privilege-log-review` | `[log-file]` | ตรวจ privilege log ขั้นแรก — แยก obvious priv / obvious not priv / ต้องให้ทนายตรวจ |
| `brief-section-drafter` | `[section]` | ร่างคำคู่ความรายส่วน (statement of facts, argument II ฯลฯ) ตาม house style + case theory |
| `oc-status` | `[--all\|--slug=...\|--no-gmail]` | ร่าง email สอบถามสถานะกับทนายภายนอก (outside counsel) รายสัปดาห์ — markdown + Gmail draft (ถ้ามี MCP) |

ตัวอย่าง frontmatter ของ skill ที่ซับซ้อนที่สุด — `claim-chart`:

```yaml
---
name: claim-chart
description: Build or review an element chart — a patent claim chart (infringement, invalidity, or review) or a civil element chart for any cause of action or defense — with every cell pin-cited and gap detection as the priority output.
argument-hint: '[--patent | --civil] [--infringement | --invalidity | --review] [--claim <n>] [--count <name>] [--target <slug>]'
---
```

จะเห็นว่า `argument-hint` เป็นแบบ multi-flag ที่ดูเหมือน CLI จริง

## Agents

`litigation-legal` มี 1 agent — `docket-watcher`

```yaml
---
name: docket-watcher
description: Scheduled agent that watches court dockets for matters in the active portfolio. Pulls new filings, computes candidate deadlines, cross-references against each matter's history and deliverables, and writes a docket status report.
model: sonnet
tools: ["Read", "Write", "mcp__trellis__*", "mcp__courtlistener__*", "mcp__*__slack_send_message"]
---
```

**หลักการทำงาน**:

1. อ่าน `_log.yaml` หาคดีที่ `status != closed`
2. ดึง filing ใหม่จาก Trellis (ศาลรัฐ) / CourtListener / PACER (ศาลกลาง) ตั้งแต่ครั้งที่ตรวจล่าสุด
3. Map filing type → candidate deadline rule (FRCP, FRAP, local rule)
4. Cross-reference กับ `history.md` ของแต่ละคดี + open deliverable
5. เขียน `out/docket-report-[date].md` + machine-readable `out/deadlines.yaml`
6. โพสต์สรุปลง Slack ตาม channel ที่กำหนดใน CLAUDE.md

**Schedule**:
- Default: รายสัปดาห์
- รายวัน: คดีที่มี hearing ใน 14 วัน, คดีอยู่ใน `trial` / late `discovery`, คดีที่ flag `risk: critical`

**ที่ agent นี้ "ไม่ทำ"** (สำคัญมาก):
- ไม่ลง deadline ในปฏิทินจริง — เป็นแค่ "lead" ที่ทนายต้องตรวจสอบเอง (การพลาด deadline ศาลคือ malpractice)
- ไม่เชื่อ classification ของ filing ตัวเอง — เป็น heuristic
- ไม่ตัดสิน posture
- ไม่ถือว่า "docket เงียบ" = "docket clean"

## Hooks

`litigation-legal` **ไม่มี** hooks directory — ทุก gate และทุก check อยู่ใน skill prompt เอง

## Workspace dirs — สำคัญที่สุดของ plugin นี้

### `matters/` — พอร์ตคดี (case portfolio)

โครงสร้างภายใน:

```
matters/
├── _log.yaml                  # ledger — แถวเดียวต่อคดี, machine-parseable, source of truth
├── _README.md
└── [slug]/                    # เช่น acme-v-us-2026/
    ├── matter.md              # narrative — intake เต็ม + theory + posture
    └── history.md             # append-only event log
```

**`_log.yaml` schema** (ตัด snippet สำคัญ):

```yaml
matters:
  - id: acme-v-us-2026          # slug, stable
    name: Acme v. Us
    type: contract              # contract|employment|ip|regulatory|investigation|product|other
    role: defendant             # plaintiff|defendant|claimant|respondent|investigated
    counterparty: Acme Corp
    jurisdiction: N.D. Cal.
    status: discovery           # threatened|active|discovery|trial|appeal|closed
    risk: high                  # critical|high|medium|low
    materiality: reserved       # reserved|disclosed|monitored|none
    exposure_range: "$2M–$5M"
    outside_counsel:
      firm: Big Firm LLP
      lead: Jane Doe
      email: jane@bigfirm.com
      engagement: signed
    conflicts:
      status: cleared
      cleared_by: Outside Counsel
      cleared_date: 2026-01-15
    legal_hold:
      issued: true
      issued_date: 2026-01-20
      custodians: [smith, jones]
      last_refresh: 2026-04-01
      next_refresh: 2026-10-01
    opened: 2026-01-10
    next_deadline: 2026-06-15
    last_updated: 2026-05-13
    path: matters/acme-v-us-2026/
```

**Slug convention**: lowercase, hyphen, ปีต่อท้าย — `acme-v-us-2026`, `ftc-inquiry-2026`, `employment-smith-2026`

**Lifecycle**:
1. `/matter-intake` สร้างโฟลเดอร์ + แถว
2. `/matter-update` append event ใน `history.md`
3. `/matter-briefing` อ่านสรุปก่อนคุย GC / outside counsel
4. `/matter-close` บันทึก outcome, **ไม่ลบ** — คดีปิดยังอยู่ใน `_log.yaml` เป็น "training set ของวิจารณญาณพอร์ต"

**กฎ append-only**: ถ้า history entry เก่าผิด — ไม่แก้ทับ ให้ append entry ใหม่อ้างอิงและแก้ไข (record ของการแก้ไขสำคัญพอ ๆ กับการแก้ไข)

### `inbound/` — เอกสารขาเข้า

```
inbound/
└── [slug]/
    ├── incoming.pdf           # หรือ .eml / .docx — ต้นฉบับ
    ├── triage.md              # วิเคราะห์: scope, merit, options, recommendation
    └── response-v1.docx       # response draft (v2, v3 ฯลฯ)
```

**Slug pattern**: `[type]-[sender-short]-[yyyy-mm]` เช่น
- `demand-rec-acme-2026-04` (จดหมายเรียกร้องที่รับเข้ามา)
- `subpoena-smith-v-us-2026-04` (หมายเรียกพยานเอกสารจากบุคคลที่สาม)
- `regulator-ftc-inquiry-2026-04`
- `preservation-vendor-2026-04`

**Workflow**: read → triage → decide → respond (หรือ escalate เป็น matter)

**ความสัมพันธ์กับ `matters/`**:
- Inbound เกี่ยวกับ matter เดิม → link ผ่าน `related_matters` ใน `_log.yaml`; ไฟล์อยู่ที่ inbound
- Inbound ที่ควรกลายเป็น matter → สร้าง matter; `matter.md` cross-link กลับมาที่ inbound
- Inbound ที่จบแล้วโดยไม่ต้องเปิด matter → ยังเก็บอยู่ที่ inbound เป็น record

### `demand-letters/` — จดหมายเรียกร้องขาออก (outbound)

```
demand-letters/
└── [slug]/
    ├── intake.md              # context gathering, leverage, BATNA, privilege filter
    ├── draft-v1.docx          # ตัวจดหมาย
    └── checklist.md           # post-send — delivery, copies, calendared deadline, follow-up
```

**Slug pattern**: `[type]-[counterparty]-[yyyy-mm]`
- `payment-acme-2026-04` (เรียกชำระเงิน)
- `ceasedesist-competitor-x-2026-04` (หยุดการกระทำ)
- `breach-supplier-2026-04` (ผิดสัญญา + ระยะเวลาเยียวยา)
- `separation-smith-2026-04` (เลิกจ้าง)
- `preservation-vendor-2026-04` (สั่งสงวนเอกสาร)

**Workflow**:
1. `/demand-intake [title]` → สัมภาษณ์ adaptive (core 8 questions เสมอ + strategic block ถ้า material) เขียน `intake.md`
2. `/demand-draft [slug]` → gate: privilege filter, admission risk, accord-and-satisfaction, **FRE 408** posture, waiver scan, tone, factual accuracy — ทุก gate ต้องผ่านก่อนสร้าง .docx

**กฎ**: ไม่ overwrite draft ที่ส่งไปแล้ว — ถ้าต้องแก้ ให้เริ่ม `draft-v2.docx` (history of versions เป็น record ที่สำคัญ)

### `oc-status/` — สถานะรายสัปดาห์ outside counsel

```
oc-status/
└── [YYYY-MM-DD]/
    ├── _summary.md            # ที่รัน, ที่ข้าม, เหตุผลที่ข้าม
    ├── [slug-1].md            # 1 email draft ต่อ matter
    ├── [slug-2].md
    └── ...
```

**Cadence**: รายสัปดาห์ (จันทร์เช้า) ถ้าตั้ง schedule, หรือเรียกเอง `/oc-status` (all) / `/oc-status --slug=[slug]` (รายคดี)

**ถ้ามี Gmail MCP**: สร้าง Gmail draft ใน inbox; ไฟล์ markdown คือ record ถาวร, Gmail draft คือ action layer

**Housekeeping**: โฟลเดอร์ date เก่าเก็บไป — ลบเกิน 30 วันได้ตามสบาย (record ที่สำคัญย้ายไป `history.md` ของ matter แล้ว)

## References / playbooks

ทุก skill load 2 ไฟล์เป็นบริบทเริ่มต้น:

1. `~/.claude/plugins/config/claude-for-legal/litigation-legal/CLAUDE.md` — practice profile ของผู้ใช้ (เขียนโดย `cold-start-interview`)
2. `~/.claude/plugins/config/claude-for-legal/litigation-legal/matters/[slug]/matter.md` — บริบทคดี (เมื่อ skill เกี่ยวกับคดีเฉพาะ)

Skill บางตัวอ้างถึงไฟล์อ้างอิงภายในเช่น `claim-chart` มี `references/element-templates.md` — รายการ element สำหรับ cause of action / defense คดีแพ่งแบบมาตรฐาน

## Connectors (MCP)

7 MCP server ใน `.mcp.json`:

| Server | บทบาท |
|--------|-------|
| **Slack** | ค้น message, อ่าน channel, หา discussion ในที่ทำงาน |
| **Google Drive** | ค้น/อ่าน/ดึงเอกสารจาก Google Drive |
| **Everlaw** | ค้น/จัดระเบียบ/ดึงเอกสารจาก project ใน Everlaw (ediscovery platform) พร้อม metadata + keyword + review link |
| **TopCounsel** | คำแนะนำ outside counsel จาก The L Suite — ชุมชน in-house counsel 5,000+ คน, sentiment, ranking, expertise evidence |
| **CourtListener** | Free Law Project — opinion ศาลสหรัฐหลายล้านชิ้น, PACER docket, profile ผู้พิพากษา, oral argument, citation verification |
| **Aurora** | Consilio ediscovery (read-only) — หา matter, list workspace, full-text search, AI cross-matter investigation, ทุก record cite กลับไปต้นฉบับ |
| **Trellis** | ชุดข้อมูล state trial court ที่ใหญ่ที่สุดในสหรัฐ — docket, ruling, verdict, filing, analytics ผู้พิพากษา/ทนายฝั่งตรงข้าม, vetting พยานผู้เชี่ยวชาญ |

`recommendedCategories`: `ediscovery, legal-research, case-law, court-analytics, outside-counsel-network, documents, chat`

## Workflow ตัวอย่าง

### Case 1: คำฟ้องใหม่เข้า → ผ่านกระบวนการเต็มไปจนร่างคำคู่ความ

1. **รับคำฟ้อง (complaint)** — `/litigation-legal:subpoena-triage [path-to-complaint]` หรือกรณีฟ้องจริงก็เริ่มที่ matter-intake โดยตรง
2. **เปิดคดีในพอร์ต** — `/litigation-legal:matter-intake "Acme v. Us"` → กรอกข้อมูล identification, conflict check, source, risk triage, materiality, outside counsel, internal owner, legal hold, key date → เขียน `matters/acme-v-us-2026/matter.md` + `history.md` + append `_log.yaml`
3. **สั่ง legal hold** — `/litigation-legal:legal-hold acme-v-us-2026 --issue` → ระบุ scope, custodian, date range, system → ร่าง `legal-hold-v1.docx` + ตั้ง `next_refresh` (default +6 เดือน)
4. **สร้างลำดับเหตุการณ์** — `/litigation-legal:chronology acme-v-us-2026` → ดึงเหตุการณ์มีวันที่จาก document source ที่ประกาศ → de-dupe → tag significance ตามทฤษฎีคดี → เขียน `chronology.md`
5. **สร้างผังองค์ประกอบ** — `/litigation-legal:claim-chart --civil --count breach-of-contract --target acme-v-us-2026` → mapping element vs. evidence + gap list (สิ่งที่ยังขาดเพื่อพิสูจน์)
6. **เตรียมให้ปากคำ (deposition)** — `/litigation-legal:deposition-prep "John Smith"` → ดึงเอกสาร authored by / mentioning Smith จาก Everlaw → outline: background, key doc, topic ตามทฤษฎี, impeachment material
7. **ตรวจ privilege log** — `/litigation-legal:privilege-log-review [log-file]` → แยก obvious priv / obvious not priv / needs attorney review → ทนายตรวจทุก flag ก่อน production
8. **ร่างคำคู่ความ (brief)** — `/litigation-legal:brief-section-drafter "statement of facts"` → ร่างตาม house style + case theory + cite ทุก fact + check ทุก case
9. **อัปเดตเป็นระยะ** — `/litigation-legal:matter-update acme-v-us-2026 "MTD ruled — denied"` → append `history.md` + อัปเดต `last_updated`
10. **เมื่อคดีปิด** — `/litigation-legal:matter-close acme-v-us-2026` → บันทึก outcome (settled / dismissed / judgment for-against / withdrawn / consolidated), final exposure, lesson learned

### Case 2: รับ subpoena จากบุคคลที่สาม → คัดกรอง → ตัดสินใจตอบโต้

1. **รับ subpoena** — `/litigation-legal:subpoena-triage ~/Downloads/subpoena.pdf`
2. **Classify** — third-party docs / third-party depo / party / CID / grand jury → ถ้า grand jury หยุดทันที, escalate ตาม `CLAUDE.md`
3. **Cross-check พอร์ต** — เทียบกับ `_log.yaml` หา matter ที่เกี่ยวข้อง (same counterparty, overlapping subject)
4. **วิเคราะห์ขอบเขต** — scope, burden, privilege, objection framework
5. **เขียน triage.md** — `inbound/subpoena-xyz-2026-05/triage.md` พร้อม compliance plan + deadline calendar
6. **Handoff**:
   - ถ้ายังไม่มี hold → `/legal-hold acme-v-us-2026 --issue`
   - ถ้า materiality สูงพอ → `/matter-intake` (pre-populated)
   - ถ้าเป็น party subpoena ใน matter เดิม → `/matter-briefing [slug]`

### Case 3: รับ demand letter → ตัดสินใจ → ตอบโต้ด้วย counter-demand

1. **รับ demand** — `/litigation-legal:demand-received ~/Downloads/demand.pdf`
2. **Extract field** — ฝ่าย, ข้อเรียกร้อง, ฐานกฎหมาย, ระยะเวลา, threat
3. **Cross-check `_log.yaml`** — มี matter เดิมกับ counterparty นี้ไหม
4. **Assess merit** — ความน่าเชื่อถือของข้ออ้าง, leverage จริงของฝ่ายตรงข้าม
5. **เสนอ options** — comply / partial / counter / ignore (พร้อม recommendation)
6. **เขียน `inbound/demand-rec-acme-2026-04/triage.md`**
7. **เลือก counter-demand**:
   - `/litigation-legal:demand-intake "Counter-demand to Acme"` → สัมภาษณ์ context, leverage, BATNA, privilege filter
   - `/litigation-legal:demand-draft [slug]` → gate ผ่านครบ (privilege, FRE 408, waiver, admission, tone, factual accuracy) → ออก `draft-v1.docx` + `checklist.md`
8. **ถ้า material พอ** — `/matter-intake` (pre-populated จาก demand) → เปิด matter ในพอร์ต

## ข้อควรระวัง

### Privilege (สิทธิสงวนเอกสารทนาย-ลูกความ)

- ทุก output ติด work-product header เป็น default — แต่ถ้า user ใส่ข้อมูลที่อาจ waive privilege (เช่น share กับ third party โดยไม่จำเป็น) plugin ไม่รู้ — ทนายต้องตรวจเอง
- `privilege-log-review` ทำเฉพาะ "obvious call" — ทุก flag (close call) ต้องให้ทนายตัดสิน
- `demand-draft` มี **privilege filter gate** — ตรวจว่า draft จะเปิดเผยข้อมูล privileged ใด ๆ ที่ไม่ตั้งใจหรือไม่ก่อนปล่อย

### Work Product

- งานทุกชิ้นใน `matters/[slug]/` ถือเป็น attorney work product — ระวังการ share file โดยตรง
- ถ้า matter ปิดแล้ว, คดียังคงเป็น work product (privilege ไม่หมดอายุ)
- การ delete ไฟล์ใน `matters/` ผิดหลัก "Nothing is Deleted" — ใช้ `/matter-close` ซึ่ง archive (`matters/_archived/`) แทน

### Deadline malpractice

- **`docket-watcher` agent ไม่ใช่ docketing system** — เป็นแค่ "lead surface" ทนายต้องตรวจ deadline แต่ละตัวกับ rule จริง ก่อน docket
- การพลาด deadline ศาล = malpractice ที่เห็นชัดที่สุด — plugin ทำหน้าที่ "upstream" ของการตัดสินใจ ไม่ใช่แทน
- Filing type mapping เป็น heuristic — administrative motion อาจถูกอ่านเป็น dispositive motion → deadline rule ผิด → fail
- "Docket เงียบ" ≠ "docket clean" — clerk docket ช้า, minute entry มาหลังเหตุการณ์หลายวัน

### FRE 408 / settlement communication

- `demand-draft` มี **FRE 408 posture gate** — ถ้าจดหมายเป็น settlement communication, plugin จะ flag การยอมรับ (admission) ที่อาจถูกใช้ทำลายเอง
- ทุก demand ที่ material ต้องผ่าน "strategic block" ก่อนปล่อย — ถ้า block ว่าง, `demand-draft` ปฏิเสธทำงาน (กฎที่ฮาร์ดโค้ดใน skill)

### Disclosed-document use restriction

- `privilege-log-review` ตั้งคำถามก่อนเริ่ม: "เอกสารชุดนี้ได้มาจาก disclosure / discovery ในคดีอื่นหรือไม่?" — ถ้าใช่ ใช้ได้เฉพาะใน scope ของ protective order เดิม

### England & Wales — PD 57AC

- `brief-section-drafter` และ `deposition-prep` มี branch พิเศษสำหรับ witness statement ใน Business & Property Courts (CPR-governed) — ต้องเขียนด้วยคำของพยานเอง, ไม่มี argument, ระบุเอกสารที่พยานใช้ refresh ความจำ, มี confirmation of compliance + legal representative's certificate

### "Every output is a draft for attorney review"

- คำนี้ปรากฏใน README — เป็น **disclaimer ที่ฮาร์ดโค้ดใน CLAUDE.md** ทุก skill
- Citation tag ตามแหล่งที่มา (research tool vs. ต้อง verify)
- Privilege marker apply แบบ conservative (over-mark ดีกว่า under-mark)
- Action ที่ irreversible — filing, sending, executing — gate ด้วย confirmation explicit เสมอ

---

## สรุปจุดที่ทำให้ `litigation-legal` พิเศษ

1. **19 skill** มากที่สุดในตระกูล — ครอบคลุม lifecycle คดีตั้งแต่ pre-litigation (demand letter) → intake → discovery (legal hold, privilege log) → fact development (chronology, claim chart) → witness prep (depo prep) → motion practice (brief drafter) → close (matter-close)
2. **4 workspace dir** ที่ออกแบบให้คน + AI ทำงานร่วมกันได้ — human edit ได้, AI append ได้, มี `_README.md` ในทุกโฟลเดอร์อธิบาย convention
3. **`_log.yaml` เป็น single source of truth** — machine-parseable, แต่ human-editable, schema ที่ design ดี (มี comment อธิบายทุก field)
4. **Append-only philosophy** — `history.md` ของแต่ละคดีไม่แก้ทับ; closed matter อยู่ใน `_log.yaml` เป็น "training set"
5. **Gate-driven safety** — ทุก action irreversible (ส่ง demand, ออก hold, draft brief) มี gate ตรวจ privilege/admission/waiver ก่อน
6. **Docket-watcher agent** — agent เดียวที่ทำงานเชิงรุก (proactive) — แต่ระวังตัวเองว่า "ไม่ใช่ docketing system" และ "ไม่ใช่ทนาย"
7. **7 MCP server** — ครอบคลุม ediscovery (Everlaw, Aurora), court research (CourtListener, Trellis), counsel network (TopCounsel), communication (Slack, Drive)

`litigation-legal` คือตัวอย่างที่ดีที่สุดของ **"plugin ที่ไม่แทนคนทำงาน แต่ทำให้คนทำงานเห็นชัดขึ้น"** — เพราะคดีหนึ่งคดีคือชุดของการตัดสินใจร้อยพันรอบ และ plugin นี้ทำหน้าที่ "บอกว่าตัดสินใจอะไรไปแล้ว, ตัดสินใจอะไรค้างอยู่, ตัดสินใจอะไรกำลังจะเข้ามา"
