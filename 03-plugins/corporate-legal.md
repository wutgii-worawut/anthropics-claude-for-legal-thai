---
title: corporate-legal
parent: 03 Plugins
nav_order: 4
---

# corporate-legal — Corporate Counsel + M&A

> "Runs M&A diligence at scale with cited tabular review, builds disclosure schedules and closing checklists, drafts board consents and minutes in house format, and tracks entity compliance deadlines across jurisdictions."
> — `corporate-legal/.claude-plugin/plugin.json`

## 1. บทนำ

`corporate-legal` เป็น plugin ที่ **ใหญ่ที่สุด** ใน Group 1 — 13 skills — เพราะครอบคลุมงาน corporate counsel ที่หลายระดับ:

- **M&A diligence** — review document ใน VDR เป็น 100–1000+ ฉบับ; tabular review (1 row = 1 doc, 1 column = 1 data point, ทุก cell มี citation)
- **Disclosure schedule** — build material contracts schedule จาก diligence finding ตาม Material Contract definition ของ purchase agreement
- **Closing checklist** — track ว่ายังเหลืออะไรก่อน close — critical path, days to close
- **Board & secretary** — draft board minutes, written consent (UWC), committee minutes ใน house format
- **Entity compliance** — annual report, good standing, filing deadline ข้าม jurisdiction
- **Post-close integration** — phased workplan, consent tracking, contract assignment ตอน integration

plugin นี้ออกแบบเป็น **modular** — cold-start interview จะถามว่า active module ไหน (M&A / Board & Secretary / Public Company / Entity Management) แล้วเขียนเฉพาะส่วนที่เกี่ยวข้องลง `CLAUDE.md`

**เหมาะกับ**: corporate counsel, M&A counsel, corporate secretary, deputy GC for corporate, entity management team, integration counsel

## 2. `.claude-plugin/plugin.json`

```json
{
  "name": "corporate-legal",
  "version": "1.0.2",
  "description": "Runs M&A diligence at scale with cited tabular review, builds disclosure schedules and closing checklists, drafts board consents and minutes in house format, and tracks entity compliance deadlines across jurisdictions.",
  "author": {
    "name": "Anthropic"
  }
}
```

| ฟิลด์ | ค่า | ความหมาย |
|------|----|---------|
| `name` | `corporate-legal` | namespace (เช่น `/corporate-legal:tabular-review`) |
| `version` | `1.0.2` | semver |
| `description` | (ยาว) | คำอธิบายแสดงใน marketplace |
| `author.name` | `Anthropic` | upstream maintainer |

## 3. โครงสร้างโฟลเดอร์

```text
corporate-legal/
├── .claude-plugin/
│   └── plugin.json
├── .gitignore
├── .mcp.json                    # 7 servers (Slack, Drive, Box, iManage, TopCounsel, Definely, Solve Intelligence)
├── CLAUDE.md                    # practice profile template (38.2 KB)
├── README.md                    # (5.6 KB)
├── agents/
│   └── dataroom-watcher.md      # monitor VDR + post closing checklist status
├── hooks/
│   └── hooks.json               # {} placeholder
├── references/                  # ว่าง (ไม่มี currency-watch แบบ privacy/product)
└── skills/                      # 13 skills (มากที่สุดใน Group 1)
    ├── ai-tool-handoff/
    ├── board-minutes/
    ├── closing-checklist/
    ├── cold-start-interview/
    ├── customize/
    ├── deal-team-summary/
    ├── diligence-issue-extraction/
    ├── entity-compliance/
    ├── integration-management/
    ├── material-contract-schedule/
    ├── matter-workspace/
    ├── tabular-review/
    └── written-consent/
```

## 4. Skills ทั้งหมด

13 skill — ทุกตัวเรียกได้โดยตรง (ไม่มี reference)

| Skill | Path | คำสั่ง | สำหรับ |
|-------|------|--------|---------|
| `tabular-review` | `skills/tabular-review/` | `/corporate-legal:tabular-review` | 1 row = 1 doc, 1 column = 1 data point, ทุก cell **cited to source** — สำหรับ M&A batch review หรือ field extraction |
| `diligence-issue-extraction` | `skills/diligence-issue-extraction/` | `/corporate-legal:diligence-issue-extraction` | อ่าน VDR แล้วดึง issue ตาม house category + materiality threshold → memo |
| `material-contract-schedule` | `skills/material-contract-schedule/` | `/corporate-legal:material-contract-schedule` | build disclosure schedule จาก finding โดย apply Material Contract definition ของ PA |
| `closing-checklist` | `skills/closing-checklist/` | `/corporate-legal:closing-checklist` | track สิ่งที่ blocking close — self-updating ingest จาก diligence + schedule |
| `deal-team-summary` | `skills/deal-team-summary/` | `/corporate-legal:deal-team-summary` | aggregate diligence — exec summary (leadership) vs working summary (team) |
| `integration-management` | `skills/integration-management/` | `/corporate-legal:integration-management` | post-close — phased workplan, consent tracking, contract assignment at scale, weekly status |
| `board-minutes` | `skills/board-minutes/` | `/corporate-legal:board-minutes` | draft board/committee minutes ใน house format — auto-detect meeting จาก calendar |
| `written-consent` | `skills/written-consent/` | `/corporate-legal:written-consent` | draft UWC (Unanimous Written Consent) ใน house format — precedent search, conflict flag, signatory tracking |
| `entity-compliance` | `skills/entity-compliance/` | `/corporate-legal:entity-compliance` | entity tracker — annual report, good standing, deadline 30/60/90 วัน ข้าม jurisdiction |
| `ai-tool-handoff` | `skills/ai-tool-handoff/` | `/corporate-legal:ai-tool-handoff` | detect ว่า Luminance/Kira/Reveal กำลังใช้อยู่ → handoff bulk-review ไปให้ตัวนั้น + QA output ตาม trust level |
| `cold-start-interview` | `skills/cold-start-interview/` | `/corporate-legal:cold-start-interview` | สัมภาษณ์ครั้งแรก — modular: M&A / Board / Public / Entities ตามที่ลูกค้ามี |
| `customize` | `skills/customize/` | `/corporate-legal:customize` | ปรับ profile — module, materiality threshold, schedule format, consent precedent |
| `matter-workspace` | `skills/matter-workspace/` | `/corporate-legal:matter-workspace` | จัดการ matter (deal/client level) |

**ลำดับ skill ตาม M&A lifecycle**:
1. **Diligence**: `tabular-review` → `diligence-issue-extraction` → `ai-tool-handoff` (ถ้ามี Luminance)
2. **Drafting schedules**: `material-contract-schedule`
3. **Pre-close**: `closing-checklist` + `deal-team-summary` (cadence briefing)
4. **Sign / close**: `written-consent` + `board-minutes`
5. **Post-close**: `integration-management`
6. **Always-on**: `entity-compliance`

## 5. Agents

1 agent

| Agent | Description | Trigger |
|-------|-------------|---------|
| `dataroom-watcher` | monitor VDR (Box/Intralinks/Datasite) — flag new upload ที่ตรง high-priority category + post closing checklist status | scheduled, หรือสั่ง "what's new in the data room", "VDR updates" |

Tools ใน frontmatter: `mcp__box__*`, `mcp__intralinks__*`, `mcp__datasite__*`, `mcp__*__slack_send_message` — agent เชื่อม VDR หลายระบบ (deal บางอันใช้ Box deal บางอันใช้ Intralinks)

## 6. Hooks

```text
corporate-legal/hooks/hooks.json
```

เนื้อหา: `{ "hooks": {} }` — placeholder เปล่า

## 7. References

**ไม่มี** `references/` folder ใน corporate-legal — ตรงข้ามกับ privacy-legal และ product-legal ที่มี `currency-watch.md`

เหตุผลที่น่าจะเป็นไปได้: งาน corporate (Material Contract definition, board governance, entity management) **ไม่ค่อย shift** เร็วเท่ากฎ privacy/product/consumer protection — ส่วนใหญ่ pin กับ Delaware General Corporation Law และ corporate code ของแต่ละ state ที่ค่อนข้าง stable

## 8. Connectors (MCP)

จาก `.mcp.json` — **7 servers** (เท่ากับ commercial-legal):

| MCP server | URL | สำหรับ |
|-----------|-----|--------|
| Slack | `mcp.slack.com/mcp` | discussion search |
| Google Drive | `drivemcp.googleapis.com/mcp/v1` | document fetch |
| Box | `mcp.box.com/mcp` | data room + document management |
| iManage | `cloudimanage.com/mcp/work` | governed document management |
| TopCounsel | `api.techgc.co/api/mcp/topcounsel` | outside counsel recommendation จาก L Suite |
| Definely | `mcp.uk.definely.com/api/proxy/core-mcp` | contract structure — definition, cross-reference, structural diff |
| Solve Intelligence | `api.solveintelligence.com/mcp/` | patent workflow — patent + non-patent literature, SEP, prior art (สำหรับ IP-heavy diligence) |

`recommendedCategories`: `legal-document-management`, `virtual-data-room`, `contract-review`, `board-governance`, `documents`, `chat`, `email`

**สังเกต**: Box อยู่ที่นี่ (ไม่ใช่ commercial) — เพราะ Box ใช้เป็น VDR หลักของ M&A; Solve Intelligence ก็มาเฉพาะที่นี่ — สำหรับ deal ที่มี patent diligence

## 9. Workflow ตัวอย่าง

### Case 1 — M&A diligence ของ target ที่มี 800 contract

1. Buy-side counsel เปิด Box VDR แล้วเรียก: `/corporate-legal:tabular-review "review these 800 target contracts for change-of-control, assignment, MAC clauses"`
2. skill ถามจะใช้ AI tool external หรือไม่ — ถ้ามี Luminance/Kira: `/corporate-legal:ai-tool-handoff "Luminance"` → bulk extraction ไปฝั่ง Luminance, Claude QA output
3. ผลออกมาเป็น grid (1 row = 1 contract): change-of-control? assignment consent needed? MAC clause? — **ทุก cell มี citation กลับไปยัง source PDF**
4. `/corporate-legal:diligence-issue-extraction` → memo ตาม category (change-of-control issue, assignment issue, MAC issue)
5. `/corporate-legal:material-contract-schedule` → apply PA's Material Contract definition (e.g., "any contract > $5M annual value or any contract that cannot be terminated within 90 days") → schedule 3.X
6. `/corporate-legal:closing-checklist` ingest finding → update outstanding items + critical path
7. `/corporate-legal:deal-team-summary` ทุกวันศุกร์ → exec summary ส่งให้ leadership

### Case 2 — Board meeting + written consent

1. Calendar event "Board Meeting — May 20" detected by `board-minutes` skill (`/corporate-legal:board-minutes` poll)
2. ทนายขอ agenda + pre-read → skill draft minutes ใน house format + reserved placeholder ที่ต้อง fill หลัง meeting
3. หลังประชุม fill in resolution + vote count → finalize
4. มี action ระหว่างประชุม (เช่น approve equity grant) → ใช้ `/corporate-legal:written-consent "approve equity grant to new VP Engineering"` → draft UWC + precedent search จาก consent repository + director conflict flag + state law notice requirement

### Case 3 — Entity compliance sweep (quarterly)

1. ทนาย run: `/corporate-legal:entity-compliance --report --days 90`
2. skill อ่าน `compliance-tracker.yaml` (built from entity table) → list ทุก filing ที่ due ใน 90 วัน ข้าม Delaware, California, New York, Texas, ฯลฯ
3. ออกเป็น table — entity name, jurisdiction, filing type, deadline, status, owner
4. export CSV ส่งให้ ops: `/corporate-legal:entity-compliance --export --format csv`

### Case 4 — Post-close integration

1. หลัง close M&A: `/corporate-legal:integration-management --init`
2. skill ingest deal artifact (purchase agreement + deal summary + closing checklist) → build phased workplan (Day 1, Week 1, Month 1, ฯลฯ)
3. `/corporate-legal:integration-management --contracts` → list contract ที่ต้อง assignment consent → track status
4. Weekly: `/corporate-legal:integration-management --report` → status report ส่งให้ deal team

## 10. ข้อควรระวัง

- **Tabular review ที่ cite ทุก cell ไม่ใช่ "accuracy guarantee"** — citation บอกว่ามาจาก source ไหน แต่ทนายต้องเปิดดู source verify ว่า cell ถูกตามที่ skill อ้าง — โดยเฉพาะใน M&A ที่ misread หนึ่ง clause = liability หลักพันล้าน
- **Material Contract definition แต่ละ deal ไม่เหมือนกัน** — skill apply definition ของ PA ฉบับนั้น แต่ถ้า PA มีหลาย definition (เช่น "Material Contract" vs "Specified Contract") ต้อง confirm ว่า skill เลือกถูก
- **AI tool handoff ต้อง QA output ของ external tool** — skill บอกว่า trust level (เช่น "spot check 10%") แต่ trust level ต้อง config ใน `CLAUDE.md` ตามจริง ไม่ใช้ default มืดๆ
- **Board minutes ที่ draft อัตโนมัติ** — ต้อง verify ว่า resolution wording + vote count + dissent ถูกต้องตาม minutes book — ผิดพลาดเรื่องนี้กระทบ corporate governance + liability ของ director
- **Written consent กับ major one-off action** — skill มี built-in scope warning แต่ทนายต้องตัดสินว่า action นั้นเหมาะกับ UWC จริงหรือไม่ (vs in-person meeting) — ขึ้นกับ bylaws + state law
- **Entity compliance deadline calculation** — skill คำนวณ filing deadline ตาม jurisdiction default แต่ entity แต่ละตัวอาจมี special case (anniversary date, biennial filing, foreign qualification) — ต้อง verify
- **Solve Intelligence MCP** สำหรับ patent diligence — output เป็น prior art + claim analysis ต้องผ่าน patent counsel ก่อนใช้
- **ทุก output ต้องผ่าน attorney review** — ตามข้อ disclaimer ใน `README.md` ของ source

---

**Source**: [`anthropics/claude-for-legal/corporate-legal/`](https://github.com/anthropics/claude-for-legal/tree/main/corporate-legal)
**Version**: 1.0.2
**Last reviewed**: 2026-05-13
