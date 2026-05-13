---
title: 08 Deployment
nav_order: 9
has_children: true
permalink: /08-deployment/
---

# 08 Deployment — ติดตั้งและรัน Claude for Legal

> "Same system prompt, same skills — you choose where it runs."
> — `README.md`, Claude for Legal

หมวดนี้พาท่าน **ติดตั้งและรัน** ระบบ Claude for Legal ในสภาพแวดล้อมจริง โดย Anthropic ออกแบบให้ **plugin ชุดเดียวกัน** สามารถ deploy ได้บน **3 surfaces** ที่ต่างกัน — UI, CLI, และ headless API — ทำให้ทนายความ in-house, firm associate, หรือทีม legal-ops สามารถเลือก surface ที่เหมาะกับ workflow ของตัวเอง โดยไม่ต้องเขียน skill ใหม่

## เปรียบเทียบ 3 surfaces

| มิติ | **Claude Cowork** (UI) | **Claude Code** (CLI) | **Managed Agents** (API) |
|------|------------------------|-----------------------|--------------------------|
| รูปแบบ | Desktop app + browser sidebar | Terminal CLI | HTTPS endpoint (headless) |
| ผู้ใช้หลัก | ทนายความผู้ใช้งานปลายทาง | นักพัฒนา / power user | DevOps / Legal-ops engineer |
| การติดตั้ง | คลิก install จาก marketplace | `/plugin install <name>@claude-for-legal` | `scripts/deploy-managed-agent.sh <slug>` |
| การ trigger | สนทนากับ Claude ใน sidebar | พิมพ์ `/<plugin>:<skill>` ใน terminal | Scheduled (cron) หรือ webhook |
| Practice profile | กรอกผ่าน wizard | `/<plugin>:cold-start-interview` | กำหนดใน `agent.yaml` |
| Connector auth | OAuth pop-up | OAuth ครั้งแรก / API key ใน `.mcp.json` | service account / API key ใน env |
| ผลลัพธ์ | แสดงในแชท, ส่งไป Word/Excel ผ่าน sidebar | เขียนลงไฟล์ในโฟลเดอร์ปัจจุบัน | POST handoff event ไปยัง orchestrator |
| Hands-on review | ทุกครั้ง (interactive) | ทุกครั้ง (interactive) | กึ่งอัตโนมัติ (gate ใน skill) |
| ตัวอย่าง use case | "เปิด Word ขึ้นมา review NDA ในมือ" | "รัน batch review 50 vendor agreements ใน folder" | "Renewal watcher รันทุกเช้าวันจันทร์ ส่งสรุปเข้า Slack" |
| Production readiness | พร้อมใช้งาน | พร้อมใช้งาน | Research Preview (subagent depth-1) |

หลักคิด: **เลือก surface ที่เหมาะกับ "ใครทำอะไรกี่ครั้งต่อสัปดาห์"** ทนายที่ทำ review หลายชิ้นต่อวันใช้ Cowork; ทีมที่ต้องการ batch automation ใช้ Claude Code; งาน scheduled watcher ที่รันโดยไม่มีคนนั่งใช้ Managed Agents

## ไฟล์ในหมวดนี้

| ลำดับ | ไฟล์ | สิ่งที่อธิบาย |
|---|---|---|
| 1 | [claude-cowork](claude-cowork.html) | ติดตั้ง plugin ใน Claude Cowork (UI), กรอก practice profile, cold-start interview |
| 2 | [claude-code](claude-code.html) | ติดตั้ง plugin ใน Claude Code (CLI), `/plugin install`, ความต่างจาก Cowork |
| 3 | [managed-agents](managed-agents.html) | Deploy แบบ headless ผ่าน Managed Agents API, `agent.yaml`, 3-tier security |
| 4 | [mcp-connectors](mcp-connectors.html) | ติดตั้งและ auth กับ MCP connector — Ironclad, Westlaw, Slack, Drive ฯลฯ |

## ลำดับการอ่านที่แนะนำ

- **ทนายความที่จะใช้งานทันที** → [claude-cowork](claude-cowork.html) (60 วินาที)
- **นักพัฒนาที่จะ batch automate** → [claude-code](claude-code.html) → [mcp-connectors](mcp-connectors.html)
- **Legal-ops engineer** → [managed-agents](managed-agents.html) → [mcp-connectors](mcp-connectors.html)
- **อยากเชื่อม Westlaw/Ironclad โดยตรง** → ข้ามไป [mcp-connectors](mcp-connectors.html)

## คำเตือนสำคัญก่อน deploy

จาก `README.md`:

> *"Every output from these plugins is a draft for attorney review — not legal advice, not a legal conclusion, not a substitute for a lawyer."*

ไม่ว่าจะ deploy บน surface ใด หลักการเดียวกันคือ — **มีทนายตรวจสอบทุกครั้ง** ก่อนนำผลผลิตไปใช้จริง guardrail ที่ฝังในระบบ (source attribution, jurisdiction assumption, gate ก่อน action ที่ irreversible) มีหน้าที่ **เร่งการ review** ไม่ใช่ทดแทนการ review
