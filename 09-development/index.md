---
title: 09 Development
nav_order: 10
has_children: true
permalink: /09-development/
---

# 09 Development — พัฒนา Skill และ Plugin

> "Everything here is markdown and JSON. Fork, edit, PR."
> — `README.md`, Claude for Legal

หมวดนี้สำหรับ **นักพัฒนาที่ต้องการสร้างหรือแก้ไข skill / plugin / cookbook ของ Claude for Legal** ไม่ว่าจะเป็น in-house team ที่อยาก fork plugin มาปรับให้เข้ากับสำนักงาน, vendor ที่ทำ MCP connector ใหม่, หรือชุมชน open source ที่จะส่ง PR กลับเข้า upstream

จุดเด่นของ codebase นี้ — **ไม่มี build step** ทุกอย่างเป็น markdown + JSON เปลี่ยนแก้แล้ว reload ได้ทันที แต่ความง่ายนั้นมาพร้อมกับ **ระเบียบวินัยทางการออกแบบ** ที่ Anthropic ใช้ทำให้ระบบรักษาคุณภาพได้ — หมวดนี้จะอธิบายระเบียบเหล่านั้น

## ไฟล์ในหมวดนี้

| ลำดับ | ไฟล์ | สิ่งที่อธิบาย |
|---|---|---|
| 1 | [adding-skills](adding-skills.html) | วิธีเพิ่ม skill ใหม่ — `SKILL.md` format, naming convention, folder structure, ตัวอย่าง skill ที่ดี vs ไม่ดี |
| 2 | [validation](validation.html) | `validate.py` และ `test-cookbooks.sh` — วิธีใช้, JSON schema, debug |
| 3 | [security-model](security-model.html) | three-layer quality, closed-schema handoff, 3-tier readers/analyzers/writers, `lint-tool-scope.py`, เหตุที่ไม่ใช้ unit test |
| 4 | [design-principles](design-principles.html) | หลัก 6 ข้อจาก `CONTRIBUTING.md` — guardrail-firing rule, citation discipline, default-free design, three states of "not found", side awareness, jurisdiction awareness |

## ปรัชญาเบื้องหลังการพัฒนา

จาก `CONTRIBUTING.md`:

> *"Keep this short — the design principles that matter most for the quality of the output, not a style guide."*

Anthropic เลือก **ปรัชญาก่อน style guide** — สำคัญที่ skill ทำหน้าที่ของมันได้ถูกต้อง ไม่ใช่ formatting ที่สวยงาม หลักสำคัญข้อเดียวที่เปลี่ยนคุณภาพได้คือ:

> *"SKILL.md encodes the right behavior; CLAUDE.md guardrails are the net."*

แปลว่า:

| Layer | หน้าที่ |
|-------|---------|
| **SKILL.md** | ตัว skill เอง — บอกโมเดลตรง ๆ ว่าทำอะไร ทีละขั้น |
| **CLAUDE.md** | guardrail ระดับ plugin — safety net ที่จับ error ที่ skill พลาด |

หากต้องอาศัย guardrail จับ error ที่ skill ควรจัดการเอง = **design smell** ของ skill นั้น

> **กฎ thumb**: ถ้า QA test ผ่านเพราะ guardrail fire — เพิ่ม behavior นั้นเข้า SKILL.md โดยตรง guardrail ยังอยู่ (belt and suspenders) แต่ skill ต้องเข้าใจเองได้

## ตัวอย่างที่ Anthropic ยกใน CONTRIBUTING.md

1. **Design patent question** ไม่ควรผ่าน infringement triage **เพราะ** "Scaffolding, not blinders" override workflow — แต่ **เพราะ** skill รู้จัก D-prefix และ route ไปที่ ordinary-observer test เอง

2. **Renewal cancel-by date** ที่ตรงวันอาทิตย์ ไม่ควรอยู่บน calendar **เพราะ** user คิดถามเรื่อง weekday — แต่ register schema และ Mode 2 output ต้อง roll-back ให้เป็น business day **เอง**

3. **FLSA back-pay** ไม่ควรคำนวณ regular rate ได้ถูกต้อง **เพราะ** model จำ §207(e) ได้ — แต่ skill ต้องมี §207(e) checklist ที่ force การคำนวณ inclusion, 0.5× vs 1.5× posture, liquidated-damages doubling, SOL lookback ลงไปทุกครั้ง

## ทำไมไม่มี unit test

นี่เป็นความตั้งใจของ Anthropic ที่บางทีน่าแปลกใจสำหรับคนที่มา background นักพัฒนา software:

| ปกติใน software | Claude for Legal |
|----------------|-------------------|
| มี unit test, integration test, CI | ไม่มี unit test, มีแค่ `validate.py` ตรวจ structural invariant |
| Test = สิ่งที่กำหนด correctness | Test ทำไม่ได้กับงานที่ output เป็น judgment ของทนายความ |
| Coverage = สำคัญ | Doctrine in skill = สำคัญ |
| Refactor ไม่ break test = ดี | Skill ทำงานบนโมเดลที่อ่อนลง, prompt ที่ terse ลง, editor ที่อ่านแค่ skill text = ต้องดี |

แทนที่ — Anthropic ใช้:

1. **Structural validation** (`validate.py`) — ตรวจ frontmatter, naming convention, file structure
2. **Tool scope linting** (`lint-tool-scope.py`) — กัน privilege escalation ใน Managed Agents
3. **Skill QA framework** (`/legal-builder-hub:skills-qa`) — 9 design parameters
4. **Three-layer quality** — SKILL.md → CLAUDE.md → matter workspace
5. **Closed-schema handoff** — output ของ agent หนึ่งเป็น input ของอีก agent ผ่าน schema เท่านั้น

ดูรายละเอียดที่ [security-model](security-model.html)

## เริ่มต้นการ contribute

จาก `CONTRIBUTING.md`:

> *"Sign the CLA. The first time you open a pull request, the CLA Assistant bot will comment with a link to the CLA and ask you to confirm."*

1. ลงนาม CLA ครั้งแรกที่เปิด PR
2. อ่าน `<plugin>/CLAUDE.md` ก่อนแก้ skill ใด ๆ ใน plugin นั้น
3. รัน `python scripts/validate.py` ก่อน push
4. รัน `python scripts/lint-tool-scope.py` หากแก้ cookbook
5. รัน `bash scripts/test-cookbooks.sh` หากแก้ managed-agent cookbook
6. Bump version ใน `plugin.json` — patch สำหรับ behavior, minor สำหรับ skill ใหม่

## ลำดับการอ่านที่แนะนำ

- **อยากเพิ่ม skill เข้า plugin ที่มี** → [adding-skills](adding-skills.html) → [validation](validation.html)
- **อยากเข้าใจปรัชญา** → [design-principles](design-principles.html) → [security-model](security-model.html)
- **อยากสร้าง plugin ใหม่ทั้งตัว** → [adding-skills](adding-skills.html) → [security-model](security-model.html) → [design-principles](design-principles.html) → [validation](validation.html)
- **อยากสร้าง cookbook headless** → [security-model](security-model.html) (3-tier) → [validation](validation.html) (test-cookbooks.sh)

## คำเตือนเรื่อง responsibility

ทุก skill ที่เขียนเพิ่ม → กลายเป็นเครื่องมือทนายความ — output มีผลต่อ:

- Client legal exposure
- Privilege protection
- Filing deadline
- Settlement strategy
- Contract enforceability

ก่อนส่ง PR — ถามตัวเองว่า skill นี้ผ่าน **9 design parameter** ของ `/legal-builder-hub:skills-qa` ไหม:

1. Clarity of purpose
2. Scope appropriateness
3. Safety & guardrails
4. Citation discipline
5. Destination awareness
6. Privilege handling
7. MCP scope (อะไรที่ skill touch ได้)
8. File-write targets
9. Injection resistance

หากไม่ผ่าน — แก้ก่อน push
