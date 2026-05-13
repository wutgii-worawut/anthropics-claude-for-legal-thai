---
title: dashboard (โครงสร้าง dashboard)
parent: 07 References
nav_order: 2
---

# dashboard-template.md — โครงสร้างมาตรฐานของ dashboard ใน Claude for Legal

> *"Referenced by the Dashboard offer guardrail. Keep dashboards simple and consistent — the value is speed of comprehension, not visual polish."*
> — header ของ template

`references/dashboard-template.md` คือ **rendering guideline** สำหรับ dashboard output ของหลาย skill ใน Claude for Legal — เช่น `commercial-legal/tabular-review`, `corporate-legal/renewal-tracker`, `privacy-legal/entity-compliance`

ไฟล์นี้ไม่ใช่ "design system" ในเชิง UI/UX แต่เป็น **opinionated structure** ที่ guardrail (hook) บังคับใช้ — เพื่อให้ทุก dashboard ของทุก plugin ดูเหมือนกัน เพื่อให้ผู้อ่านรู้ว่าจะหา content อะไรตรงไหนได้เร็ว

## ทำไมต้อง standardize dashboard

ปัญหาที่ template นี้แก้:

1. **ทุก skill ผลิต dashboard ในรูปทรงแตกต่างกัน** = ผู้ใช้ต้องเรียนใหม่ทุกครั้ง
2. **Dashboard ที่ดู visual หรูเกินไป** = ช้าและทำให้ overlook insight
3. **HTML artifact ที่มี attacker-controlled string** = XSS surface (จาก document content ที่ trust ไม่ได้)

Solution ของ Anthropic:

- บังคับโครงสร้าง 6 ส่วน (top → bottom)
- บังคับ rendering rule ตาม surface (Cowork vs Claude Code vs Excel)
- บังคับ HTML escape ทุก attacker-controlled string

## โครงสร้าง 6 ส่วน (top to bottom)

จาก template:

```text
┌────────────────────────────────────────────────────────────────┐
│ 1. Title และ metadata — what + when + scope (one line)        │
├────────────────────────────────────────────────────────────────┤
│ 2. Summary stats — color-coded counts                          │
│    "40 findings: 🔴 3 blocking · 🟠 8 high · 🟡 15 medium ·    │
│     🟢 14 low — 6 due this week"                               │
├────────────────────────────────────────────────────────────────┤
│ 3. The reviewer note — sources, scope, flags, before-relying   │
├────────────────────────────────────────────────────────────────┤
│ 4. Chart(s) — max 2                                            │
│    - Risk distribution (bar) = findings/flags                  │
│    - Category breakdown (pie/stacked bar) = types              │
│    - Timeline (Gantt-lite or table) = dates                    │
├────────────────────────────────────────────────────────────────┤
│ 5. The table — sortable, filterable, color-coded               │
├────────────────────────────────────────────────────────────────┤
│ 6. The decision tree — "What next?"                            │
└────────────────────────────────────────────────────────────────┘
```

หลักการในแต่ละส่วน:

| ส่วน | ค่าใช้สอย | กฎเข้ม |
|---|---|---|
| 1. Title + metadata | ระบุว่าอะไร, เมื่อไหร่, ครอบคลุมแค่ไหน | **บรรทัดเดียว** เท่านั้น |
| 2. Summary stats | บรรทัด "ทรงที่สำคัญที่สุด" ของ dashboard | ต้อง **scannable** — emoji + count + due window |
| 3. Reviewer note | safety metadata (เหมือนทุก output) | **ห้ามข้าม** เพราะ dashboard ก็คือ output |
| 4. Charts | ดู "shape" ของ data | **สูงสุด 2** — มากกว่านั้น = report ไม่ใช่ dashboard |
| 5. Table | data เต็ม | **last column ต้องเป็น details/notes** — เป็นคอลัมน์ที่ตัดได้ |
| 6. Decision tree | คำตอบของ "what next" | options ตาม decision pattern ของ output text |

## Rendering rules ต่อ surface

dashboard ของ Claude for Legal render ต่างกันใน 3 surface:

### Cowork / Claude Desktop — HTML artifact

```markdown
- Self-contained, single file, inline CSS
- No external dependencies, no CDN, no npm
- Tables: HTML <table> with data-sort attributes + small inline JS sorter
- Charts: inline SVG or Unicode block chars
- Keep JS minimal — sorting and filtering, nothing else
```

หลักการคือ **offline-first** — dashboard ที่ break เมื่อไม่มี network = dashboard ที่ break

### Claude Code — HTML + markdown

```markdown
- เขียน HTML ลง ~/.claude/plugins/config/claude-for-legal/<plugin>/outputs/
  dashboard-<topic>-<date>.html
- บอก user ให้ "open <path>" (macOS) หรือ "open in browser"
- **also** produce markdown version with Unicode block charts สำหรับ summary
```

ทำไมต้องมีทั้ง HTML + markdown? เพราะหลายคนใช้ Claude Code ใน terminal — ไม่อยาก context-switch ไป browser แค่เพื่อดู summary

### Excel (optional)

```markdown
- For tabular-review, renewal-tracker, entity-compliance
- Anything ที่ user จะเอาไปคุยใน meeting หรือ share กับ non-tech stakeholder
- Apply formula-injection defense
```

Excel มีไว้สำหรับ case ที่ผู้รับ output ไม่ใช่ legal tech — เช่น product manager, board member, audit committee

## Security — escape attacker-controlled input

ส่วนที่สำคัญมากในเทมเพลตคือ:

> *"Every value that came from outside this session — OSS package/license fields from third-party manifests, counterparty contract text, diligence findings, vendor names, matter descriptions, any user- or VDR-supplied string — must be HTML-escaped before it lands in the document."*

มีรายการชัด:

| Untrusted source | ตัวอย่าง |
|---|---|
| OSS package/license fields | `package.json`, `pom.xml` ของ vendor |
| Counterparty contract text | NDA, vendor agreement ที่อีกฝั่งส่งมา |
| Diligence findings | data room ที่ target company upload |
| Vendor names | จาก vendor onboarding form |
| Matter descriptions | จาก ticket system / VDR |

วิธีจัดการ:

1. HTML escape `& < > " '` → entities ก่อนใส่ใน table cell, summary line, chart label, tooltip
2. ใน inline JS sorter/filter — ใช้ `textContent` ไม่ใช่ `innerHTML`
3. ไม่ emit `<script>` ที่มี untrusted string ใส่ใน body
4. ไม่ render untrusted URL เข้า `href` / `src` ก่อน scheme-check (allow `http:`/`https:`/`mailto:` only)

นี่คือ **HTML-surface equivalent ของ Excel formula-injection defense** — threat model เดียวกัน (attacker-controlled cell content), execution surface ต่างกัน (browser JS vs spreadsheet formula)

## "Keep it boring" rule

template ปิดท้ายด้วย principle:

```markdown
- Color palette: Red / orange / yellow / green for severity. Gray for neutral.
  Blue for status. Nothing else.
- No animations, no frameworks, no external fonts.
- No clever layouts. Summary, reviewer note, chart, table, decision tree.
  Top to bottom. Every dashboard looks the same so the reader knows where to look.
- The markdown version matters. Some users are in a terminal.
  Summary stat line with Unicode bars:
    🔴 ███ 3  🟠 ████████ 8  🟡 ███████████████ 15  🟢 ██████████████ 14
```

หลักการ "boring" ตั้งใจ — เพราะ:

1. **Visual variety = cognitive load** → ผู้ใช้กฎหมายต้องตัดสินใจเร็ว ไม่ใช่ชมงาน design
2. **Frameworks = dependency** → repo legal อาจไม่ออนไลน์ตอน critical moment
3. **Consistency = trust** → ทุก dashboard ดูเหมือนกัน → reviewer มั่นใจว่าไม่มี hidden information

## ตัวอย่างใช้งานจริง

### Use case 1: `commercial-legal/tabular-review` ผลิต NDA review dashboard

```markdown
NDA Review — Counterparty: ACME Corp — Generated 2026-05-13 14:32 UTC
                                                                          (1. Title)

40 redlines: 🔴 3 blocking · 🟠 8 high · 🟡 15 medium · 🟢 14 low — 6 due this week
                                                                       (2. Summary stat)

Reviewer note: ใช้ playbook v3.2; ครอบคลุม clauses 1-22 + Schedule A.
3 blocking redlines ต้องการ GC approval ก่อน sign. Flag: governing law ระบุ Delaware
ทั้งที่ counterparty อยู่ Singapore. Before relying: verify Sec 18.2 IP carve-out
หลัง counterparty redline ครั้งหน้า.                                    (3. Reviewer note)

[bar chart of redline severity]                                          (4. Chart)

| Clause | Severity | Position | Action | Notes |
|---|---|---|---|---|
| 4.1 Indemnity | 🔴 blocking | Reject | Escalate to GC | Counterparty inserted... |
| 7.3 Termination | 🟠 high | Negotiate | ... | ...                       (5. Table)

What next?
- Send markup back to counterparty with our redlines accepted: 14 low-severity
- Escalate 3 blocking + 8 high to GC review
- Pause negotiation until counterparty responds on Sec 4.1                (6. Decision tree)
```

### Use case 2: `corporate-legal/renewal-tracker` — renewal calendar

```markdown
Vendor Renewal Tracker — Q2 2026 — Generated 2026-05-13

25 renewals: 🔴 2 overdue · 🟠 5 due ≤30d · 🟡 8 due ≤60d · 🟢 10 due >60d

[timeline chart by date]

| Vendor | Contract | Renewal Date | Days Left | Owner | Status |
| ...
```

ใช้ structure เดียวกันแต่ chart เป็น timeline แทน bar

## หลักการที่ encode ในไฟล์นี้

1. **Speed of comprehension > visual polish** — dashboard ไม่ใช่ presentation slide
2. **Consistency across plugins** — ทุก skill ผลิต dashboard ตาม template เดียวกัน
3. **Markdown version mandatory** — terminal user เป็น first-class citizen
4. **Untrusted input = always escape** — ไม่ trust ใด ๆ ที่ไม่ใช่ output ของ skill เอง
5. **Reviewer note บน dashboard ก็ต้องมี** — safety metadata ไม่ใช่ optional

## ไฟล์นี้กับ plugin ไหน

จากการอ่าน CLAUDE.md ของแต่ละ plugin ใน `claude-for-legal` (ดู [03-plugins](../03-plugins/)) — plugin ที่ skill อ้าง dashboard template:

- `commercial-legal` — `tabular-review` (NDA, vendor)
- `corporate-legal` — `renewal-tracker`, M&A diligence summary
- `privacy-legal` — `entity-compliance` dashboard
- `ip-legal` — OSS license dashboard
- `regulatory-legal` — rule diff summary

ทั้งหมดอ้าง `references/dashboard-template.md` ใน guardrail หรือใน SKILL.md เพื่อ enforce structure

## ข้อสังเกตจากการอ่าน template

- **"Reviewer note" ไม่ optional** — บรรทัดที่ 3 ส่วนของโครงสร้าง คือ safety metadata block ที่ทุก output ของ Claude for Legal ต้องมี (ดู `CONTRIBUTING.md` ของ repo) — dashboard ก็ไม่ข้าม
- **"What next?" decision tree = options ไม่ใช่คำแนะนำ** — ตามหลักการ "skill เสนอ option ให้ human เลือก ไม่ตัดสินแทน" (เหมือนกับ External Brain principle ใน CLAUDE.md ของผู้ใช้)
- **"Excel optional, where it fits"** — template ตั้งใจเลือก surface ไม่ทุก dashboard ต้อง Excel — บางตัวเป็น diff รายเดือนที่ Excel ใช้ไม่ได้
- **Color palette ตายตัว** — `Red/orange/yellow/green` สำหรับ severity, `gray` สำหรับ neutral, `blue` สำหรับ status, **"Nothing else"** — ไม่อนุญาตให้แต่ละ plugin คิดสีของตัวเอง

อ่านคู่กับ [company-profile](company-profile.html) — สองไฟล์นี้คือ cross-plugin standard ของ Claude for Legal ที่ทุก plugin ต้องเคารพ
