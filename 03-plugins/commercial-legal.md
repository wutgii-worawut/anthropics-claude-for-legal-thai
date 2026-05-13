---
title: commercial-legal
parent: 03 Plugins
nav_order: 1
---

# commercial-legal — สัญญาเชิงพาณิชย์

> "Reviews vendor agreements, NDAs, and SaaS subscriptions against your sales-side or purchasing-side playbook, tracks renewals and cancel-by deadlines before they're missed, routes escalations to the right approver, and translates reviews into summaries business stakeholders will actually read."
> — `commercial-legal/.claude-plugin/plugin.json`

## 1. บทนำ

`commercial-legal` คือ plugin สำหรับ **ทนายในบริษัทที่ดูสัญญาเชิงพาณิชย์** — งานที่เกิดทุกวัน เช่น ตรวจ NDA จาก vendor, review SaaS subscription ของ marketing tool, ตรวจ MSA ก่อนลงนาม, ติดตาม cancel-by deadline ของสัญญาที่กำลังจะ auto-renew และส่งสรุปให้ business stakeholder อ่านในสองนาที

plugin นี้รองรับทั้ง **sales-side** (ขายของให้ลูกค้า) และ **purchasing-side** (จัดซื้อจาก vendor) — ใน cold-start interview จะถามว่าทีมทำงานฝั่งไหน หรือทั้งสองฝั่ง แล้วจะเลือก playbook ที่เหมาะสมไปปรับใน `CLAUDE.md` ของผู้ใช้

**เหมาะกับ**: in-house counsel, BD/sales legal, procurement legal, contracts manager — โดยเฉพาะทีมที่ต้องการให้ sales/procurement triage NDA เองได้ก่อนส่งให้ legal

## 2. `.claude-plugin/plugin.json`

```json
{
  "name": "commercial-legal",
  "version": "1.0.2",
  "description": "Reviews vendor agreements, NDAs, and SaaS subscriptions against your sales-side or purchasing-side playbook, tracks renewals and cancel-by deadlines before they're missed, routes escalations to the right approver, and translates reviews into summaries business stakeholders will actually read.",
  "author": {
    "name": "Anthropic"
  }
}
```

**อธิบายแต่ละฟิลด์**:

| ฟิลด์ | ค่า | ความหมาย |
|------|----|---------|
| `name` | `commercial-legal` | ชื่อ plugin — ใช้เป็น namespace ของ command (เช่น `/commercial-legal:review`) |
| `version` | `1.0.2` | semver — ทุก plugin ใน claude-for-legal อยู่ที่ 1.0.2 ณ commit นี้ |
| `description` | (ยาว) | คำอธิบายที่จะปรากฏใน marketplace listing — เขียนเป็นกริยา ("Reviews...", "tracks...") |
| `author.name` | `Anthropic` | ผู้ดูแล plugin ทาง upstream |

**สังเกต**: ไม่มีฟิลด์ `marketplace`, `category`, `tags`, `license` ในระดับ plugin manifest — ฟิลด์เหล่านั้นไปอยู่ที่ `.mcp.json` (`recommendedCategories`) และ root `.claude-plugin/marketplace.json` ของ repo

## 3. โครงสร้างโฟลเดอร์

```text
commercial-legal/
├── .claude-plugin/
│   └── plugin.json              # manifest
├── .gitignore
├── .mcp.json                    # MCP server config (Ironclad, DocuSign, iManage, ...)
├── CLAUDE.md                    # practice profile template (42 KB) — ใช้ใน cold-start
├── README.md                    # แนะนำ plugin (8.4 KB)
├── agents/                      # scheduled / data-triggered agents
│   ├── deal-debrief.md
│   ├── playbook-monitor.md
│   └── renewal-watcher.md
├── hooks/
│   └── hooks.json               # {} (placeholder)
├── logs/                        # runtime log (gitignored)
└── skills/                      # 12 skills (1 โฟลเดอร์ = 1 skill)
    ├── amendment-history/
    ├── cold-start-interview/
    ├── customize/
    ├── escalation-flagger/
    ├── matter-workspace/
    ├── nda-review/              # reference (loaded by review)
    ├── renewal-tracker/
    ├── review/                  # ตัวรวบ — routes ไปยัง vendor/nda/saas review
    ├── review-proposals/
    ├── saas-msa-review/         # reference
    ├── stakeholder-summary/
    └── vendor-agreement-review/ # reference
```

## 4. Skills ทั้งหมด

12 skill — 3 ตัวเป็น **reference** (`user-invocable: false` โหลดจาก `review`) ที่เหลือเรียกได้โดยตรง

| Skill | Path | คำสั่ง | สำหรับ |
|-------|------|--------|---------|
| `review` | `skills/review/` | `/commercial-legal:review` | router หลัก — จำแนกชนิดสัญญาแล้วเรียก nda/vendor/saas review พร้อมรวม memo ออกมา |
| `nda-review` | `skills/nda-review/` | (reference) | triage NDA เป็น GREEN/YELLOW/RED — ออกแบบให้ sales/BD self-serve ก่อนยิงเข้า legal |
| `vendor-agreement-review` | `skills/vendor-agreement-review/` | (reference) | review MSA / services agreement / inbound vendor — flag deviation, redline, route approver |
| `saas-msa-review` | `skills/saas-msa-review/` | (reference) | review SaaS subscription — auto-renewal, price escalation, data portability, SLA, subprocessor |
| `amendment-history` | `skills/amendment-history/` | `/commercial-legal:amendment-history` | trace ว่าสัญญาเปลี่ยนอะไรไปบ้างจาก base agreement → amendments หลายฉบับ |
| `renewal-tracker` | `skills/renewal-tracker/` | `/commercial-legal:renewal-tracker` | แสดง cancel-by deadline ที่กำลังมาถึง — ทำงานจาก renewal register ที่ได้รับ handoff จาก saas-msa-review |
| `escalation-flagger` | `skills/escalation-flagger/` | `/commercial-legal:escalation-flagger` | route issue ไปหาผู้อนุมัติที่ถูกต้องตาม escalation matrix |
| `stakeholder-summary` | `skills/stakeholder-summary/` | `/commercial-legal:stakeholder-summary` | แปลง review เป็น two-minute brief สำหรับ business stakeholder |
| `review-proposals` | `skills/review-proposals/` | `/commercial-legal:review-proposals` | review playbook update proposals ที่ playbook-monitor agent สรุปมา |
| `cold-start-interview` | `skills/cold-start-interview/` | `/commercial-legal:cold-start-interview` | สัมภาษณ์ครั้งแรกตอนติดตั้ง — สร้าง practice profile |
| `customize` | `skills/customize/` | `/commercial-legal:customize` | ปรับ profile โดยไม่ต้องเริ่ม cold-start ใหม่ทั้งกระบวน |
| `matter-workspace` | `skills/matter-workspace/` | `/commercial-legal:matter-workspace` | จัดการ matter (new/list/switch/close/none) สำหรับ multi-client practitioner |

**ข้อสังเกตเรื่อง routing**: skill `review` คือ "ปากทาง" — ผู้ใช้แค่ส่งไฟล์ สัญญาประเภทไหน skill `review` จะตรวจจาก title แล้วเรียก reference skill ที่ถูกต้อง

## 5. Agents

3 agent อยู่ใน `agents/` — เป็น `.md` ที่มี YAML frontmatter ระบุ tools และ trigger

| Agent | Description | Trigger |
|-------|-------------|---------|
| `deal-debrief` | weekly agent — surface สัญญาที่ลงนามเมื่อสัปดาห์ที่ผ่านมาและมี deviation จาก playbook แล้วถามทนายให้บันทึก context ตอนที่ยังจำได้ | จันทร์เช้า (default) หรือสั่งเอง: "deal debrief", "log deviations", "debrief last week's deals" |
| `playbook-monitor` | data-triggered — เฝ้า deviation log ถ้า clause หนึ่งถูก deviate มาเกิน threshold (default = 5 ครั้งใน rolling 12 เดือน) จะเสนอ playbook update | "check playbook", "any playbook updates" หรือ auto หลัง `deal-debrief` ทุกครั้ง |
| `renewal-watcher` | scheduled — เช็ค renewal register แล้ว post ลงช่อง Slack ที่กำหนดใน `CLAUDE.md` | weekly, หรือ "what's renewing", "check renewals", "renewal report" |

Agent ใช้ `model: sonnet` และระบุ tools เป็นรายการชัดเจน เช่น `mcp__ironclad__*`, `mcp__*__slack_send_message` — เป็นการ scope tool ให้ agent เข้าถึงเฉพาะที่จำเป็น (principle of least privilege)

## 6. Hooks

```text
commercial-legal/hooks/hooks.json
```

เนื้อหา: `{ "hooks": {} }` — placeholder สำหรับ pre/post tool hook แต่ในเวอร์ชัน 1.0.2 ยัง **ไม่มี hook ที่ activate** ทำให้ plugin นี้พึ่งพา validator scripts ในระดับ repo (`scripts/lint-tool-scope.py`) มากกว่าการใส่ hook ใน plugin ตัวเอง

## 7. References

`commercial-legal` ไม่มีโฟลเดอร์ `references/` เป็นของตัวเอง — playbook ถูกบรรจุไว้ใน `CLAUDE.md` ของ user (เกิดจาก cold-start interview) ไม่ใช่ในไฟล์ static ของ plugin

## 8. Connectors (MCP)

จาก `.mcp.json` — **7 servers**:

| MCP server | URL | สำหรับ |
|-----------|-----|--------|
| Ironclad | `mcp.na1.ironcladapp.com/mcp` | ค้น contract repository — expiring MSA, termination clause, vendor agreement |
| DocuSign | `mcp.docusign.com/mcp` | agreement search, status tracking, signature workflow |
| iManage | `cloudimanage.com/mcp/work` | governed document management — เอกสารอยู่ใน iManage, access permission-bound |
| TopCounsel | `api.techgc.co/api/mcp/topcounsel` | outside counsel recommendation จาก L Suite (5,000+ in-house counsel community) |
| Definely | `mcp.uk.definely.com/api/proxy/core-mcp` | deterministic access to contract structure — resolve definition, cross-reference, structural diff |
| Slack | `mcp.slack.com/mcp` | search message, read channel, find discussion |
| Google Drive | `drivemcp.googleapis.com/mcp/v1` | search/read/fetch จาก Drive |

`recommendedCategories`: `contract-management`, `e-signature`, `legal-document-management`, `email`, `documents`, `chat`, `contract-review`, `outside-counsel-network`

**สังเกต**: plugin นี้คือ "ตัวต่อ MCP เยอะที่สุด" ใน Group 1 — เพราะ commercial work อยู่ใน contract lifecycle management (CLM) ระบบจริง

## 9. Workflow ตัวอย่าง

### Case 1 — Inbound NDA จาก vendor (sales-side scenario)

1. Sales rep ได้รับ NDA จาก lead → drag-and-drop ไฟล์เข้า Claude
2. พิมพ์: `/commercial-legal:review path/to/nda.pdf`
3. `review` skill ตรวจจาก title ว่าเป็น NDA → load `nda-review` (reference)
4. `nda-review` เทียบกับ playbook ใน `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`
5. ออกผลเป็น tag — **GREEN** (lawyer ไม่ต้องดู), **YELLOW** (lawyer skim ได้), **RED** (ต้อง escalate)
6. ถ้า RED → ใช้ `/commercial-legal:escalation-flagger` ส่งหาผู้อนุมัติตาม matrix
7. ถ้า GREEN → ใช้ `/commercial-legal:stakeholder-summary` แปลงเป็น brief สำหรับ sales rep

### Case 2 — Renewal radar (proactive)

1. SaaS marketing tool ราคา $50k/ปี กำลังจะ auto-renew ในอีก 35 วัน
2. `renewal-watcher` agent (weekly) เช็ค renewal register → เห็นว่า cancel-by อยู่ใน 30 วัน → post ลง Slack #legal-renewals
3. ทนาย click link → เปิด `/commercial-legal:renewal-tracker --days 30` ดูทั้งหมดที่ใกล้ deadline
4. ตัดสินใจว่าจะ renew, renegotiate หรือ cancel — ถ้า renegotiate → ส่งกลับเข้า `/commercial-legal:review` รอบใหม่

### Case 3 — Playbook drift loop (long-term)

1. หกเดือนผ่านไป ทนายที่ใช้ plugin ได้ override "payment terms = Net 30" ไป Net 45 จำนวน 6 ครั้ง
2. `playbook-monitor` agent ตรวจ deviation log → threshold (5 ครั้ง) ครบ → เสนอ playbook update
3. ทนายเปิด `/commercial-legal:review-proposals` → อ่าน proposal → approve → playbook อัปเดต
4. รอบถัดไป `review` จะใช้ Net 45 เป็น default — feedback loop ปิด

## 10. ข้อควรระวัง

- **GREEN/YELLOW/RED ไม่ใช่คำตัดสินทางกฎหมาย** — เป็น triage signal เพื่อช่วยจัดลำดับเวลาของทนาย ไม่ใช่ "อนุมัติ" หรือ "ปฏิเสธ" ออกจาก legal
- **NDA review โดย sales/BD** ต้องตั้ง threshold ให้ชัดใน `CLAUDE.md` — ถ้า counterparty เป็น large enterprise หรือ มูลค่าดีล > $X ต้องเข้าทนายเสมอ ไม่ว่า tag จะเป็นอะไร
- **renewal-tracker ทำงานจาก register ที่ user ดูแล** — ถ้าไม่มีการ enroll สัญญาเข้า register ตั้งแต่ตอน review ก็จะไม่เตือน (ไม่ใช่ระบบ scan อัตโนมัติทุก contract ใน CLM)
- **playbook auto-update ผ่าน playbook-monitor** — proposal ต้องผ่าน `review-proposals` แล้วทนาย approve เอง ไม่มีการ auto-merge
- **ทุก output ต้องผ่าน attorney review ก่อนใช้กับ counterparty จริง** — output ของ plugin ไม่ใช่ legal advice ต่อ counterparty ของลูกค้า

ตามคำเตือนใน `README.md` ของ source:

> "These plugins do not represent Anthropic's legal positions. ... The attorney using the plugin — not the plugin, and not Anthropic — is responsible for the legal positions taken in their work product."

---

**Source**: [`anthropics/claude-for-legal/commercial-legal/`](https://github.com/anthropics/claude-for-legal/tree/main/commercial-legal)
**Version**: 1.0.2
**Last reviewed**: 2026-05-13
