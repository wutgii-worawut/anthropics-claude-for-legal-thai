---
title: privacy-legal
parent: 03 Plugins
nav_order: 2
---

# privacy-legal — กฎหมายความเป็นส่วนตัว

> "Triages processing activities, generates PIAs, reviews DPAs as controller or processor, drafts DSAR responses within statutory timelines, and monitors policy drift against practice."
> — `privacy-legal/.claude-plugin/plugin.json`

## 1. บทนำ

`privacy-legal` เป็น plugin สำหรับ **ทนายและทีม privacy/compliance** ที่ต้องดูแล data protection program — งานที่เกิดทุกวันสำหรับ in-house privacy counsel: triage processing activity ใหม่, สร้าง **การประเมินผลกระทบความเป็นส่วนตัว (PIA — Privacy Impact Assessment)**, review **ข้อตกลงประมวลผลข้อมูล (DPA — Data Processing Agreement)** ทั้งฝั่ง controller และ processor, draft คำตอบให้ **คำขอใช้สิทธิของเจ้าของข้อมูล (DSAR — Data Subject Access Request)** ภายในกำหนดเวลาตามกฎหมาย และเฝ้าระวัง policy drift เมื่อ practice เปลี่ยนแต่ privacy policy ยังไม่ตาม

plugin นี้ออกแบบให้ทำงานข้าม regime — GDPR, CCPA/CPRA, state privacy laws (VA, CO, CT, UT, ...), COPPA 2025, EU-US DPF — โดยมีไฟล์ `references/currency-watch.md` เป็น "staleness check" เตือนเมื่อ rule เปลี่ยน

**เหมาะกับ**: in-house privacy counsel, DPO (Data Protection Officer), GRC team, privacy program manager

## 2. `.claude-plugin/plugin.json`

```json
{
  "name": "privacy-legal",
  "version": "1.0.2",
  "description": "Triages processing activities, generates PIAs, reviews DPAs as controller or processor, drafts DSAR responses within statutory timelines, and monitors policy drift against practice.",
  "author": {
    "name": "Anthropic"
  }
}
```

| ฟิลด์ | ค่า | ความหมาย |
|------|----|---------|
| `name` | `privacy-legal` | namespace ของคำสั่ง (เช่น `/privacy-legal:dpa-review`) |
| `version` | `1.0.2` | semver — ตรงกันทั้ง 12 plugin |
| `description` | (ยาว) | คำอธิบายแสดงใน marketplace listing |
| `author.name` | `Anthropic` | upstream maintainer |

## 3. โครงสร้างโฟลเดอร์

```text
privacy-legal/
├── .claude-plugin/
│   └── plugin.json
├── .gitignore
├── .mcp.json                    # Slack + Google Drive (เบาที่สุดใน Group 1)
├── CLAUDE.md                    # practice profile template (43.6 KB)
├── README.md                    # (6.9 KB)
├── agents/                      # ว่าง (privacy-legal ไม่มี scheduled agent)
├── hooks/
│   └── hooks.json               # {} placeholder
├── references/
│   └── currency-watch.md        # staleness check สำหรับ privacy rule (3 KB)
└── skills/                      # 9 skills
    ├── cold-start-interview/
    ├── customize/
    ├── dpa-review/
    ├── dsar-response/
    ├── matter-workspace/
    ├── pia-generation/
    ├── policy-monitor/
    ├── reg-gap-analysis/
    └── use-case-triage/
```

## 4. Skills ทั้งหมด

9 skill — ทุกตัวเรียกได้โดยตรง (ไม่มี reference skill แบบ commercial-legal)

| Skill | Path | คำสั่ง | สำหรับ |
|-------|------|--------|---------|
| `use-case-triage` | `skills/use-case-triage/` | `/privacy-legal:use-case-triage` | ตัดสินว่า processing activity ใหม่ต้องทำ PIA, mandatory GDPR DPIA หรือผ่านไปได้ — surface policy conflict |
| `pia-generation` | `skills/pia-generation/` | `/privacy-legal:pia-generation` | สร้าง PIA ตาม house format โดยเรียนรู้โครงสร้างจาก seed PIA ของลูกค้า |
| `dpa-review` | `skills/dpa-review/` | `/privacy-legal:dpa-review` | review DPA — auto-detect ว่าลูกค้าเป็น controller หรือ processor แล้ว apply playbook ฝั่งที่ถูกต้อง |
| `dsar-response` | `skills/dsar-response/` | `/privacy-legal:dsar-response` | walk through DSAR (access/deletion/portability/correction) — verify identity, locate data system-by-system, assess exemption, draft response |
| `policy-monitor` | `skills/policy-monitor/` | `/privacy-legal:policy-monitor` | weekly sweep PIA/DPA review/triage result เพื่อหา **policy drift** — หรือ direct query สำหรับ practice ใหม่ ("does our policy cover this?") |
| `reg-gap-analysis` | `skills/reg-gap-analysis/` | `/privacy-legal:reg-gap-analysis` | diff regulation ใหม่กับ policy + practice ปัจจุบัน — ออก gap list + remediation plan (มี owner + date) |
| `cold-start-interview` | `skills/cold-start-interview/` | `/privacy-legal:cold-start-interview` | สัมภาษณ์ครั้งแรก — เขียน `CLAUDE.md` จาก policy + DPA template + reference PIA ของลูกค้า |
| `customize` | `skills/customize/` | `/privacy-legal:customize` | ปรับ profile โดยไม่ต้องเริ่ม cold-start ใหม่ — risk posture, DPA playbook, PIA house style, DSAR process |
| `matter-workspace` | `skills/matter-workspace/` | `/privacy-legal:matter-workspace` | จัดการ matter workspace สำหรับ multi-client practitioner |

**ลำดับใช้งานทั่วไป**: `use-case-triage` → ถ้าต้อง PIA → `pia-generation`; ตามมาด้วย `dpa-review` ตอน vendor sign deal; เมื่อ DSAR เข้า → `dsar-response`; พอนานเข้า `policy-monitor` เรียกใช้ weekly เพื่อจับ drift

## 5. Agents

**privacy-legal ไม่มี scheduled agent** — โฟลเดอร์ `agents/` ว่าง (ตรงข้ามกับ commercial-legal ที่มี 3 agent)

เหตุผลที่น่าจะเป็นไปได้: งาน privacy ส่วนใหญ่เป็น **event-driven** (DSAR เข้า, regulation ใหม่ประกาศ, vendor ส่ง DPA มา) — ไม่ใช่ scheduled pattern แบบ renewal-watch ใน commercial

## 6. Hooks

```text
privacy-legal/hooks/hooks.json
```

เนื้อหา: `{ "hooks": {} }` — placeholder เปล่าเช่นเดียวกับ plugin อื่นใน Group 1

## 7. References

**`references/currency-watch.md`** เป็นไฟล์เด่นของ plugin นี้ — เป็น "staleness check sheet" ที่บอกว่ากฎ privacy ใดมีโอกาสเปลี่ยนตั้งแต่ training cutoff ของโมเดล

โครงสร้าง:

- **Last verified date** ที่ด้านบนสุด — ถ้าเก่ากว่า 90 วัน ให้ถือว่า stale และต้อง verify ก่อนใช้
- **COPPA 2025 amendments** — deadline 22 เมษายน 2026 (biometric/government ID เป็น "personal information", verifiable parental consent สำหรับ third-party disclosure ที่เชื่อมกับ targeted ad)
- **State privacy laws** — รายชื่อ ณ พ.ค. 2026: CA, VA, CO, CT, UT, IA, IN, TN, MT, OR, TX, FL, DE, NH, NJ, KY, MD, MN, NE, RI
- **Cross-border transfers** — EU-US DPF (ก.ค. 2023, อยู่ระหว่าง Schrems III), UK-US Data Bridge, Swiss-US DPF
- **FTC enforcement trends** — Humor Rainbow/OkCupid, health data, dark patterns
- **DSAR response timelines** — CCPA 45+45 วัน, GDPR 1 เดือน + 2 เดือน

แนวคิดสำคัญ: "**stale watch list แย่กว่าไม่มี watch list** เพราะดูเหมือนทันสมัยทั้งที่ผิด" — เมื่อ skill อ้างถึง rule ต้องบอกว่า "verify at [source]"

## 8. Connectors (MCP)

จาก `.mcp.json` — **2 servers เท่านั้น** (น้อยที่สุดใน Group 1):

| MCP server | URL | สำหรับ |
|-----------|-----|--------|
| Slack | `mcp.slack.com/mcp` | search message, read channel — สำหรับ track discussion เรื่อง processing activity |
| Google Drive | `drivemcp.googleapis.com/mcp/v1` | search/read/fetch document — PIA, DPA, privacy policy |

`recommendedCategories`: `documents`, `chat`, `email`, `legal-document-management`

**เหตุที่น้อย**: งาน privacy ไม่มี vendor-specific tool เท่าฝั่ง contracts (ไม่มี "Ironclad ของ privacy") — DSAR และ PIA tracking มักทำใน custom Drive/Notion/Sharepoint ที่อยู่ใน Drive อยู่แล้ว

## 9. Workflow ตัวอย่าง

### Case 1 — Feature ใหม่ที่ต้องการเก็บข้อมูลผู้ใช้

1. PM ถามใน Slack: "เราจะเก็บ phone number สำหรับ 2FA ของฟีเจอร์ใหม่ — ต้องทำ PIA ไหม?"
2. ทนาย privacy พิมพ์: `/privacy-legal:use-case-triage "เก็บ phone number สำหรับ 2FA"`
3. `use-case-triage` ตรวจ policy + applicable law → output:
   - **PIA needed**: ใช่ (sensitive identifier + new processing purpose)
   - **GDPR DPIA mandatory**: ไม่ (ไม่ใช่ large-scale + ไม่ใช่ special category)
   - **Policy conflict**: ไม่ (policy ครอบคลุม authentication purpose แล้ว)
4. ทนายเรียก `/privacy-legal:pia-generation "2FA phone number collection"` → ได้ draft PIA ใน house format
5. review draft + verify ตาม `currency-watch.md` (COPPA ถ้ามีผู้ใช้ใต้ 13) → ส่งให้ approver

### Case 2 — DSAR เข้าทาง privacy@company.com

1. อีเมล: "ขอข้อมูลทั้งหมดที่บริษัทเก็บเกี่ยวกับผม — ตามสิทธิ์ CCPA"
2. ทนาย paste request เข้า Claude: `/privacy-legal:dsar-response`
3. skill walk through:
   - **Step 1 — Verify identity** (เกณฑ์ตาม CCPA)
   - **Step 2 — Locate data** ทุก system ที่อาจมีข้อมูล (CRM, support, marketing, analytics, ...)
   - **Step 3 — Assess exemption** (litigation hold, security exception, ...)
   - **Step 4 — Draft acknowledgment letter** (ต้อง confirm ภายใน 10 วันตาม CCPA)
   - **Step 5 — Draft substantive response** (ภายใน 45 วัน + 45 วัน extension)
4. ทนายดูตรงไหนต้องการ verify (timeline ของ state อื่น) → `currency-watch.md` มี table CCPA 45+45 / GDPR 1m+2m

### Case 3 — Quarterly policy drift sweep

1. ทุก quarter ทนาย privacy รัน: `/privacy-legal:policy-monitor --sweep`
2. skill crawl saved PIA, DPA review, triage result ที่ผ่านมา 3 เดือน
3. report ออกมาว่า:
   - PIA #14 บอกว่า "เก็บ behavioral data สำหรับ recommendation" — ไม่อยู่ใน current policy
   - DPA review #23 ระบุ subprocessor ใหม่ที่ไม่ได้ list ใน policy
4. ทนายเปิด `/privacy-legal:reg-gap-analysis` กับ policy ปัจจุบัน → ออก remediation plan (owner + date)
5. update policy → ส่งให้ comms ประกาศ

## 10. ข้อควรระวัง

- **PIA ที่ generate ออกมาไม่ใช่ "พร้อม submit ต่อ regulator"** — เป็น first draft ที่ทนายต้อง review, ปรับ tone และ verify ตาม current law ก่อน
- **DPA auto-detect controller/processor ไม่ใช่ legal conclusion** — เป็นการอ่าน language pattern ของ DPA หากเป็น dual-role (controller-to-controller + controller-to-processor) plugin จะถามให้ confirm
- **DSAR timeline ใน `currency-watch.md` อาจ stale** — ทุกครั้งให้ verify state law ที่ใช้บังคับ (โดยเฉพาะ state ใหม่ที่เพิ่งมีผล)
- **Policy monitor weekly sweep ทำงานจาก saved artifact** — ถ้าทีมไม่ได้ save PIA/DPA review ไว้ในที่ที่ plugin scan ได้ ก็จะตรวจไม่เจอ drift
- **ภาษาในตัวอย่าง EU เป็น GDPR baseline** — สำหรับลูกค้าที่ขายในไทย ต้อง bridge กับ **PDPA (Personal Data Protection Act พ.ศ. 2562)** เอง — plugin ไม่ได้ map PDPA โดยตรง (ทำผ่าน `customize` + เพิ่ม jurisdiction-specific section ใน `CLAUDE.md`)
- **ทุก output ต้องผ่าน attorney review** — ตามข้อ disclaimer ใน `README.md` ของ source

---

**Source**: [`anthropics/claude-for-legal/privacy-legal/`](https://github.com/anthropics/claude-for-legal/tree/main/privacy-legal)
**Version**: 1.0.2
**Last reviewed**: 2026-05-13
