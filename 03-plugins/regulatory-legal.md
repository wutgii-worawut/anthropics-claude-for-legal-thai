---
title: regulatory-legal (กฎหมายกำกับดูแล)
parent: 03 Plugins
nav_order: 6
---

# regulatory-legal — Plugin งานกำกับดูแล

## บทนำ

`regulatory-legal` คือ plugin สำหรับทีม compliance / regulatory counsel ที่ต้อง "อ่าน Federal Register แทนทีม" — มันเฝ้า feed กฎใหม่ที่ออกจากหน่วยงานกำกับ (regulator), diff กฎที่เปลี่ยนกับห้องสมุดนโยบาย (policy library) ภายในองค์กร, ติดตามกำหนดส่ง comment ของ NPRM (Notice of Proposed Rulemaking — ประกาศร่างกฎเพื่อรับฟังความเห็น), เก็บ tracker ของช่องว่าง (gap) ที่ยังไม่ปิด และเขียน "digest เช้าวันจันทร์" ที่ทีมอ่านจริง — ไม่ใช่ noise ทุกคำพูดของกรรมาธิการ (commissioner)

หัวใจของ plugin นี้คือ "**materiality threshold**" — ความเป็นสาระสำคัญที่เรียนรู้จาก cold-start เพื่อกรองว่าอะไร "ต้องอ่านวันนี้" vs. "FYI" vs. "ทิ้งได้" — ทุกอย่างเป็น "regulatory change" ในเชิงเทคนิค แต่ plugin เรียนรู้ว่าอะไรสำคัญต่อทีมนี้จริง

เหมาะกับ — compliance / regulatory counsel, privacy / product counsel (รับ alert ที่กรองแล้ว), GC (รับ escalation gap สำคัญที่มี deadline)

---

## `.claude-plugin/plugin.json`

```json
{
  "name": "regulatory-legal",
  "version": "1.0.2",
  "description": "Watches regulatory feeds, diffs new rules against your policy library, tracks comment deadlines and open gaps, and writes the digest your team reads Monday morning.",
  "author": {
    "name": "Anthropic"
  }
}
```

| Field | คำอธิบาย |
|---|---|
| `name` | `regulatory-legal` (ใช้ใน `/regulatory-legal:...`) |
| `version` | 1.0.2 |
| `description` | สรุป — เฝ้า feed, diff กับ policy library, track comment deadlines + open gaps, ออก Monday digest |
| `author.name` | Anthropic |

---

## โครงสร้างโฟลเดอร์

```text
regulatory-legal/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json                    # MCP — Slack, Google Drive
├── .gitignore
├── CLAUDE.md                    # template practice profile (38 KB)
├── README.md
├── agents/
│   └── reg-change-monitor.md    # scheduled agent — weekly feed digest
├── hooks/
│   └── hooks.json               # empty
└── skills/                      # 8 skills (ไม่มี data folder)
    ├── cold-start-interview/    # ตั้ง watchlist + index policy + materiality
    ├── customize/
    ├── comments/                # NPRM comment tracker
    ├── gap-surfacer/            # reference skill (framework)
    ├── gaps/                    # open gaps tracker
    ├── matter-workspace/
    ├── policy-diff/             # diff reg ใหม่กับ policy
    ├── policy-redraft/          # ร่าง markup ปิด gap
    └── reg-feed-watcher/        # check feed ทันที
```

---

## Skills ทั้งหมด

| Skill | Path | คำสั่ง | สำหรับ |
|---|---|---|---|
| cold-start-interview | `skills/cold-start-interview/` | `/regulatory-legal:cold-start-interview` | สร้าง watchlist, index policy library, เรียน materiality threshold |
| customize | `skills/customize/` | `/regulatory-legal:customize` | ปรับ profile (watchlist, threshold, อื่น ๆ) ทีละจุด |
| reg-feed-watcher | `skills/reg-feed-watcher/` | `/regulatory-legal:reg-feed-watcher` | เช็ค feed เดี๋ยวนี้ รายงานสิ่งใหม่ตั้งแต่ check ล่าสุด (กรองด้วย threshold) |
| policy-diff | `skills/policy-diff/` | `/regulatory-legal:policy-diff [reg]` | diff กฎที่เปลี่ยนแปลงเทียบกับ policy library — ระบุนโยบายที่กระทบและขนาดของช่องว่าง |
| gaps | `skills/gaps/` | `/regulatory-legal:gaps` | ดู open gap — flag, status remediation; รองรับ `--close GAP-ID` / `--accept GAP-ID` |
| comments | `skills/comments/` | `/regulatory-legal:comments` | ดู NPRM ที่ comment period เปิดอยู่ — log การตัดสิน file/not file/waived; รองรับ `--decide CMT-ID` |
| policy-redraft | `skills/policy-redraft/` | `/regulatory-legal:policy-redraft [GAP-ID]` | ร่าง markup ของนโยบายเพื่อปิด gap — draft สำหรับ internal review ไม่ใช่แก้ไฟล์นโยบายโดยตรง |
| gap-surfacer | `skills/gap-surfacer/` | (reference) | กรอบงานติดตาม gap/comment ที่ skill `gaps` และ `comments` ใช้ร่วมกัน |
| matter-workspace | `skills/matter-workspace/` | `/regulatory-legal:matter-workspace` | จัดการ workspace สำหรับสำนักงานหลายลูกค้า |

### Logical flow

```
reg-feed-watcher    →  คัดกรอง material/review-worthy/FYI
       ↓ (เจอ material)
policy-diff         →  diff vs. policy → gap?
       ↓ (เจอ gap)
gaps (tracker)      →  เปิด GAP-ID
       ↓
policy-redraft      →  ร่าง markup สำหรับ human review
       ↓
gaps --close GAP-ID →  ปิดเมื่อ policy ถูกอนุมัติแก้
```

---

## Agent — reg-change-monitor

`agents/reg-change-monitor.md` (1.8 KB) — agent สั้นกระชับ เพราะตรรกะหลักอยู่ใน skill `reg-feed-watcher` + `policy-diff`

**Schedule** — ตาม profile `~/.claude/plugins/config/claude-for-legal/regulatory-legal/CLAUDE.md` → Feed configuration → Check cadence; default รายสัปดาห์ (รายวันถ้า environment กำลัง active)

**Pipeline**:
1. อ่าน profile — watchlist + materiality threshold
2. รัน `reg-feed-watcher` — ดึงทุก feed กรอง
3. สำหรับ item "always material" — รัน `policy-diff` ทันทีเพื่อรวม gap summary
4. โพสต์ digest

**Output format**:

```
📋 Regulatory digest — [date]

🔴 Material (action likely needed)
• [Regulator] — [title] — [one line] — [link]
  → Gap check: [policy X may need update — see diff]

🟡 Review-worthy
• [Regulator] — [title] — [one line] — [link]

📝 FYI — [N] items — [expandable list]

Open gaps: [N] — oldest [days]
```

**ไม่ทำ** — agent ไม่อัปเดต policy เอง (มนุษย์อัปเดตเสมอ), ไม่ตัดสิน edge case (item ที่ borderline เข้า "review-worthy" — รอมนุษย์ตัดสิน)

### Managed Agent version — scheduled

ถ้าต้องการให้รันอัตโนมัติเต็มรูปแบบบน server ของ Anthropic ใช้ template `managed-agent-cookbooks/reg-monitor/` — ใช้ source เดียวกับ `reg-change-monitor.md` ของ Claude Code agent แต่ deploy ผ่าน `POST /v1/agents` (Managed Agent API)

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export GDRIVE_MCP_URL=...
../../scripts/deploy-managed-agent.sh reg-monitor
```

ข้อต่างหลัก —
- Claude Code agent → ตั้ง reminder/cron ภายนอกเอง
- Managed Agent (cookbook) → cron-on-server, Anthropic จัดการ scheduling + retries

---

## Hooks

`hooks/hooks.json` มีเนื้อหา `{ "hooks": {} }` — empty placeholder เช่นเดียวกับ `employment-legal` (สงวนพื้นที่ไว้สำหรับ hook ในอนาคต เช่น redact sensitive policy text ก่อนส่ง MCP)

---

## Connectors (MCP)

จาก `.mcp.json`:

```json
{
  "mcpServers": {
    "Slack": { "type": "http", "url": "https://mcp.slack.com/mcp" },
    "Google Drive": { "type": "http", "url": "https://drivemcp.googleapis.com/mcp/v1" }
  },
  "recommendedCategories": ["legal-research", "regulatory-intelligence", "documents", "chat", "email"]
}
```

- **Slack** — โพสต์ digest, notification เรื่อง gap assignment + comment deadline
- **Google Drive** — เข้าถึง policy library
- **`recommendedCategories`** แนะนำให้เชื่อม Thomson Reuters Regulatory Intelligence หรือ CourtListener เพิ่ม — เพราะ "citation verification" เป็นหัวใจ: citation จาก research tool ที่ verify ได้ tag ด้วยแหล่ง; citation อื่น ๆ tag `[verify]` หรือ `[verify-pinpoint]`

**ข้อสำคัญ** — owner notification (gap assignment, due-date reminder, NPRM alert) ต้องการ Slack MCP — ถ้าไม่มี gap tracker + comment tracker ยังทำงาน เพียงแต่ไม่โพสต์อัตโนมัติ จะ flag ใน status report แทน

---

## Workflow ตัวอย่าง

### Case 1 — สัปดาห์ปกติ — Monday digest

```
[ตี 6 วันจันทร์ — agent รันอัตโนมัติตาม schedule]

Slack #legal-reg-watch:

📋 Regulatory digest — 2026-05-11

🔴 Material (3)
• SEC — Final rule on cybersecurity disclosure timing — เปลี่ยนระยะเวลา
  รายงาน material incident จาก 4 → 2 วันทำการ
  → Gap check: Incident Response Policy v3.2 บัญญัติ 4 วัน → ต้องอัปเดต
  → GAP-2026-018 เปิดแล้ว, owner @priya, due 2026-05-25

• FTC — Negotiated rulemaking on AI training transparency
  → Gap check: ยังไม่กระทบ — practice ไม่ได้ใช้ training data จากลูกค้า

• EU — Digital Omnibus provisional agreement on AI Act timelines
  → Gap check: AI policy ต้องเพิ่ม transparency grace period ที่สั้นลง

🟡 Review-worthy (5) — DOL, CFPB, ...

📝 FYI — 12 items

Open gaps: 4 — oldest 23 days (GAP-2026-014, @raj)
```

### Case 2 — comment period ของ NPRM ใกล้ปิด

```
User: /regulatory-legal:comments

Output:
Open comment periods — 3 active

🔴 CMT-2026-007 — SEC AI in trading rules (NPRM)
   Comment deadline: 2026-05-30 (12 days)
   Decision logged: file (Q&A draft 60% complete)
   Owner: @sarah

🟡 CMT-2026-008 — FTC AI vendor disclosure
   Comment deadline: 2026-06-15 (28 days)
   Decision: not yet logged
   → ใช้ /regulatory-legal:comments --decide CMT-2026-008 เพื่อบันทึก

⚪ CMT-2026-009 — FCC robocall AI
   Decision: waived (out of scope)
```

---

## ข้อควรระวัง

- **Materiality threshold คือคุณค่าหลัก** — ทุกอย่างเป็น "regulatory change" ในเชิงเทคนิค plugin เรียนรู้ว่า "อะไรสำคัญต่อทีมนี้จริง" ตอน cold-start — ถ้า threshold ตั้งหลวมเกินไป → digest จะเป็น noise; แน่นเกินไป → พลาดเรื่องสำคัญ ทบทวนเป็นระยะ
- **Digest items = "screened leads, not legal conclusions"** — materiality filter เป็น threshold ที่ตั้ง ไม่ใช่ legal judgment — ทนายเป็นผู้ตัดสินใจสุดท้าย
- **Gap check เป็น first pass ไม่ใช่ legal assessment** — heuristics เทียบ text ของกฎใหม่กับ policy text "Gap" = lead สำหรับทนาย; "aligned" ไม่ใช่ certificate ว่า compliant
- **Watchlist = coverage assertion** — regulator ที่ไม่อยู่ใน watchlist อาจออกกฎที่เกี่ยวข้องได้ การพลาด regulator = configuration bug ไม่ใช่ feed bug — review watchlist ทุก 3-6 เดือน
- **policy-redraft ไม่แก้ source document โดยตรง** — output คือ "marked-up draft" ที่ทีมเอาไปทำ manual edit ใน CMS ของ policy
- **คู่กับ privacy-legal** — `regulatory-legal:reg-change-monitor` เป็น "watcher" รายสัปดาห์; `privacy-legal:reg-gap-analysis` เป็น "deep dive" ต่อ regulation รายตัว — ใช้คู่กัน
- **ถ้าไม่มี research tool connector** — citation จะ tag `[verify]` ทั้งหมด และ reviewer note หน้าเอกสารจะระบุว่าไม่ได้ verify source — plugin ยังทำงานได้ เพียงแต่ทนายต้อง verify เพิ่มเอง
