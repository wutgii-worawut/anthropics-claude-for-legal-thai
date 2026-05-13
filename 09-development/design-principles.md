---
title: Design Principles
parent: 09 Development
nav_order: 4
---

# Design Principles — หลักการที่อยู่เบื้องหลังทุก skill

> "Keep this short — the design principles that matter most for the quality of the output, not a style guide."
> — `CONTRIBUTING.md`

หมวดนี้รวบรวม **หลักการออกแบบ** ที่ Anthropic ใช้ตลอด codebase ของ Claude for Legal — ไม่ใช่ style guide แต่เป็น **กฎที่ฝังตัว** ในทุก skill ทุก plugin ที่ทำให้ระบบให้ผลผลิตที่น่าเชื่อถือสำหรับงานทนายความ

ทำความเข้าใจหลักเหล่านี้ก่อนแก้ skill ใด ๆ — จะเห็นว่าหลายอย่างที่ดู "ซับซ้อนเกิน" ในตอนแรก จริง ๆ แล้วเป็นการตอบ failure mode ที่ Anthropic เคยพบมาก่อน

## 1. "If guardrail fires, move to SKILL.md"

จาก `CONTRIBUTING.md`:

> *"Rule of thumb: if a QA test passes only because a guardrail fired, add the behavior to the SKILL.md directly. The guardrail stays (belt and suspenders), but the skill now carries the knowledge it needs on its own."*

### หลักการ

หาก skill ผ่าน QA ได้เพราะ **CLAUDE.md guardrail** จับ error → **design smell** เพราะ:

- โมเดลในอนาคตอาจอ่อนลง → guardrail ไม่ fire
- Prompt terse กว่าปกติ → guardrail miss
- Editor ใหม่อ่านแค่ skill text → ไม่เห็น guardrail
- Edge case ที่ guardrail ไม่ครอบคลุม → fail

### ตัวอย่างจาก CONTRIBUTING.md

| Failure | Skill ที่ถูก | Skill ที่ผิด |
|---------|-------------|--------------|
| Design patent question routed to utility-patent workflow | Skill branch บน D-prefix → ordinary-observer test | Skill อาศัย "Scaffolding, not blinders" override |
| Renewal cancel-by ตรงวันอาทิตย์ | Register schema + Mode 2 output ทำ business-day roll-back | User คิดถามเรื่อง weekday |
| FLSA back-pay regular-rate | Skill มี §207(e) checklist | Model จำ §207(e) ได้ |

### ใช้กฎนี้อย่างไร

ก่อน push skill:

1. Run skill ใน edge case
2. ถ้า output ถูกเพราะ guardrail — ถามตัวเอง "Skill ทำได้เองมั้ย?"
3. ถ้าทำไม่ได้ — เพิ่ม doctrine เข้า SKILL.md
4. ถ้าทำได้แต่ไม่อยาก redundant — ใส่ก็ดี (belt and suspenders)

## 2. Citation Discipline — Verbatim Quotes

### หลักการ

ทุก citation ต้องตามรอย source — **ห้าม paraphrase กฎหมายโดยไม่ tag**

### Three states ของ citation

| สถานะ | Tag | ตัวอย่าง |
|--------|-----|----------|
| Verified through research connector | tagged with source | `Smith v. Jones, 123 F.3d 456 (9th Cir. 2020) [via CourtListener]` |
| From model training data | `[verify]` | `Smith v. Jones, 123 F.3d 456 [verify]` |
| No research connector at all | reviewer note | "Sources not verified — connect research tool" |

### Provenance tag rule

จาก `CONTRIBUTING.md`:

> *"Attach provenance tags to numbers, not to paragraphs."*

ทำแบบนี้:

```markdown
Notice deadline: 2026-06-15 [model calculation — verify against §11.2 notice clause]

Back-pay estimate: $45,250 [verify — consult wage-and-hour counsel before asserting or paying]
```

ไม่ใช่:

```markdown
The notice deadline appears to be in June. Some back-pay may be owed.
[verify all calculations]
```

**เหตุผล**: tag บน prose หลุดง่ายเมื่อ format/copy-paste — tag บน digit ติดทน

## 3. Default-Free Design

### หลักการ

**Skill ไม่ตั้งสมมุติฐานเริ่มต้น** — ทุกอย่างมาจาก practice profile หรือถาม user

### ทำไม

- Practice profile แต่ละสำนักงานไม่เหมือนกัน
- Sales-side และ purchasing-side มี posture ต่างกัน
- Jurisdiction แต่ละแห่งมี rule ต่างกัน
- Risk appetite ของแต่ละบริษัทไม่เท่ากัน

หาก skill มี default — เกิด **silent error** เมื่อ default ไม่ตรงกับสำนักงาน

### ตัวอย่าง

**Skill ที่ดี (default-free):**

```markdown
## Step 1: Load practice profile

Read `~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md`.

If profile contains `[PLACEHOLDER]` markers → STOP.
Output: "Run /commercial-legal:cold-start-interview first."

Extract:
- side: sales | purchasing
- escalation_threshold: number
- jurisdictions: list

If any missing → STOP, ask user.
```

**Skill ที่ไม่ดี (มี default):**

```markdown
## Step 1: Apply standard playbook

(default to sales-side, California jurisdiction, $100K escalation)
```

> *"Skipping setup is the single most common reason a skill produces generic output."*
> — `README.md`

## 4. Three States of "Not Found"

### หลักการ

เมื่อ skill หาข้อมูลไม่พบ มี **3 สถานะ** ที่ต่างกัน — ห้าม conflate

| สถานะ | ความหมาย | Output |
|--------|----------|--------|
| **Searched, not found** | ค้นแล้ว → ไม่มี | "Searched [X]; no matches" |
| **Could not search** | เครื่องมือไม่พร้อม | "Could not search [X] (connector unavailable)" |
| **Did not search** | skill ไม่ได้ค้นในนี้ | "Did not search [X]" |

### ทำไมสำคัญ

ทนายต้องรู้ว่า "no matches" หมายถึงอะไรจริง ๆ:

- "Searched Westlaw, no controlling authority" → ปลอดภัยที่จะ proceed
- "Could not search Westlaw (auth expired)" → ต้องค้นเองก่อน
- "Did not search Westlaw" → อาจมีอำนาจอยู่ที่ skill ไม่ค้น

ถ้า skill รวมทั้ง 3 เป็น "no results" — ทนายจะตัดสินใจผิด

### ตัวอย่าง

```markdown
## Search results

| Source | Status |
|--------|--------|
| CourtListener | Searched — 0 results |
| Westlaw (via CoCounsel) | Could not search — auth expired |
| Trellis | Did not search — not in scope for this skill |
```

## 5. Side Awareness (Sales vs Purchasing)

### หลักการ

Contract review skill **ต้องรู้ว่าใครเป็นลูกค้า** — sales-side หรือ purchasing-side — เพราะ playbook กลับด้านกัน

### ตัวอย่างประเด็น

| Clause | Sales-side posture | Purchasing-side posture |
|--------|---------------------|---------------------------|
| Indemnification cap | คงสูงสุด, exclusion เยอะ | บีบ uncapped, exclusion น้อย |
| Limitation of liability | จำกัด direct damages เท่านั้น | open up consequential/lost profits |
| Auto-renewal | ใส่ + cancel-by 90 วัน | ตัดออก หรือ cancel-by 30 วัน |
| Termination for convenience | ไม่ใส่ | ใส่ — 30 วัน notice |
| IP assignment | จำกัด work product เฉพาะ deliverable | broad assignment ของทุก IP ที่ใช้ |
| Audit right | ปฏิเสธ หรือ จำกัด เวลา | กว้าง — annual + on-demand |

### Implementation

Cold-start interview **บังคับให้กรอก side** ก่อนใช้ skill ใด ๆ:

```markdown
## Practice profile (commercial-legal/CLAUDE.md)

side: [sales | purchasing | both]

If "both" — skill จะถามทุกครั้งว่าเอกสารนี้เป็น side ไหน
```

Skill `review` อ่าน `side:` แล้ว load playbook ที่ตรงกัน

## 6. Jurisdiction Awareness

### หลักการ

หลาย skill ต้องรู้ jurisdiction — เพราะ rule ต่างกันสาระสำคัญ

### ตัวอย่าง

| Skill | Jurisdiction matters because |
|-------|------------------------------|
| `employment-legal:termination-review` | At-will state vs just-cause, WARN Act threshold |
| `employment-legal:worker-classification` | ABC test (CA/MA/NJ) vs 20-factor (IRS) vs economic-reality (FLSA) |
| `privacy-legal:dsar-response` | CCPA 45 วัน vs GDPR 30 วัน vs Virginia CDPA |
| `ip-legal:cease-desist` | TM law (federal) vs unfair competition (state) |
| `litigation-legal:demand-draft` | FRE 408 (federal) vs state evidentiary rules |
| `legal-clinic:deadlines` | State-specific SOL |

### Pattern

ทุก skill ที่ jurisdiction-aware ต้อง:

1. Load `jurisdictions:` จาก practice profile
2. ถ้า matter cross-jurisdiction → ถาม primary
3. Apply controlling rule ของ jurisdiction นั้น
4. หาก rule conflict → flag for escalation

### ตัวอย่างใน skill

```markdown
## Step 2: Identify controlling jurisdiction

Sources (in order):
1. matter.md > jurisdiction field
2. User explicit input
3. Practice profile > primary_jurisdiction
4. If none → STOP, ask user

Apply controlling test:
- California → ABC test (Lab. Code §2750.3)
- Massachusetts → ABC test (G.L. c. 149 §148B)
- ...
- If not in list → STOP, escalate
```

## 7. หลักเสริมที่ใช้ตลอด codebase

### Pre-flight citation banner

ทุก skill output เริ่มด้วย banner ว่า citation ตรวจสอบหรือยัง:

```markdown
> ⚠️ Citations from model knowledge only — sources not verified.
> Connect a research connector (CourtListener, Westlaw, Descrybe) for verified cites.
```

### Work-product header (privilege)

ทุก output ของ skill ที่เกี่ยวกับ matter จะมี:

```markdown
[ATTORNEY WORK PRODUCT — PRIVILEGED & CONFIDENTIAL]
[PREPARED IN ANTICIPATION OF LITIGATION]
[NOT LEGAL ADVICE — DRAFT FOR ATTORNEY REVIEW]
```

### Decline pathway = scaffold, not escape hatch

```markdown
## Decline conditions (DO NOT compute)

If ANY of:
- Multiple jurisdictions with conflicting tests
- Engagement < 30 days (insufficient data)
- Worker is a corporate officer (separate analysis)

THEN: STOP. Output "Cannot classify; escalate to [contact]."
```

### Gate header = default-on

```markdown
## Gate: DO NOT send

This skill outputs draft only — never auto-sends.

Exception (narrow):
- User explicitly says "send via Slack"
- AND user has confirmed final draft in this session
- THEN render send instructions (still requires user-initiated action)
```

## 8. หลักการระดับ Repo

### Apache 2.0 License

ทุกอย่างเปิด — fork, edit, redistribute ได้ตาม Apache 2.0

### CLA requirement

Contributor ลงนาม CLA ครั้งแรกที่ PR ก่อน merge

### No build step

Markdown + JSON เท่านั้น — ไม่มี compilation, ไม่มี bundling, ไม่มี black box

### Version bump policy

| ประเภท change | Bump |
|--------------|------|
| Bug fix, behavior tweak | Patch (1.0.x → 1.0.x+1) |
| New skill, new required input | Minor (1.x.0 → 1.x+1.0) |
| Breaking change (plugin.json schema, skill signature) | Major (x.0.0 → x+1.0.0) |

## หลักรวบยอด

> 1. **Doctrine in skill, net in CLAUDE.md** — ไม่อาศัย rescue
> 2. **Citation tag on number, not paragraph** — provenance ติดทน
> 3. **Default-free** — load practice profile หรือถาม
> 4. **Three states of not found** — searched / could-not / did-not
> 5. **Side awareness** — sales vs purchasing คนละ playbook
> 6. **Jurisdiction awareness** — controlling test ต่างกัน
> 7. **Decline pathway = scaffold** — non-overridable
> 8. **Gate default-on** — narrow exception, not load-bearing parenthetical

ทุก skill ใหม่ ทุก plugin ใหม่ ทุก cookbook ใหม่ ควรผ่านการตรวจ 8 ข้อนี้

## สรุปหมวด 09

หมวดนี้ครอบคลุม:
- [adding-skills](adding-skills.html) — วิธีเพิ่ม skill ใหม่
- [validation](validation.html) — `validate.py`, `lint-tool-scope.py`, `test-cookbooks.sh`
- [security-model](security-model.html) — 5 layers ของ defense
- [design-principles](design-principles.html) — หลักการที่ใช้ตลอด codebase

จบ part development — Claude for Legal เปิดให้ทุกคน contribute ได้ ขอแค่ระเบียบวินัยทั้ง 8 ข้อนี้ยังอยู่ ระบบจะรักษาคุณภาพได้แม้จะมีคนแก้ไขเป็นพัน
