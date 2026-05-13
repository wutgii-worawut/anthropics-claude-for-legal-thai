---
title: หลักการออกแบบ
parent: 01 ภาพรวม
nav_order: 4
---

# หลักการออกแบบ

> "These plugins make that review faster; they do not replace it."
> — README, anthropics/claude-for-legal

หน้านี้สรุป **หลักการออกแบบ (design principles)** ที่ Anthropic ฝังไว้ในระบบ Claude for Legal เมื่ออ่านจบ ผู้อ่านจะเห็นได้ว่าเหตุใด architecture จึงเป็นเช่นนี้ — ไม่ใช่บังเอิญ แต่เป็นผลจากการ trade-off ที่ตั้งใจ

หลักการมี **6 ข้อหลัก** ตามที่ปรากฏและสกัดได้จาก README, CONTRIBUTING, และ source code ของ plugin

## หลักที่ 1 — Single Source, Two Runtimes

> "Everything here is available **two ways from one source**: install it as a Claude Cowork or Claude Code plugin, or deploy it through the Claude Managed Agents API behind your own workflow engine. Same system prompt, same skills — you choose where it runs."
> — README

### ความหมาย

**source เดียว** (ไฟล์ markdown ใน repository) ใช้ได้ทั้งใน **interactive runtime** (Cowork, Claude Code) และ **headless runtime** (Managed Agents API)

### ทำไม

| ทางเลือกที่ Anthropic เลี่ยง | ปัญหา |
|------------------------------|------|
| เขียน skill 2 versions (interactive + scheduled) | drift — version ต่างกันเรื่อย ๆ |
| เขียนแค่ interactive แล้ว wrap ใน cron | ไม่มี security boundary, ไม่มี typed handoff |
| เขียนแค่ headless แล้ว expose เป็น chat | UX แย่, ไม่สามารถ converge กับผู้ใช้ได้ |

### ผลที่ตามมาในโค้ด

1. ทุก agent ที่รัน headless มี **counterpart** ใน `managed-agent-cookbooks/`
2. `agent.yaml` ใน cookbook อ้างถึง **system prompt เดียวกัน** กับ plugin
   ```yaml
   system_prompt:
     file: ../../commercial-legal/agents/renewal-watcher.md
   ```
3. skill ที่ agent เรียก ก็เป็น **ไฟล์เดียวกัน** กับที่ผู้ใช้เรียกแบบ interactive

### ประโยชน์สำหรับสำนักงาน

- เริ่ม pilot ใน Cowork → คุณภาพดีพอ → ย้าย production ขึ้น Managed Agents ได้ทันที **ไม่ต้องเขียน prompt ใหม่**
- สำนักงานเล็กไม่ต้องมี infra เลย — ใช้ Cowork ตัวเดียวก็พอ
- สำนักงานใหญ่มี orchestrator ของตัวเอง (Temporal, Airflow) → swap `scripts/orchestrate.py` แล้วใช้ได้

## หลักที่ 2 — Three-Layer Quality

> "If a skill's correct output depends on a CLAUDE.md guardrail catching a mistake the SKILL.md would have made, that's a design smell."
> — CONTRIBUTING.md

### ความหมาย

คุณภาพของผลลัพธ์ถูก enforce ผ่าน **3 layer** ที่ทำงานเสริมกัน

```
Layer 1: CLAUDE.md (plugin-wide guardrails)
    ↓ ครอบทุก skill ใน plugin
    
Layer 2: SKILL.md (per-skill instructions)
    ↓ ทำงานเฉพาะของ skill นั้น
    
Layer 3: Validators (structural checks)
    ↓ ใน scripts/validate.py, scripts/lint-tool-scope.py
```

### Layer 1 — CLAUDE.md (Wide Net)

ทุก plugin มี shared guardrails ใน `CLAUDE.md`

- "Scaffolding, not blinders" — skill เป็นโครง ไม่ใช่ผ้าคลุมตา
- Source-tag discipline — ทุก citation ต้องมี source
- "Verify user-stated legal facts"
- Premise verification
- Destination check
- Cross-skill severity floor
- Pre-flight citation banner

### Layer 2 — SKILL.md (Narrow Scaffold)

แต่ละ skill ต้องเขียน logic ของตัวเองให้สมบูรณ์ — **ไม่ฝากหวังให้ CLAUDE.md guardrail มา rescue**

> "Rule of thumb: if a QA test passes only because a guardrail fired, add the behavior to the SKILL.md directly. The guardrail stays (belt and suspenders), but the skill now carries the knowledge it needs on its own."

### Layer 3 — Validators (Structural Enforcement)

`scripts/validate.py` และ `scripts/lint-tool-scope.py` ตรวจสอบ

- โครงสร้างไฟล์ (frontmatter required field)
- Tool scope ของแต่ละ agent (อย่าให้ leaf-worker มีสิทธิ์มากกว่าที่จำเป็น)
- naming convention

### ตัวอย่างจริงในเอกสาร

ใน CONTRIBUTING.md ระบุ 3 ตัวอย่างของหลักนี้

| ปัญหา | วิธีแก้ที่ "ผิด" | วิธีแก้ที่ "ถูก" |
|------|------------------|----------------|
| Design patent ผ่าน infringement triage เพราะ utility patent workflow ไม่ครอบ | พึ่ง "Scaffolding, not blinders" ใน CLAUDE.md ให้ override workflow | skill branch บน D-prefix เอง routing ไป ordinary-observer test |
| Renewal cancel-by ตกวันอาทิตย์ | หวังว่าผู้ใช้จะคิดถาม | register schema ฝัง business-day roll-back ไว้เอง |
| FLSA back-pay regular-rate ผิดเพราะ model ลืม §207(e) | หวัง CLAUDE.md guardrail catch | SKILL.md ฝัง §207(e) checklist ที่บังคับ inclusion, 0.5× vs 1.5× posture, liquidated damages doubling, SOL lookback |

### ใจความหลัก

> "The net stays. The goal is a skill that doesn't need the net, not a plugin without one."

## หลักที่ 3 — Closed-Schema Handoffs

> "The orchestrator never interpolates untrusted text into the steering prompt."
> — ARCHITECTURE.md (สกัดจาก scripts/orchestrate.py)

### ความหมาย

ในระบบ Managed Agents — agent หลักไม่มีสิทธิ์เขียนข้อมูลออก (no write, no send, no file) **ทำได้แค่ emit `handoff_request` event** ที่มี **schema ปิด** (closed schema)

Orchestrator (`scripts/orchestrate.py`) ทำหน้าที่ firewall — validate event แล้วถึงจะรัน action

### Closed-schema คืออะไร

**intent ที่อนุญาตมีเฉพาะรายการนี้** เท่านั้น

| intent | ทำอะไร | ใครเรียก |
|--------|--------|---------|
| `slack_send_message` | ส่งข้อความเข้า Slack channel | renewal-watcher, deal-debrief |
| `launch_review` | trigger skill review launch | launch-watcher |
| `deal_debrief` | post-signature debrief | deal-debrief |
| `playbook_monitor` | propose policy update | playbook-monitor |

intent อื่นใด — รวมถึง intent ที่ดูคล้ายแต่ไม่อยู่ในรายการ — จะถูก **reject** และ log ลง audit

### Validation ที่ orchestrator ทำ

```
1. intent อยู่ใน closed schema?
2. target agent อยู่ใน allowlist?
3. parameter ทุกตัวผ่าน regex pattern?
   - Slack channel: ^[CGD][A-Z0-9]{8,}$
   - file path: ^\./out/[A-Za-z0-9_.-]+\.(md|json)$
   - ticket ID: ^[A-Z]{2,10}-[0-9]{1,7}$
4. ไม่มี instruction-like string ใน text parameter (denylist)?
5. ถ้าผ่าน → render steering template ด้วย parameter ที่ validate แล้ว
6. ถ้าไม่ผ่าน → log ลง handoff-audit.jsonl
```

### ทำไม

> "The orchestrator is the firewall between untrusted document readers and consequential actions."

agent ที่อ่านเอกสาร (เช่น contract ใน Ironclad, dossier ใน VDR) **ไม่ควรเชื่อ** เนื้อหาในเอกสาร — เพราะอาจมี prompt injection ฝังไว้

หาก agent มีสิทธิ์ส่ง Slack หรือ trigger action โดยตรง — attacker สามารถ inject คำสั่งใน contract แล้วทำให้ agent ทำสิ่งที่ไม่พึงประสงค์ (เช่น ส่งข้อมูลออกไปยัง channel ที่ไม่ถูกต้อง)

closed-schema handoff ตัดทาง attack นี้ขาด — agent ส่งได้แค่ "intent" ที่ predefined ส่วน text ที่อยู่ใน parameter ถูก **validate** และ **interpolate ใน template** ไม่ใช่ "ใช้เป็น prompt"

### ผลที่ตามมา

- Audit ตามได้ทุก action — ไฟล์ `./out/handoff-audit.jsonl`
- Test ได้ง่าย — orchestrator มี closed input space
- ปลอดภัยจาก prompt injection ระดับ tool-use

## หลักที่ 4 — Default-Free Design / Cold-Start Interview

> "Run the cold-start interview first. Every other skill in a plugin reads from the practice profile it writes. Skipping setup is the single most common reason a skill produces generic output."
> — README

### ความหมาย

ระบบนี้ **ไม่มี "sensible default"** ที่ผู้ใช้ใช้ได้ทันที — เพราะ "sensible default" ในงานกฎหมายมัก ผิดสำหรับกรณีจริง

แทนที่ default → ระบบบังคับให้ผู้ใช้ทำ **cold-start interview** ก่อนใช้งาน skill อื่นใด

### Cold-start interview ทำอะไร

1. ขอ **seed document** จากผู้ใช้ (5–10 ฉบับ MSAs, playbook PDFs, prior memo)
2. อ่านเอกสาร → สกัด position จริงของสำนักงาน
3. ถามคำถาม fill in the gap (jurisdiction, escalation threshold, house style)
4. เขียน practice profile ลง `~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md`
5. เสนอ next step (test run, bulk-load, integration check)

ใช้เวลา 10–20 นาที (quick-start mode: 2 นาที + refine ภายหลัง)

### ทำไมหลักนี้ต่างจากระบบอื่น

| ระบบทั่วไป | Claude for Legal |
|-----------|-----------------|
| มี default ที่ออกแบบมาดี ใช้ได้ทันที | **ไม่มี default** — ผู้ใช้ต้องทำ profile เอง |
| Config เป็น YAML ที่ผู้ใช้แก้ | profile เป็น **plain English markdown** |
| Setup wizard 1-time ตอนติดตั้ง | Setup เป็น **skill ตัวหนึ่ง** รัน reset ได้ตลอดเวลา |

### ตัวอย่าง — Sales-side playbook ของ commercial-legal

หลัง cold-start, CLAUDE.md ของ user จะมีโครงประมาณ

```markdown
## Company
- Industry: SaaS, B2B
- Jurisdictions: Delaware, California

## Active side: sales (we are the vendor)

## Sales-side playbook
- **Liability cap**: 12 months fees, no carve-outs for indirect/consequential
- **Indemnity**: mutual; IP from us, data from them
- **Termination**: 30 days notice for convenience; we do not accept termination for inconvenience
- **Auto-renewal**: 12 months auto with 60 days cancel-by; we will accept 30 days if customer pushes

## Escalation Matrix
| Issue | Threshold | Approver |
|-------|-----------|----------|
| Uncapped liability | any request | GC |
| IP assignment to customer | any | CTO + GC |
| MFN clause | any | CFO |

## House style
- Redline: tracked changes in .docx
- Memo header: "[AI-ASSISTED DRAFT — requires review]"
- Slack alerts to: #legal-vendor-alerts
```

ทุก skill (review, escalation-flagger, stakeholder-summary) อ่านไฟล์นี้ก่อนทำงาน — **personalization เกิดที่นี่ทั้งหมด ไม่ใช่ในตัว skill**

## หลักที่ 5 — Citation Discipline

> "Citations are the *mechanism* by which users discover what needs review. Verified citations are tagged; unverified ones are flagged."
> — ARCHITECTURE.md

### ความหมาย

ทุก citation ที่ออกจากระบบ ต้องมี **source tag** บอกว่ามาจากไหน

- จาก **research MCP** (CourtListener, Westlaw, Trellis, Descrybe) → tag ด้วยชื่อ source
- จาก **training data ของ model** → tag `[verify]`
- ไม่มี research tool ติดตั้ง → **reviewer header ระบุ** ว่า "sources weren't verified"

### GREEN / YELLOW / RED Triage

หลายๆ skill ใช้ traffic-light triage สำหรับการตัดสินใจ

| สี | ความหมาย | ตัวอย่าง |
|----|---------|---------|
| 🟢 GREEN | ผ่าน playbook, มาตรฐาน | NDA แบบ standard, ไม่ deviate |
| 🟡 YELLOW | มี deviation แต่ยอมรับได้ในกรอบ | liability cap ลดเหลือ 6 เดือน, แต่ counterparty มี leverage |
| 🔴 RED | เกินเกณฑ์ ต้อง escalate | uncapped liability, IP assignment ที่ไม่ควรให้ |

### Verbatim Quote Discipline

> "Attach provenance tags to numbers, not to paragraphs. `[model calculation — verify against the notice clause]` next to the date; `[verify — consult wage-and-hour counsel before asserting or paying]` on the line the back-pay number appears. Tags on surrounding prose get lost; tags on the load-bearing digit do not."
> — CONTRIBUTING.md

หลักการนี้บอกว่า — tag ต้องอยู่ติดกับ **ตัวเลขที่ load-bearing** (จำนวนเงิน, วันที่, threshold) ไม่ใช่อยู่ใน paragraph ที่อ่าน skim ๆ จะหลุดได้

### ผลที่ตามมา

- ผู้ใช้ scan output เห็น `[verify]` แล้วรู้ว่าต้อง double-check
- หาก research MCP ไม่ทำงาน — header บอก, output ไม่มี false certainty
- ทนายที่ tracking ใช้เวลาที่จุดต้องตรวจ ไม่ใช่ทุกประโยค

## หลักที่ 6 — ABA Op. 512 + Privilege-Conscious

> "Built within ABA Formal Op. 512."
> — README, legal-clinic plugin

### ABA Formal Opinion 512 คืออะไร

American Bar Association ออก Formal Opinion 512 ในปี 2024 ว่าด้วย **การใช้ generative AI โดยทนายความ** หลักการสำคัญ

1. **Competence (Rule 1.1)** — ทนายต้องเข้าใจเครื่องมือ AI ที่ใช้ ทั้งความสามารถและข้อจำกัด
2. **Confidentiality (Rule 1.6)** — ข้อมูลลูกค้าต้องไม่หลุดผ่าน prompt หรือ training
3. **Communication (Rule 1.4)** — แจ้งลูกค้าหาก AI ใช้ในงาน (ขึ้นกับ jurisdiction)
4. **Supervision (Rules 5.1, 5.3)** — supervisor responsible หาก AI ทำพลาด
5. **Fees (Rule 1.5)** — ห้ามคิดเวลา AI เป็นเวลาทนาย
6. **Candor (Rule 3.3)** — verify AI output ก่อน file ในศาล

### ระบบ Claude for Legal embed หลักการเหล่านี้อย่างไร

#### Competence

- Cold-start interview บังคับ — ทนายต้องทำความเข้าใจ playbook ก่อน
- เอกสารทุกชิ้นกำกับ `[AI-ASSISTED DRAFT — requires review]`

#### Confidentiality

- **Matter workspace** ใน plugin ส่วนใหญ่ — แยก context ของ matter แต่ละราย
- ข้อมูลไม่ leak cross-matter
- MCP server แบบ on-premise (iManage, Ironclad ของบริษัท) — ไม่ไป third party
- Privilege header `[Attorney Work Product / Privileged]` ใน output

#### Communication

- Skill บางตัวมี gate ก่อนส่งข้อความออกถึงลูกค้า
- `legal-clinic:client-letter` แยก template "AI-assisted" และ "AI-not-disclosed" ตาม jurisdiction

#### Supervision

- `legal-clinic:supervisor-review-queue` — รวบรวมงาน student รอ professor approve
- ทุก output มี reviewer note ระบุจุดต้องตรวจ

#### Fees

- skill ไม่มี billing logic — เพื่อไม่ blur ระหว่าง AI time กับ attorney time

#### Candor

- Citation tag — verified vs unverified
- Gate ก่อน file/send

### Privilege-conscious — ตัวอย่างเฉพาะ

ใน `litigation-legal` plugin

- **Privilege log review** — first-pass แต่ flag ที่ต้อง attorney call
- **Internal investigation** memo มี privilege header default
- **Demand letter** มี FRE 408 (settlement negotiation privilege) gate ก่อนส่ง — กันไม่ให้ใช้ admission ผิดบริบท

### ใจความหลัก

> Claude for Legal ไม่ใช่ "lawyer replacement" — เป็น "lawyer assistance" ที่ **ออกแบบให้ผ่าน duty of competence + confidentiality + supervision** โดยกลไก architecture ไม่ใช่แค่ disclaimer

## สรุปทั้ง 6 หลัก

| หลัก | ความหมาย | กลไกในโค้ด |
|------|---------|-----------|
| 1. Single source, two runtimes | source เดียวใช้ได้ทั้ง Cowork/Code และ Managed Agents | `agent.yaml` อ้าง system prompt จาก plugin |
| 2. Three-layer quality | CLAUDE.md → SKILL.md → validators | guardrail ใน CLAUDE.md เป็น "net" ไม่ใช่ "primary logic" |
| 3. Closed-schema handoffs | orchestrator validate intent ก่อนทำ | `scripts/orchestrate.py` + audit log |
| 4. Default-free / cold-start | บังคับ interview ก่อนใช้ skill | `cold-start-interview` ใน plugin ทุกตัว |
| 5. Citation discipline | tag source, GREEN/YELLOW/RED triage | tag ติดที่ตัวเลข, reviewer header |
| 6. ABA Op. 512 + privilege | competence, confidentiality, supervision | matter workspace, privilege header, gate |

## เอกสารถัดไป

- [Quickstart — เริ่มต้นใช้งาน](./quickstart.html) — ลองรันจริงให้เห็นหลักการเหล่านี้ทำงาน

← กลับไป [01 ภาพรวม](./)
