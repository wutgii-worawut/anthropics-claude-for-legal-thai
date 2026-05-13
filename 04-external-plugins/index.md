---
title: 04 External Plugins
nav_order: 5
has_children: true
permalink: /04-external-plugins/
---

# 04 External Plugins — Plugin จากผู้ให้บริการภายนอก

> "The connector launches with Westlaw Deep Research and will expand to additional CoCounsel Legal capabilities over time."
> — `external_plugins/cocounsel-legal/README.md`

หมวดนี้คือคำตอบของคำถาม *"ทำไม Anthropic ถึงเปิดโฟลเดอร์พิเศษให้บริษัทอื่นเอา plugin มาวางในที่เดียวกับ plugin ของตัวเอง?"*

ในหมวด `03-plugins` เราเห็น 12 plugin ที่ **Anthropic เขียนเอง** — `commercial-legal`, `litigation-legal`, `corporate-legal` และอื่น ๆ ซึ่งใช้ skill, agent และ workflow ที่ Anthropic ออกแบบเอง โดยไม่ผูกกับผู้ให้บริการรายใดรายหนึ่ง (vendor-agnostic) เป็นค่าตั้งต้น

แต่ในงานทางกฎหมายจริง — ทีมกฎหมายเกือบทั้งหมดไม่ได้ทำงานบน "ตัวบทกฎหมายอย่างเดียว" พวกเขามี **ฐานข้อมูลเฉพาะทาง (specialized databases)** ที่จ่ายเงินแพง ๆ ในการเข้าถึง: Westlaw, LexisNexis, Bloomberg Law, Practical Law และอื่น ๆ ฐานข้อมูลเหล่านี้คือ "แหล่งความจริง (sources of authority)" ของวงการกฎหมายอเมริกัน

## External plugin คืออะไร

External plugin คือ **plugin ที่บริษัทภายนอกพัฒนาและรับผิดชอบเอง** แต่ Anthropic จัดให้มีพื้นที่ใน repo เดียวกัน เพื่อให้ผู้ใช้ติดตั้งได้สะดวกในขั้นตอนเดียวกับ plugin หลัก

โครงสร้างของ external plugin เหมือนกับ plugin ปกติทุกประการ:

```text
external_plugins/<vendor-name>/
├── .claude-plugin/
│   └── plugin.json          # ทะเบียน plugin (ชื่อ, version, author)
├── .mcp.json                # ทะเบียน MCP server (URL + OAuth)
├── skills/
│   └── <skill-name>/
│       └── SKILL.md         # คำสั่งให้ Claude
└── README.md                # คู่มือผู้ใช้
```

ความต่างอยู่ที่ **author** ใน `plugin.json` — ไม่ใช่ Anthropic แต่เป็นบริษัทเจ้าของฐานข้อมูล และ **MCP server URL** ใน `.mcp.json` ที่ชี้ไปยัง endpoint ของบริษัทนั้น พร้อม OAuth client ID สำหรับยืนยันสิทธิ์ผู้ใช้ที่จ่ายค่าบริการ

## ปรัชญาเบื้องหลัง — แยกหลักการออกจากแหล่งข้อมูล

ทำไม Anthropic ออกแบบให้เป็นแบบนี้?

- **Plugin หลัก** (`litigation-legal`, `corporate-legal` ฯลฯ) ให้ workflow ที่ใช้งานได้กับฐานข้อมูลใดก็ได้ — ทีมที่ไม่ได้สมัครสมาชิก premium ใช้ได้ ทีมที่สมัครก็ใช้ได้
- **External plugin** เพิ่มความสามารถเฉพาะของแต่ละผู้ให้บริการ (vendor-specific) — เช่น Westlaw Deep Research ที่ค้นได้ลึกกว่า web search ทั่วไป เพราะมีฐานข้อมูล caselaw ครบทุกศาล

ผู้ใช้ที่มีสิทธิ์ (subscription) สามารถ "เสียบ" external plugin เข้าไปได้โดย plugin หลักไม่ต้องถูกแก้ — เป็น pattern ที่เรียกว่า **separation of concerns** ระหว่าง **business workflow** กับ **authority source**

## รายการ External Plugin ปัจจุบัน

ในเวอร์ชันปัจจุบันของ repo มี external plugin หนึ่งตัว:

| Plugin | ผู้ให้บริการ | ความสามารถ | ต้องสมัครสมาชิก |
|---|---|---|---|
| [CoCounsel Legal](cocounsel-legal.html) | Thomson Reuters | Westlaw Deep Research — ค้น caselaw, statute, Practical Law พร้อม citation | ต้องมี CoCounsel Legal subscription |

จำนวนนี้น่าจะเพิ่มขึ้นในอนาคต — Anthropic ระบุไว้ใน README ของ CoCounsel เองว่า *"will expand to additional CoCounsel Legal capabilities over time"* ซึ่งเป็นการส่งสัญญาณว่า external_plugins/ จะเป็นพื้นที่เปิดสำหรับผู้ให้บริการรายอื่นในอนาคตด้วย

## ใครควรอ่านหมวดนี้

- **Lawyer ในสำนักงานที่มี Westlaw subscription** อยู่แล้ว — อยากใช้ความสามารถ Deep Research ผ่าน Claude
- **IT / DevOps** ที่ต้องตั้งค่า OAuth กับ Thomson Reuters
- **Plugin developer** ที่อยากเข้าใจ pattern ของ external plugin เพื่อสร้าง plugin ของตัวเอง (เช่นถ้าบริษัทคุณเป็น vendor ฐานข้อมูลกฎหมาย)
