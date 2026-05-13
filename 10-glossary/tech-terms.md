---
title: ศัพท์ Technical
parent: 10 อภิธานศัพท์
nav_order: 2
---

# ศัพท์ Technical

> รวบรวมคำศัพท์เฉพาะของระบบ Claude/Anthropic และคำที่ใช้ใน `claude-for-legal` ซึ่งหลายคำ **ไม่ใช่ศัพท์มาตรฐานของวงการ software ทั่วไป** แต่เป็นภาษากลางที่ Anthropic นิยามขึ้น

## หมวด 1 — หน่วยพื้นฐานของระบบ (Core building blocks)

| คำศัพท์ | ความหมาย | บริบทใน claude-for-legal |
|---------|---------|------------------------|
| Plugin | ชุด skills/agents/hooks/MCP ที่แพ็คกันเป็นหน่วยเดียว — ติดตั้งและถอนได้พร้อมกัน | 12 plugins ใน root ของ repo เช่น `commercial-legal/`, `privacy-legal/`, `ip-legal/` |
| Skill | คำสั่งเดี่ยว ๆ ที่ผู้ใช้เรียกใช้ผ่าน slash command — หนึ่ง skill ทำงานหนึ่งอย่าง | `<plugin>/skills/<name>/SKILL.md` ทั้งหมด 151 skills ใน claude-for-legal |
| Agent | ตัวงานที่ทำงาน autonomous (มี trigger เฉพาะ) — ต่างจาก skill ตรงที่ไม่ต้องรอผู้ใช้สั่ง | scheduled/named workers เช่น `renewal-watcher`, `docket-watcher` |
| Subagent | agent ลูกที่ orchestrator (agent หลัก) เรียกใช้เพื่อทำงานคู่ขนาน | `diligence-grid` pattern ที่ spawn subagents หลายตัวอ่านสัญญาพร้อมกัน |
| Hook | shell command ที่รันอัตโนมัติเมื่อมี event เช่น ก่อน/หลังเรียก tool, ก่อน commit | `<plugin>/hooks/` — มักใช้ตรวจสอบความปลอดภัยหรือ format output |
| Managed Agent | agent ที่ deploy แบบ headless ผ่าน Anthropic API — รันบน server ของ Anthropic ไม่ใช่ in client | ใช้ใน `managed-agent-cookbooks/` สำหรับ background worker |

## หมวด 2 — โปรโตคอลและการเชื่อมระบบ (Protocols & Integrations)

| คำศัพท์ | ความหมาย | บริบทใน claude-for-legal |
|---------|---------|------------------------|
| MCP (Model Context Protocol) | มาตรฐาน open ที่ Anthropic ออกแบบสำหรับเชื่อม Claude กับเครื่องมือภายนอก — ทำให้ tool integration เป็น modular | ใช้เชื่อม Ironclad, DocuSign, Westlaw, Lexis, Slack ฯลฯ |
| MCP Server | server กระบวนการที่ implement MCP protocol — ทำหน้าที่เป็นตัวกลางระหว่าง Claude กับระบบ third-party | ตัวอย่าง `mcp-ironclad`, `mcp-westlaw` |
| Connector | ชื่อเรียกที่ user-facing ของ MCP server — เน้นว่าเป็นตัวเชื่อมระบบ | Lexis Connector, Westlaw Connector ใน 04-external-plugins |
| Marketplace | registry ที่รวม plugins ไว้ — เป็นไฟล์ JSON ที่บอกว่ามี plugin อะไรบ้าง ติดตั้งจากไหน | `.claude-plugin/marketplace.json` ใน root |
| Tool | ฟังก์ชันที่ Claude เรียกใช้ระหว่างคิด เช่น file read, web search, MCP call | ใน claude-for-legal มี tool หลายชั้น แบ่งตามสิทธิ์ |

## หมวด 3 — โครงสร้างไฟล์และ configuration

| คำศัพท์ | ความหมาย | บริบทใน claude-for-legal |
|---------|---------|------------------------|
| Frontmatter | ส่วน YAML header ที่อยู่ต้นไฟล์ markdown ครอบด้วย `---` | บอก name, description, version, allowed-tools ของ skill |
| Practice Profile | ไฟล์ CLAUDE.md ที่ user กรอกค่าเฉพาะของสำนักงาน — กลายเป็น "context หลัก" ที่ทุก skill อ่านได้ | สร้างจาก `cold-start-interview` skill ของแต่ละ plugin |
| CLAUDE.md | ไฟล์มาตรฐานที่ Claude อ่านอัตโนมัติเมื่อเริ่ม session — เก็บ context ระดับ project | อยู่ใน root ของ project, ของ plugin, หรือ home dir |
| Cookbook | template สำหรับ deploy managed agent — ครบชุดทั้ง prompt, schedule, error handling | 5 cookbooks ใน `managed-agent-cookbooks/` |
| Slash Command | คำสั่งรูป `/command` ที่ user พิมพ์ใน Claude Code/Cowork — แต่ละ skill มี slash command ของตัวเอง | เช่น `/commercial-legal:nda-review`, `/privacy-legal:dsar-response` |
| Allowed-Tools | field ใน frontmatter ที่ระบุว่า skill นี้ใช้ tool อะไรได้บ้าง — เป็นชั้นจำกัดสิทธิ์ | บังคับโดย Anthropic runtime ที่จะ deny tool ที่ไม่อยู่ในรายการ |

## หมวด 4 — รูปแบบการออกแบบ (Design patterns)

| คำศัพท์ | ความหมาย | บริบทใน claude-for-legal |
|---------|---------|------------------------|
| Closed-Schema Handoff | การส่งงาน inter-agent ผ่าน enumerated intent — agent ปลายทางรับเฉพาะ intent ที่ระบุไว้ ไม่รับ free-form text | ใช้ใน `orchestrate.py` ของ plugin ใหญ่ ๆ ป้องกัน prompt injection ระหว่าง agent |
| Cold-Start Interview | บทสนทนาแบบมีโครงสร้างก่อนเริ่มงานจริง เพื่อกรอก context ใส่ practice profile | บังคับเสมอใน Cowork — plugin จะปฏิเสธทำงานถ้ายังไม่มี CLAUDE.md |
| Default-Free | หลักการที่ skill **ไม่ใช้ค่าเริ่มต้นเงียบ ๆ** — ถ้าไม่รู้ value ต้องถามผู้ใช้ ไม่เดา | ป้องกันการสมมติเขตอำนาจศาลผิด หรือสมมติสถานะลูกความผิด |
| Single Source, Two Runtimes | source markdown ชุดเดียว รันได้ทั้งใน Claude Code (CLI) และ Claude Cowork (UI) | คำขวัญหลักของ claude-for-legal |
| Three-Tier Security | การแบ่ง tool เป็น 3 ระดับ — readers (อ่านอย่างเดียว) / analyzers (วิเคราะห์) / writers (เขียน/ส่ง) | บังคับโดย linter `lint-tool-scope.py` ที่ scan ทุก skill |
| Privilege Escalation | การที่ skill เพิ่มสิทธิ์ของ tool โดยไม่ตั้งใจ เช่น skill ที่ควรเป็น reader แต่ขอ write tool | ป้องกันด้วย linter — fail build ถ้าเจอ |
| Verbatim Quote | การยกข้อความ **คำต่อคำ** จากเอกสารต้นฉบับ ไม่ paraphrase | ทุก finding/output มี verbatim quote ประกอบ เพื่อให้ทนายตรวจสอบกลับได้ |
| Citation Discipline | วินัยในการอ้างอิงแหล่งที่มาของทุกข้อกล่าวอ้าง — ทั้ง section, paragraph, page | บังคับโดยทุก skill ที่ output legal analysis |

## หมวด 5 — ระดับและสถานะ (Levels & States)

| คำศัพท์ | ความหมาย | บริบทใน claude-for-legal |
|---------|---------|------------------------|
| GREEN / YELLOW / RED | ระดับความเสี่ยงของ output — GREEN = ผ่าน, YELLOW = ต้องตรวจ, RED = ห้ามใช้ก่อนแก้ | ใช้ใน `nda-review`, `vendor-dpa-review`, `ip-clearance` ฯลฯ |
| Three States of "Not Found" | (1) ไม่มีในเอกสาร (2) มีแต่ไม่เข้าเงื่อนไข (3) ไม่ได้ตรวจ | structured output discipline — บังคับให้ output ระบุชัด ไม่เงียบ |
| Found / Not Present / Not Reviewed | สามสถานะที่ต้องระบุชัดในทุก finding | เพื่อป้องกัน false negative ที่ทนายอาจเข้าใจผิดว่า "ไม่มีปัญหา" |
| Draft / Final | สถานะของเอกสาร — output ของ plugin **เสมอเป็น draft** ไม่ใช่ final | คำเตือนใต้ทุก output: "for attorney review only" |

## หมวด 6 — ความตระหนัก (Awareness)

| คำศัพท์ | ความหมาย | บริบทใน claude-for-legal |
|---------|---------|------------------------|
| Side Awareness | การที่ skill รู้ว่ากำลังทำงานฝั่งไหน — sales-side vs purchasing-side, buyer vs seller | `commercial-legal` มี playbook แยกฝั่ง ป้องกันการแนะนำผิดข้าง |
| Jurisdiction Awareness | การที่ skill รู้ว่ากำลังทำกฎหมายของรัฐ/ประเทศไหน — ถ้าไม่รู้ ต้องถาม | `employment-legal` (กฎหมายแรงงานต่างกันทุกรัฐ) และ `ip-legal` |
| Privilege-Conscious | การที่ระบบไม่เก็บ/ส่ง/log ข้อมูลที่อาจเป็น attorney-client privileged โดยไม่จำเป็น | บังคับใน ABA Op. 512 — มีผลต่อ telemetry และ logging |
| ABA Op. 512 Compliance | การปฏิบัติตาม ABA Formal Opinion 512 เรื่อง generative AI ของทนาย | guardrail ที่ฝังในทุก plugin — competence, confidentiality, candor, supervision, fees |

## หมวด 7 — Runtime และ deployment

| คำศัพท์ | ความหมาย | บริบทใน claude-for-legal |
|---------|---------|------------------------|
| Claude Code | CLI tool ของ Anthropic สำหรับ developer — รันใน terminal | one ของ two runtimes |
| Claude Cowork | UI ของ Anthropic สำหรับ professional/legal user — รันในเบราว์เซอร์ | runtime หลักที่ทนายใช้ |
| Anthropic API | API ที่ใช้สร้าง managed agent — ไม่ใช่ interactive | runtime ที่สามสำหรับ headless deployment |
| Steering Event | event ที่ส่ง context ใหม่เข้า managed agent ระหว่างที่มันรันอยู่ — เปลี่ยน behavior โดยไม่ restart | ใช้ใน `docket-watcher`, `reg-monitor` เมื่อมีข่าวใหม่เข้ามา |
| Headless | การรันโดยไม่มี user interaction — output ไป log/database ไม่ใช่ terminal | managed agents เป็น headless ทั้งหมด |
| Schedule / Cron | การตั้งเวลาให้ agent ทำงานเป็นรอบ ๆ เช่น ทุกวัน 8 โมงเช้า | กำหนดใน cookbook frontmatter ของ managed agent |

## หมวด 8 — เครื่องมือพัฒนาและ QA

| คำศัพท์ | ความหมาย | บริบทใน claude-for-legal |
|---------|---------|------------------------|
| Linter | เครื่องมือตรวจคุณภาพ code/markdown แบบ static — รันก่อน commit | `lint-tool-scope.py`, `lint-frontmatter.py` ใน `06-scripts/` |
| Pre-commit Hook | git hook ที่รันก่อน commit — fail commit ถ้าตรวจไม่ผ่าน | บังคับว่าทุก skill มี frontmatter ถูกต้อง |
| Smoke Test | การทดสอบเบื้องต้นที่ยืนยันว่า skill รันได้ ไม่ crash | ทุก plugin มี smoke test สั้น ๆ |
| Golden File Test | การทดสอบที่เทียบ output กับไฟล์มาตรฐานที่บันทึกไว้ | ใช้กับ skill ที่ output โครงสร้างคงที่ |
| Eval | การวัดคุณภาพ output ของ LLM ด้วยชุดข้อมูลทดสอบและเกณฑ์คะแนน | ใช้กับ skill ที่สำคัญ เช่น `nda-review` |

## หมวด 9 — คำที่มักสับสน

| คำศัพท์ | ความหมาย | ความต่างที่สับสนบ่อย |
|---------|---------|---------------------|
| Plugin vs Skill | plugin = ชุดรวม, skill = หน่วยเดี่ยว | 1 plugin มีหลาย skills เสมอ |
| Skill vs Agent | skill = ผู้ใช้เรียก, agent = ทำงานเอง | skill มี slash command, agent มี trigger/schedule |
| Agent vs Managed Agent | agent = ทำงาน autonomous ใน runtime ใดก็ได้, managed agent = รันบน Anthropic infra | managed agent ต้อง deploy ผ่าน API |
| MCP vs Tool | MCP = โปรโตคอล, tool = ฟังก์ชันที่ Claude เรียก | MCP เป็นทางหนึ่งในการ expose tool |
| Hook vs Agent | hook = รันเมื่อมี event เฉพาะ, agent = รันต่อเนื่อง/ตามเวลา | hook เป็น reactive, agent เป็น proactive |
| Cowork vs Code | Cowork = UI สำหรับ professional, Code = CLI สำหรับ developer | source markdown ชุดเดียวรันได้ทั้งสอง |

> **ข้อสังเกต**: คำศัพท์เช่น *plugin*, *skill*, *agent*, *MCP* เป็น **คำที่ Anthropic นิยามขึ้นเฉพาะ** — ความหมายอาจไม่ตรงกับการใช้คำเดียวกันในระบบอื่น (เช่น "plugin" ใน WordPress, "agent" ใน LangChain) ผู้อ่านควรยึดนิยามในตารางนี้เมื่ออ่านเอกสารชุด claude-for-legal
