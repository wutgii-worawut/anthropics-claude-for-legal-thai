---
title: ai-governance-legal (กำกับดูแล AI)
parent: 03 Plugins
nav_order: 7
---

# ai-governance-legal — Plugin กำกับดูแล AI (AI ใช้กับ AI)

## บทนำ

`ai-governance-legal` เป็น plugin "เมตา" — เป็น AI ที่ช่วยกำกับดูแลการใช้ AI ในองค์กร plugin นี้สร้างขึ้นรอบงานหลัก 4 ส่วน:
1. **AI Use-Case Triage** — คัดกรอง use case ที่เสนอเข้ามาเทียบกับ AI registry (ทะเบียน AI system) ขององค์กร — approved / conditional / not approved
2. **AI Impact Assessment (AIA)** — การประเมินผลกระทบเชิงอัลกอริทึม (algorithmic impact assessment) ครอบคลุมทุก regime ที่ in scope (EU AI Act, US state AI laws, NIST RMF ฯลฯ)
3. **Vendor AI Review** — ตรวจสัญญา vendor AI โดยเน้น "training-on-data clauses" (เงื่อนไขการนำข้อมูลลูกค้าไป train โมเดล), liability, model changes, AI policy alignment
4. **Reg Gap Analysis + Policy Monitor** — diff กฎใหม่กับ posture ปัจจุบัน + เฝ้า drift ระหว่าง AI policy กับ practice จริง

หัวใจคือ "**builder vs. deployer obligations**" — ภาระทางกฎหมายต่าง regime แยกผู้สร้างกับผู้ใช้ AI (เช่น EU AI Act แยก provider/deployer/importer/distributor/authorized representative) — ทีมต้องรู้ว่าตัวเองสวมหมวกไหนในแต่ละ use case

เหมาะกับ — privacy/AI governance counsel, product counsel ที่ launch ผลิตภัณฑ์มี AI component, GC ในเรื่อง board-level AI risk, procurement counsel ตรวจสัญญา AI vendor

---

## `.claude-plugin/plugin.json`

```json
{
  "name": "ai-governance-legal",
  "version": "1.0.2",
  "description": "Triages proposed AI use cases against your registry, runs impact assessments across the regimes in scope, reviews vendor AI terms for training-on-data and liability gaps, and keeps your AI policy current with practice.",
  "author": {
    "name": "Anthropic"
  }
}
```

| Field | คำอธิบาย |
|---|---|
| `name` | `ai-governance-legal` |
| `version` | 1.0.2 |
| `description` | สรุป — triage use case, run impact assessment, review vendor AI terms (training-on-data + liability gap), keep AI policy current |
| `author.name` | Anthropic |

---

## โครงสร้างโฟลเดอร์

```text
ai-governance-legal/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json                    # MCP — Slack, Google Drive
├── .gitignore
├── CLAUDE.md                    # template practice profile (44 KB — ใหญ่สุดในกลุ่ม)
├── README.md
├── references/
│   └── currency-watch.md        # ตาราง AI law ปัจจุบัน + last-verified date
└── skills/                      # 10 skills (ไม่มี agents/ folder)
    ├── ai-inventory/            # EU AI Act per-system inventory
    ├── aia-generation/          # AI impact assessment
    ├── cold-start-interview/
    ├── customize/
    ├── matter-workspace/
    ├── policy-monitor/          # weekly drift sweep + direct query
    ├── policy-starter/          # ร่าง AI policy แรกจาก published model policies
    ├── reg-gap-analysis/        # diff กฎใหม่กับ posture
    ├── use-case-triage/         # คัดกรอง use case vs. registry
    └── vendor-ai-review/        # ตรวจสัญญา vendor AI
```

ข้อสังเกต — plugin นี้ไม่มี `agents/` folder (ไม่มี scheduled agent ใน Claude Code) แต่ `policy-monitor` skill ทำหน้าที่ weekly sweep เมื่อถูกเรียก

---

## Skills ทั้งหมด

| Skill | Path | คำสั่ง | สำหรับ |
|---|---|---|---|
| cold-start-interview | `skills/cold-start-interview/` | `/ai-governance-legal:cold-start-interview` | สัมภาษณ์ครั้งแรก — เรียน practice (builder/deployer/both), regime ที่บังคับใช้, red line ของ use case, house style ของ impact assessment |
| customize | `skills/customize/` | `/ai-governance-legal:customize` | ปรับ profile ทีละจุด — risk posture, regime ใหม่, etc. |
| use-case-triage | `skills/use-case-triage/` | `/ai-governance-legal:use-case-triage [use case]` | คัดกรอง use case vs. registry — approved/conditional/not approved + flag missing assessment |
| aia-generation | `skills/aia-generation/` | `/ai-governance-legal:aia-generation [use case]` | รัน AI impact assessment — intake, risk analysis, regulatory classification ตาม regime, policy consistency check, mitigation conditions |
| ai-inventory | `skills/ai-inventory/` | `/ai-governance-legal:ai-inventory` | EU AI Act per-system inventory — บันทึกบทบาท (provider/deployer/importer/distributor/authorized rep) ของแต่ละระบบ AI |
| vendor-ai-review | `skills/vendor-ai-review/` | `/ai-governance-legal:vendor-ai-review [file/vendor]` | ตรวจสัญญา/ToS ของ vendor AI — flag training-on-data, liability, model changes, AI policy fit |
| reg-gap-analysis | `skills/reg-gap-analysis/` | `/ai-governance-legal:reg-gap-analysis [regulation]` | diff กฎใหม่ vs. governance posture — surface gap + remediation plan with owner/deadline |
| policy-monitor | `skills/policy-monitor/` | `/ai-governance-legal:policy-monitor` | weekly sweep ของ AIA, triage, vendor review เพื่อหา policy drift; รองรับ direct query สำหรับ practice ที่เสนอใหม่ |
| policy-starter | `skills/policy-starter/` | `/ai-governance-legal:policy-starter` | ร่าง AI usage policy แรกจาก published model policies (ABA, state bars, ILTA, CLOC, NIST, EU AI Act, peer policies) ปรับตาม practice profile |
| matter-workspace | `skills/matter-workspace/` | `/ai-governance-legal:matter-workspace` | จัดการ matter workspace สำหรับ private practice หลายลูกค้า |

### กลุ่ม "use case lifecycle" (3 skills)

```
use-case-triage  →  approved / conditional / not approved
       ↓ (conditional หรือ high-risk)
aia-generation   →  full impact assessment + mitigation conditions
       ↓ (ลงทะเบียน)
ai-inventory     →  บันทึกใน EU AI Act inventory ถ้าเกี่ยวข้อง
```

### กลุ่ม "policy lifecycle" (3 skills)

```
policy-starter    →  ร่างแรกของ AI usage policy
       ↓
reg-gap-analysis  →  diff regulation ใหม่กับ posture
       ↓
policy-monitor    →  weekly drift sweep — practice แยกจาก policy?
```

---

## Agents

ไม่มี agent ใน Claude Code (`agents/` folder ไม่มี) — `policy-monitor` skill ทำงานคล้าย agent เมื่อรันเป็น weekly sweep แต่ผู้ใช้ต้องเรียกเอง

ถ้าต้องการ scheduled run แบบ managed — `managed-agent-cookbooks/` ไม่มี template เฉพาะ ai-governance ในเวอร์ชันปัจจุบัน — ใช้ `reg-monitor` cookbook เป็นแม่แบบและปรับให้ trigger policy-monitor

---

## Hooks

plugin นี้ไม่มี `hooks/` folder (ไม่มี hook configuration)

---

## References

`references/currency-watch.md` — ไฟล์พิเศษที่เก็บ "ตาราง AI law ปัจจุบัน" พร้อม "last-verified date" — เพราะกฎหมาย AI เคลื่อนเร็วกว่า training data ของ model

**โครงสร้างไฟล์**:
- last-verified date ที่ด้านบน — ถ้าเกิน 90 วัน skill ที่อ่านจะแจ้งว่า "stale checklist เท่านั้น ไม่ใช่ source of truth"
- ตาราง US state AI laws — Colorado SB 24-205, Texas TRAIGA, Nebraska LB 525, Maine LD 2082, NYC Local Law 144, Illinois AIPA + HB 3773 ฯลฯ พร้อมสถานะ (in force / effective date postponed / pending) + ลิงก์ verify
- EU AI Act implementation — Digital Omnibus, implementing acts, national transposition
- Federal (US) — EEOC, FTC §5 (FTC v. Humor Rainbow/OkCupid case), executive orders

**หลักการใช้** — เมื่อ skill จะอ้างถึง effective date / threshold / obligation ต้องเช็คตาราง currency-watch ก่อน + note ว่า "AI law moving fast — date/rule อาจเปลี่ยน" + verify ที่ source ก่อนสรุปให้ผู้ใช้

> "stale watch list is worse than no watch list — it looks current while being wrong" — quote จากตัว reference

---

## Connectors (MCP)

จาก `.mcp.json`:

```json
{
  "mcpServers": {
    "Slack": { "type": "http", "url": "https://mcp.slack.com/mcp" },
    "Google Drive": { "type": "http", "url": "https://drivemcp.googleapis.com/mcp/v1" }
  },
  "recommendedCategories": ["documents", "chat", "email"]
}
```

- **Slack** — หา conversation เกี่ยวกับ AI use case ที่ทีมคุยกันก่อนเข้า triage formal
- **Google Drive** — เข้าถึง AI policy, AIA history, vendor agreement
- **`recommendedCategories`** เน้น documents + chat + email — plugin ไม่ต้องการ legal research connector specific (ต่างจาก `regulatory-legal`) เพราะ AI regime ส่วนใหญ่ยังไม่มีฐานข้อมูลพิเศษเหมือนงาน regulatory

---

## Workflow ตัวอย่าง

### Case 1 — Sales อยากใช้ AI score leads อัตโนมัติ

```
User: /ai-governance-legal:use-case-triage
      "Sales team wants to use AI to score leads automatically"

Skill:
1. โหลด profile → company มี practice = deployer (ใช้ vendor model)
   + regime in scope = NYC Local Law 144 (ถ้ามี hiring decision component)
   + Illinois AIPA + HB 3773 + GDPR (Lead เป็นบุคคล EU)
2. คัดกรอง vs. registry — ไม่อยู่ใน registry → new use case
3. risk tier — มี automated decision affecting humans + commercial impact
   → conditional (ไม่ใช่ approved by default)
4. เงื่อนไข:
   - ต้องรัน AIA ก่อน production
   - ต้อง human-in-the-loop หรือ explainability disclosure
   - vendor ต้องผ่าน vendor-ai-review (training-on-data + liability)
   - ถ้ามี employment context → coordinate กับ employment-legal
5. ส่งต่อ: /ai-governance-legal:aia-generation
```

### Case 2 — ตรวจสัญญา OpenAI Enterprise

```
User: /ai-governance-legal:vendor-ai-review openai-terms.pdf

Skill output:
🔴 Training-on-data clause — flag
   "OpenAI will not use Customer Content to train models" — ✓ aligned with
   policy position #3 (no training on our data)
   → ต้องระบุชัดเจนว่ารวมถึง fine-tuning data ไม่แค่ prompts/completions

🟡 Liability cap — review
   $1M general cap; AI-specific carve-out: indemnity for output infringement
   only "where Customer used in compliance with usage policies"
   → conditional indemnity = practical gap; redline แนะนำ:
     unconditional indemnity for IP claims arising from training data

🔴 Model changes notification — gap
   "OpenAI may update models at any time without notice" — ไม่ตรงกับ
   policy position #7 (require 30-day notice + parallel-run option for
   production critical workflows)
   → redline แนะนำ + escalate to procurement

📋 AI policy alignment — 3 gaps ระบุข้างต้น
📋 Currency check (currency-watch.md last verified 2026-05-10):
   - EU AI Act provider obligations apply if model is deployed in EU
   - check vendor's CE marking + risk classification
```

### Case 3 — Policy drift sweep (weekly)

```
User: /ai-governance-legal:policy-monitor

Skill output:
📊 Policy drift sweep — week of 2026-05-11

🔴 Drift detected (2)
1. Practice: ทีม ML ใช้ Anthropic API สำหรับ customer-facing chatbot
   Policy: บัญญัติ "approved vendor list = OpenAI, Google, Azure only"
   → ต้องอัปเดต policy เพิ่ม Anthropic หรือ enforce list เดิม
   → owner: @ai-gov-counsel, due 2026-05-25

2. Practice: AIA 3 รายการล่าสุดข้าม "EU AI Act risk classification" step
   Policy: บัญญัติ classification "required for all AIA"
   → ตรวจว่า template ปัจจุบันมี step นี้หรือไม่ — ถ้าหายให้ใส่กลับ

🟡 Aligned but worth noting (5) ...
```

---

## ข้อควรระวัง

- **AI law moving fast** — `references/currency-watch.md` ต้อง verify ทุก 90 วัน ทุก skill ต้องเช็ค last-verified date ก่อนอ้างถึง effective date / threshold
- **Builder vs. deployer** — ภาระทางกฎหมายต่างกันมาก ทีมที่ "build เอง + deploy เอง" สวมสองหมวกพร้อมกัน skill จะถาม "หมวกไหนสำหรับ task นี้" ถ้าไม่ชัด
- **Registry คือฐาน triage** — `use-case-triage` ดีเท่าที่ registry แม่นยำ — ตอน cold-start ต้องตั้ง "red line" ให้ดี (use case ที่ห้ามอย่างเด็ดขาด เช่น "AI ตัดสินใจ hire/fire เอง", "AI giving legal advice to clients")
- **Impact assessment ใช้รูปแบบ house** — มาจาก seed AIA ที่ผู้ใช้ให้ตอน cold-start ถ้าไม่ได้ให้จะใช้ baseline structure — แนะนำ rerun cold-start พร้อม reference เพื่อให้ format ตรงกับ practice
- **Gap analysis = manual** — ต้องชี้ regulation เอง — ถ้าต้องการ automated monitoring ของกฎใหม่ คู่กับ `regulatory-legal` plugin
- **Policy monitor ต้อง outputs folder** — config ตอน setup — ถ้าไม่ได้ตั้ง direct query mode ใช้ได้ แต่ sweep weekly จะไม่ทำงาน
- **Plugin triangle** — `ai-governance-legal` + `product-legal` + `privacy-legal` ออกแบบให้ทำงานคู่กัน:
  - product-legal เจอ AI component → handoff ไป `use-case-triage` + `aia-generation`
  - privacy-legal เจอ AI use case ที่มี personal data → handoff ไป `pia-generation`
  - ai-governance เจอ data protection issue → handoff ไป `pia-generation`
- **Company profile shared** — `## Company profile` block ของ CLAUDE.md ใช้ร่วมกันได้กับ 12 plugins อื่น ไม่ต้องกรอกข้อมูลซ้ำ
