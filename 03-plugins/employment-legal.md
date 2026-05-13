---
title: employment-legal (กฎหมายแรงงาน)
parent: 03 Plugins
nav_order: 5
---

# employment-legal — Plugin กฎหมายการจ้างงาน

## บทนำ

`employment-legal` คือ plugin สำหรับงานที่ปรึกษากฎหมายแรงงาน (employment counsel) แบบ in-house — ครอบคลุมการตรวจสอบจ้างงาน (hiring review), การเลิกจ้าง (termination review), การร่างนโยบาย (policy drafting), การจำแนกสถานะแรงงาน (worker classification) และการติดตามวันลาที่มีกำหนดทางกฎหมาย (statutory leave) plugin นี้สร้างขึ้นรอบ "footprint เชิงเขตอำนาจ (jurisdictional footprint)" ที่เรียนรู้จากขั้น cold-start ทำให้ระบบรู้ว่าทีมมีพนักงานในมลรัฐใดของสหรัฐ ฯ และประเทศใด พร้อมรู้ว่ากฎแต่ละที่ต่างกันอย่างไร — เป็น plugin ที่ skill เยอะที่สุดในชุด (20 skills) เพราะแต่ละขั้นตอนของวงจรการจ้างงานต้องการ workflow แยกเฉพาะ

เหมาะกับ — ทนายความด้านแรงงาน (employment counsel), HR business partner, GC ที่รับเรื่อง escalation จากการเลิกจ้างความเสี่ยงสูงและ RIF (reduction in force)

> **หลักสำคัญ** — ทุก output คือ "ฉบับร่างสำหรับทนายตรวจสอบ (draft for attorney review)" ไม่ใช่ข้อสรุปทางกฎหมาย ทนายเป็นผู้ verify, decide เสมอ และการกระทำที่ irreversible (filing, sending, executing) ถูก gate ไว้รอ explicit confirmation

---

## `.claude-plugin/plugin.json`

```json
{
  "name": "employment-legal",
  "version": "1.0.2",
  "description": "Reviews hires and terminations for jurisdiction-specific risk flags, classifies workers against the controlling state test, tracks leave deadlines before they're missed, runs internal investigations, and drafts policies with state supplements where the law differs.",
  "author": {
    "name": "Anthropic"
  }
}
```

| Field | คำอธิบาย |
|---|---|
| `name` | ชื่อ plugin ที่ใช้ใน slash command (`/employment-legal:...`) |
| `version` | เวอร์ชันปัจจุบัน (semver) — 1.0.2 |
| `description` | สรุปขอบเขต: hire/termination review, worker classification, leave tracking, investigations, policy drafting |
| `author.name` | Anthropic |

---

## โครงสร้างโฟลเดอร์

```text
employment-legal/
├── .claude-plugin/
│   └── plugin.json              # metadata
├── .mcp.json                    # MCP — Slack, Google Drive
├── .gitignore
├── CLAUDE.md                    # template practice profile (39 KB)
├── README.md
├── agents/
│   └── leave-tracker.md         # scheduled agent — weekly leave deadline check
├── data/
│   └── .gitkeep                 # placeholder; ผู้ใช้วาง register/data ที่นี่
├── hooks/
│   └── hooks.json               # empty — { "hooks": {} }
└── skills/                      # 20 skills (มากสุดในชุด)
    ├── cold-start-interview/
    ├── customize/
    ├── expansion-kickoff/
    ├── expansion-update/
    ├── handbook-updates/
    ├── hiring-review/
    ├── internal-investigation/  # reference skill (framework)
    ├── international-expansion/ # reference skill (framework)
    ├── investigation-add/
    ├── investigation-memo/
    ├── investigation-open/
    ├── investigation-query/
    ├── investigation-summary/
    ├── leave-tracker/
    ├── log-leave/
    ├── matter-workspace/
    ├── policy-drafting/
    ├── termination-review/
    ├── wage-hour-qa/
    └── worker-classification/
```

หมายเหตุ — โฟลเดอร์ `data/` ว่างเปล่าตอนติดตั้ง ใช้สำหรับวางไฟล์ที่ผู้ใช้สร้าง เช่น `leave-register.yaml`, investigation logs, expansion trackers ที่ persist อยู่ใน path `~/.claude/plugins/config/claude-for-legal/employment-legal/`

---

## Skills ทั้งหมด

| Skill | Path | คำสั่ง | สำหรับ |
|---|---|---|---|
| cold-start-interview | `skills/cold-start-interview/SKILL.md` | `/employment-legal:cold-start-interview` | สัมภาษณ์ครั้งแรก — เรียน footprint รัฐ/ประเทศ + escalation table จาก handbook + termination memo |
| customize | `skills/customize/` | `/employment-legal:customize` | ปรับ practice profile ทีละจุด ไม่ต้องรัน cold-start ใหม่ |
| hiring-review | `skills/hiring-review/` | `/employment-legal:hiring-review` | ตรวจ offer letter + restrictive covenant (ข้อสัญญาห้ามแข่ง/ห้ามชักชวน) ตามเขตอำนาจ |
| termination-review | `skills/termination-review/` | `/employment-legal:termination-review` | ตรวจการเลิกจ้าง — high-risk flag, severance + release, final pay timing แยกตามรัฐ |
| worker-classification | `skills/worker-classification/` | `/employment-legal:worker-classification` | จำแนกแรงงาน — employee/IC/temp/vendor ผ่าน test ของแต่ละรัฐ (FLSA test, ABC test ของ CA ฯลฯ) |
| wage-hour-qa | `skills/wage-hour-qa/` | `/employment-legal:wage-hour-qa` | ตอบคำถามเรื่อง classification, overtime, meal/rest break, final pay แยกตามรัฐ/ประเทศ |
| policy-drafting | `skills/policy-drafting/` | `/employment-legal:policy-drafting [topic]` | ร่างนโยบายพร้อม "state supplement" ที่กฎต่างกัน |
| handbook-updates | `skills/handbook-updates/` | `/employment-legal:handbook-updates` | diff handbook เก่า/ใหม่ flag ripple effect + state supplement impact |
| expansion-kickoff | `skills/expansion-kickoff/` | `/employment-legal:expansion-kickoff [country]` | เริ่ม project ขยายธุรกิจไปต่างประเทศ — EOR vs. entity framing |
| expansion-update | `skills/expansion-update/` | `/employment-legal:expansion-update [country]` | อัปเดต tracker การขยาย — flag overdue, recalculate unblocked items |
| international-expansion | `skills/international-expansion/` | (reference) | กรอบงานวางแผนการจ้างต่างประเทศ — โหลดโดยสกิลขยายธุรกิจ |
| leave-tracker | `skills/leave-tracker/` | `/employment-legal:leave-tracker` | เช็ค leave ที่เปิดอยู่ — alert เฉพาะที่ต้อง decide |
| log-leave | `skills/log-leave/` | `/employment-legal:log-leave` | เพิ่มรายการลาเข้า leave register ทีละราย |
| investigation-open | `skills/investigation-open/` | `/employment-legal:investigation-open` | เปิดคดีสอบสวนภายในใหม่ — intake + sources checklist + log |
| investigation-add | `skills/investigation-add/` | `/employment-legal:investigation-add` | เพิ่มเอกสาร/บันทึกการสัมภาษณ์เข้าสำนวนสอบสวน |
| investigation-query | `skills/investigation-query/` | `/employment-legal:investigation-query` | ถามคำถามต่อ investigation log — ใครพูดอะไร, ข้อขัดแย้งระหว่างผู้ให้การ, ช่องว่างของหลักฐาน |
| investigation-memo | `skills/investigation-memo/` | `/employment-legal:investigation-memo` | ร่างบันทึกการสอบสวนแบบ privileged (ใช้ attorney-client privilege) |
| investigation-summary | `skills/investigation-summary/` | `/employment-legal:investigation-summary` | สรุปสำหรับ HR / leadership / outside counsel แยก audience |
| internal-investigation | `skills/internal-investigation/` | (reference) | กรอบงานสอบสวนภายในร่วม — โหลดโดยสกิล investigation-* |
| matter-workspace | `skills/matter-workspace/` | `/employment-legal:matter-workspace` | สร้าง/list/switch/close workspace สำหรับสำนักงานที่รับหลายลูกค้า |

### กลุ่ม "investigation" (5 skill + 1 reference)

`investigation-open → investigation-add → investigation-query → investigation-memo → investigation-summary` — pipeline เต็มของการสอบสวนภายใน (เช่น คดี harassment, retaliation) โดยมี `internal-investigation` reference skill เป็น framework กลางที่ทุกสกิลโหลดเข้ามา

### กลุ่ม "leave" (2 skill + 1 agent)

`log-leave` (เพิ่มข้อมูล) + `leave-tracker` (เช็ค deadline) + agent `leave-tracker` (รันสัปดาห์ละครั้ง) — ออกแบบมาเพื่อไม่ให้ designation notice, medical certification หรือ exhaustion analysis พลาดเส้นตาย

---

## Agent — leave-tracker

`agents/leave-tracker.md` (12 KB) — agent ที่ตั้งใจให้รันสัปดาห์ละครั้ง (เช้าวันจันทร์) เพื่อเฝ้า "นาฬิกาตามกฎหมาย (statutory clocks)" ที่ทนายส่วนใหญ่ไม่ได้นั่งจับเวลา

**ขอบเขตการเฝ้า**:
- FMLA (Family and Medical Leave Act — กฎหมายลาดูแลครอบครัว/รักษาตัวของรัฐบาลกลางสหรัฐ ฯ)
- รัฐต่าง ๆ ที่มีกฎคล้ายกัน (CA CFRA, NY PFL, CO FAMLI, WA PFML, OR PFML)
- USERRA (Uniformed Services Employment and Reemployment Rights Act — กฎหมายสิทธิคืนตำแหน่งของทหารกองหนุน)
- ADA leave as accommodation (ลาเพื่อปรับสภาพการทำงานตาม Americans with Disabilities Act)

**ไม่เฝ้า** — PTO, bereavement, jury duty เพราะไม่มี statutory deadline

**Alert tiers**:
- IMMEDIATE ACTION — ต้องตัดสินใจภายใน 3 วันทำการ
- ACTION NEEDED THIS WEEK — ภายใน 7 วัน
- COMING UP — ภายในประมาณ 30 วัน

**ข้อระวังสำคัญ** — agent ตัวนี้ "ไม่ self-schedule" ผู้ใช้ต้องตั้ง reminder เอง (cron / calendar) เพราะ Claude Code agent ไม่มีตัวรัน scheduler ในตัว — ถ้าต้องการรันอัตโนมัติเต็มรูปแบบ ใช้ template ของ `managed-agent-cookbooks/` ผ่าน Managed Agent API

---

## Hooks

ไฟล์ `hooks/hooks.json` มีเนื้อหา `{ "hooks": {} }` — ว่างเปล่าโดยตั้งใจ เป็น placeholder สำหรับ plugin ที่อาจเพิ่ม PreToolUse/PostToolUse hooks ในอนาคต (เช่น redact PII ก่อนส่งให้ MCP)

---

## Data folder

`data/.gitkeep` — โฟลเดอร์ว่างที่ติดมากับ plugin เพื่อรองรับการเก็บไฟล์ที่ผู้ใช้สร้าง อย่างไรก็ตามไฟล์จริงเช่น `leave-register.yaml`, investigation logs, expansion trackers จะถูกเขียนไปยัง path version-independent คือ `~/.claude/plugins/config/claude-for-legal/employment-legal/` — เพื่อให้รอด plugin update และเก็บข้อมูล privileged แยกจาก codebase

---

## Connectors (MCP)

จาก `.mcp.json`:

```json
{
  "mcpServers": {
    "Slack": { "type": "http", "url": "https://mcp.slack.com/mcp" },
    "Google Drive": { "type": "http", "url": "https://drivemcp.googleapis.com/mcp/v1" }
  },
  "recommendedCategories": ["hris", "documents", "chat", "email"]
}
```

- **Slack** — ค้นข้อความ, อ่าน channel (ใช้ตอน internal investigation ดึงประวัติการสื่อสาร)
- **Google Drive** — ค้นและอ่านเอกสาร (handbook, offer letter, termination memo)
- **`recommendedCategories`** แนะนำให้ผู้ใช้เชื่อมต่อ HRIS เพิ่ม (Workday, BambooHR, Rippling) เพื่อให้ `leave-tracker` ดึง active leave จาก HRIS ตรง ๆ ได้ ไม่ต้อง maintain `leave-register.yaml` ด้วยมือ

---

## Workflow ตัวอย่าง

### Case 1 — เลิกจ้างพนักงานในแคลิฟอร์เนีย

```
User: /employment-legal:termination-review
User: Engineering manager ที่ CA, performance issue 6 เดือน, ทีมเคยมี PIP
       (performance improvement plan) เมื่อ 4 เดือนก่อน อายุ 52 ปี
       severance ที่เสนอ = 8 สัปดาห์

Skill:
1. โหลด profile → CA jurisdiction → flag CA final pay rule (จ่ายทันทีวันสุดท้าย)
2. flag high-risk: อายุ ≥ 40 → ADEA + OWBPA (Older Workers Benefit
   Protection Act) consideration period 21 วัน + 7 วัน revocation
3. flag PIP gap — 4 เดือนระหว่าง PIP กับ termination → ต้อง document
   ว่าทำไมยังเก็บไว้ + performance ปัจจุบัน
4. flag CA-specific: severance ≥ 6 เดือนที่ตำแหน่ง manager → ตรวจสอบ
   CA WARN Act ถ้าเป็น layoff หลายคน
5. ร่าง release agreement พร้อม CA-specific carve-outs (ไม่ release
   wage claims, ไม่ release indemnification rights)
6. escalate → GC ตามตาราง escalation
```

### Case 2 — Worker classification — IC vs. employee

```
User: /employment-legal:worker-classification
User: data scientist 6 เดือน, work-from-home ใน CA, ใช้อุปกรณ์ของเรา,
      เข้าประชุมรายสัปดาห์, ค่าจ้าง flat monthly

Skill:
1. CA → run ABC test (Assembly Bill 5)
   - A: free from control? — ใช้อุปกรณ์เรา + ประชุมประจำ → ไม่ผ่าน
   - B: outside usual course of business? — data scientist สำหรับบริษัท
     tech → ไม่ผ่าน
   - C: independently established trade? — ต้อง verify
2. Misclassification gap: 2/3 prong ไม่ผ่าน → ควรเป็น W-2 employee
3. Flag — CA penalty + back wages + benefits liability ถ้า audit
4. แนะนำ: convert เป็น W-2 part-time หรือ ใช้ EOR (Employer of Record)
   หากต้องการความยืดหยุ่น
```

---

## ข้อควรระวัง

- **กฎแยกตามรัฐคือหัวใจ** — plugin ตั้งใจ "ไม่เก็บ" ตัวเลขเฉพาะรัฐ (salary threshold, covenant enforceability, final-pay timing, release consideration) ไว้ในไฟล์ ทุก rule ต้อง research + cite ตอน review เพราะกฎมลรัฐเปลี่ยนบ่อย — ตัวอย่าง paid leave program ของรัฐต่าง ๆ ออกใหม่/แก้ไขเกือบทุกปี
- **leave-tracker ไม่ตัดสินใจเลิกจ้าง** — บอกแค่ว่า process อะไรต้องทำก่อนตัดสินใจ (เช่น ADA interactive process ก่อน separation หลัง FMLA exhaustion)
- **Investigation logs ต้อง privileged** — เก็บใน `~/.claude/plugins/config/...` ซึ่ง access-controlled ไม่ commit เข้า git repo สาธารณะ
- **Worker classification เป็น prospective only** — สำหรับการจ้างใหม่ ความสัมพันธ์ที่มีอยู่แล้วต้อง consult counsel ไม่ใช่รัน skill
- **Termination review ไม่ทดแทนคุยกับ HR/manager** — เป็น checklist กันลืม ไม่ใช่กระบวนการตัดสินใจเลิกจ้างที่สมบูรณ์
- **Citation tagging** — citation จาก research tool tag ด้วยแหล่งที่มา; citation จากความรู้ของ model หรือ web search tag `[verify]` ต้อง check ก่อนใช้
- **Outside counsel** — สำหรับเขตอำนาจใหม่หรือ close call ต้องส่ง outside counsel ไม่ใช้ output ของ plugin โดยตรง
