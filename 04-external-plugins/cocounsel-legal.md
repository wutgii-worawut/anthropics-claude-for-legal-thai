---
title: CoCounsel Legal (Thomson Reuters)
parent: 04 External Plugins
nav_order: 1
---

# CoCounsel Legal — Westlaw Deep Research ใน Claude

> "CoCounsel Legal brings Westlaw Deep Research into Claude for CoCounsel Legal subscribers."
> — `external_plugins/cocounsel-legal/README.md`

## CoCounsel คืออะไร

**CoCounsel** คือผลิตภัณฑ์ AI assistant สำหรับนักกฎหมายที่ **Thomson Reuters** เปิดตัวในปี 2023 (เดิมพัฒนาโดยบริษัท Casetext แล้ว Thomson Reuters เข้าซื้อกิจการในเดือนสิงหาคม 2023 มูลค่าประมาณ 650 ล้านดอลลาร์)

CoCounsel ทำหน้าที่:

1. **การค้นคว้าทางกฎหมาย (legal research)** — ค้นบทกฎหมาย คำพิพากษา และความเห็นทางวิชาการ
2. **การตรวจร่างเอกสาร (document review)** — อ่านสัญญาเป็นจำนวนมากแล้วสกัดประเด็นสำคัญ
3. **การร่างเอกสาร (drafting)** — ร่างบันทึก คำให้การ จดหมายข้อเรียกร้อง
4. **การเตรียมพยาน (deposition prep)** — สรุปประเด็นและสร้างชุดคำถาม

CoCounsel ทั้งหมดรันบนฐานข้อมูล **Westlaw** + **Practical Law** ของ Thomson Reuters

## Thomson Reuters คือใคร

**Thomson Reuters** คือบริษัทข้อมูลและบริการมืออาชีพข้ามชาติ (ตั้งสำนักงานใหญ่ที่โตรอนโต้ แคนาดา) เป็นผู้ผูกขาด **ฐานข้อมูลกฎหมายอเมริกัน** ร่วมกับคู่แข่งหลัก (LexisNexis)

ผลิตภัณฑ์ในเครือที่นักกฎหมายอเมริกันใช้ทุกวัน:

- **Westlaw** — ฐานข้อมูล caselaw, statute, regulation ที่ครอบคลุมที่สุดในสหรัฐฯ (ครอบคลุม federal + state + administrative ทั้งหมด)
- **Practical Law** — บทความวิชาการแบบ "how-to" สำหรับนักกฎหมายฝึกหัด ครอบคลุมหัวข้อต่าง ๆ (เช่น "วิธีร่าง NDA", "ขั้นตอนการยื่นฟ้องในศาล Delaware")
- **JD Supra** — แพลตฟอร์มเผยแพร่บทความวิเคราะห์กฎหมาย (current awareness)

ฐานข้อมูลเหล่านี้คือ "ห้องสมุดกลาง" ของวงการกฎหมายอเมริกัน — สำนักงานกฎหมายเกือบทุกแห่งจ่ายค่าสมาชิก Westlaw หรือ LexisNexis รายเดือนเป็นเงินจำนวนมาก

## ทำไม Anthropic เปิด external_plugins/ ให้ Thomson Reuters

ใน `claude-for-legal` plugin หลัก 12 ตัวที่ Anthropic เขียนเอง ทำงานบนข้อมูล **ที่ผู้ใช้ป้อนเข้าไปเอง** — สัญญาในโฟลเดอร์ matter, เอกสาร PDF, ฯลฯ — โดยไม่เรียกฐานข้อมูลภายนอก

แต่งานบางอย่าง — โดยเฉพาะการ **ค้นคว้า caselaw (legal research)** — ต้องเข้าถึงฐานข้อมูลที่:

1. มีคำพิพากษาครบทุกศาล (ซึ่ง Westlaw มี แต่ web search ไม่มี)
2. มีการ "Shepardize" — ตรวจว่าคำพิพากษาที่อ้างยังใช้ได้อยู่หรือถูก overrule ไปแล้ว
3. มีการเชื่อม cross-reference ระหว่าง caselaw ↔ statute ↔ regulation
4. มีข้อมูลที่ paywall และไม่ปรากฏใน Google

ทางออกที่ Anthropic เลือก: **เปิดพื้นที่ `external_plugins/`** ให้ vendor ที่มีฐานข้อมูลพิเศษเข้ามาวาง plugin ของตัวเอง โดย Anthropic ไม่เข้าไปแก้ — vendor รับผิดชอบ skill, MCP server และการ support เอง

นี่คือ pattern ที่จะเปิดทางให้:

- **Thomson Reuters** เพิ่ม CoCounsel Legal (ตัวที่อยู่ใน repo ตอนนี้)
- **LexisNexis** เพิ่ม Lexis+ AI ในอนาคต (ถ้าตกลงกัน)
- **Bloomberg Law, Fastcase, vLex** เพิ่ม plugin ของตัวเอง

โดยที่ผู้ใช้ปลายทาง **ติดตั้งทั้งหมดผ่าน marketplace เดียวกัน** ไม่ต้องเรียนรู้ระบบ plugin ใหม่ของแต่ละ vendor

## โครงสร้างไฟล์ของ cocounsel-legal/

```text
external_plugins/cocounsel-legal/
├── .claude-plugin/
│   └── plugin.json          # 299 bytes — ทะเบียน plugin
├── .mcp.json                # 259 bytes — ทะเบียน MCP server
├── skills/
│   └── deep-research/
│       └── SKILL.md         # 7.5 KB — คำสั่งให้ Claude
└── README.md                # 3.1 KB — คู่มือผู้ใช้
```

### `.claude-plugin/plugin.json`

```json
{
  "name": "cocounsel-legal",
  "version": "0.1.0",
  "description": "CoCounsel Legal delivers comprehensive Westlaw Deep Research reports with inline, linked citations to Westlaw and Practical Law sources.",
  "author": {
    "name": "Thomson Reuters",
    "email": "cocounselsupport@tr.com"
  }
}
```

สังเกตว่า `author.name` คือ **"Thomson Reuters"** — ไม่ใช่ Anthropic นี่คือสัญลักษณ์ที่บอก marketplace ว่า plugin นี้บริษัทอื่นเป็นผู้รับผิดชอบ

### `.mcp.json`

```json
{
    "mcpServers": {
        "cocounsel-legal": {
            "type": "http",
            "url": "https://legal-mcp.thomsonreuters.com/mcp",
            "oauth": {
                "clientId": "QCgP4IGN5JiLqXRHxiAVr3wu1ySo2nQx"
            }
        }
    }
}
```

ไฟล์นี้บอก Claude ว่า:

- มี MCP server หนึ่งตัวชื่อ `cocounsel-legal`
- ประเภท HTTP — ไม่ใช่ MCP ที่รันใน local แต่เรียกผ่าน HTTPS ไปยังเซิร์ฟเวอร์ของ Thomson Reuters โดยตรง
- URL: `https://legal-mcp.thomsonreuters.com/mcp`
- ต้องยืนยันสิทธิ์ผ่าน **OAuth** ด้วย `clientId` ที่ระบุ — ผู้ใช้จะถูกพาไปหน้า login ของ Thomson Reuters ครั้งแรกที่ใช้

นี่คือ pattern ที่ทำให้ Anthropic **ไม่ต้องเก็บข้อมูลผู้ใช้หรือ credential ใด ๆ** — การยืนยันสิทธิ์เกิดระหว่างผู้ใช้กับ Thomson Reuters โดยตรง

## Skill ที่มี — Westlaw Deep Research

ใน `skills/deep-research/SKILL.md` คือคำสั่งที่บอก Claude ว่าจะใช้ MCP server ของ Thomson Reuters อย่างไร เพื่อทำการค้นคว้าแบบ "deep"

### Deep Research คืออะไร

ปกติการค้นใน Westlaw ใช้ keyword หรือ boolean query แบบ:

```text
"non-compete" /s "executive" & da(aft 2020) & jurisdiction(CA)
```

แต่ Westlaw Deep Research ทำงานในแบบ **agentic** — รับคำถามภาษาธรรมชาติแล้ววางแผนการค้น แต่ละขั้น ค้นเอง อ่านเอง สังเคราะห์เอง แล้วส่งกลับเป็น **รายงานวิจัยที่มี citation ครบทุกการอ้าง**

ตัวอย่างคำถาม:
> "Research how California courts have treated non-compete agreements for executive employees since 2020."

ผลลัพธ์: รายงานหลายหน้าที่อธิบายการพัฒนาของกฎหมาย พร้อมอ้างคำพิพากษาทุกประเด็น โดยทุก citation เป็น hyperlink กลับไปที่ Westlaw

### Workflow ของ skill

จาก `SKILL.md` workflow มี 4 ขั้น:

1. **Frame the query** — แยกประเด็นจากคำถามผู้ใช้ ถามว่าให้ค้นที่ jurisdiction ไหนถ้ายังไม่ระบุ (สูงสุด 3 jurisdiction ต่อการค้น)
2. **Start Research** — เรียก MCP tool `legal_research_start_deep_research(query, jurisdictions)` ได้ `conversation_id` กลับมา
3. **Poll for Completion** — เรียก `check_deep_research_status` ซ้ำ ๆ จนกว่าจะเสร็จ (`is_terminal: true`) — ระหว่างนั้นต้อง `sleep` ตาม `next_action_poll_backoff_ms`
4. **Retrieve and Present Report Verbatim** — เรียก `get_deep_research_report` แล้วเอา `answer_text` มาแสดง **โดยไม่แก้ไขใด ๆ** (เพราะมี anchor citation, blockquote, horizontal rule ที่ถูกออกแบบมาให้แสดงผลในรูปแบบเฉพาะ)

ขั้นที่ 4 สำคัญมาก — Claude **ห้ามแก้ไข, ตัด, หรือเรียบเรียงใหม่** เพราะ Thomson Reuters เป็นผู้ออกแบบ format ของรายงาน และต้องการให้ผู้ใช้เห็น "ผลลัพธ์จาก Westlaw" ตามที่เป็น ไม่ใช่ "การตีความของ Claude เกี่ยวกับ Westlaw"

### กฎการใช้ — When to Use vs When NOT to Use

`SKILL.md` ระบุชัดเจนว่า Deep Research **ไม่ใช่เครื่องมือทั่วไป** — ออกแบบมาเฉพาะการค้นคว้าที่ต้องการคำตอบเชิงสังเคราะห์ จึงไม่เหมาะกับ:

| ไม่เหมาะ | ทางเลือก |
|---|---|
| ขอข้อความเต็มของเอกสารชิ้นใดชิ้นหนึ่ง | ใช้ traditional search box ของ Westlaw |
| ขอสรุปสาระของกฎหมายเฉพาะมาตรา | ใช้ search box ปกติ |
| ขอ analytics (เช่น "ผู้พิพากษา X ตัดสินให้ฝ่ายใดบ่อยกว่า") | ใช้ Litigation Analytics |
| ขอการคำนวณ (เช่น "วันสุดท้ายที่ยื่นได้") | นอกขอบเขต |
| ขอคำพยากรณ์ผลคดี | นอกขอบเขต |
| ขอร่างเอกสาร | ใช้ CoCounsel (ผลิตภัณฑ์อื่น) |
| ข้อมูลกฎหมายต่างประเทศ | ใช้ Westlaw UK / Canada / International |
| เปรียบเทียบเกิน 3 jurisdiction | ใช้ AI Jurisdictional Surveys |

นี่คือการบอกผู้ใช้อย่างตรงไปตรงมาว่า **เครื่องมือนี้เก่งเรื่องเดียว: การสังเคราะห์เชิงกฎหมายข้ามแหล่งอ้างอิง** ไม่ใช่เครื่องมือ "ค้นทุกอย่าง"

### กฎการสื่อสารกับผู้ใช้

`SKILL.md` มีหมวด **Communication Rules** ที่น่าสนใจ:

> "Never mention tool calls, tool-call budgets, polling, status checks, internal limits, conversation IDs, percent_complete, or any other implementation details to the user."

แปลว่า — Claude **ห้ามบอกผู้ใช้** ว่ากำลัง poll status, ว่ามี conversation_id อะไร, หรือว่าตอนนี้ทำไป 30% แล้ว ให้พูดในระดับ **เนื้องาน** เท่านั้น ("กำลังค้นคว้า...", "พบประเด็นสำคัญในด้าน X...")

นี่เป็น UX design ที่ Thomson Reuters ออกแบบเพื่อให้ผู้ใช้รู้สึกว่ากำลังคุยกับ "นักวิจัยกฎหมาย" ไม่ใช่ "เครื่องมือ API"

## การใช้งานคู่กับ Plugin หลัก

Deep Research **ไม่ได้แทน** plugin ที่ Anthropic เขียนเอง แต่ **เสริม** ในจังหวะที่เหมาะสม:

### กับ `litigation-legal`

ในงานคดี — ทนายมักต้องการรู้ว่า **ศาลในเขตอำนาจนี้** ตัดสินประเด็นแบบนี้อย่างไร เพื่อเตรียม motion หรือ brief

- ใช้ `litigation-legal:research-memo` เพื่อให้ Claude ร่างบันทึกการวิจัย
- ใน workflow บันทึก ตอนที่ต้องการ caselaw จริงพร้อม citation — `deep-research` จะถูกเรียกใช้
- ผลลัพธ์ Deep Research ถูกอ้างใน research-memo ที่ส่งให้หัวหน้าทีม

### กับ `ip-legal`

ในงานทรัพย์สินทางปัญญา (intellectual property) — มักต้องการรู้ว่าศาลเคยตีความ "fair use" หรือ "infringement" อย่างไร

- `ip-legal:claim-chart` ต้องการเอกสารอ้างอิงคำพิพากษาที่เกี่ยวกับการละเมิดสิทธิบัตร
- `deep-research` จะถูกเรียกให้ค้น caselaw ที่ใกล้เคียงที่สุดกับรูปแบบการละเมิดที่อ้าง

### กับ `regulatory-legal`

ในงานกำกับดูแล (regulatory) — มักต้องเข้าใจ **การตีความ regulation โดยศาล** (ไม่ใช่แค่ตัวบท)

- `regulatory-legal:reg-feed-watcher` แจ้งเตือนเมื่อมี regulation ใหม่
- เมื่อพบ regulation สำคัญ — `deep-research` ค้น caselaw ที่ตีความ regulation นั้น เพื่อเข้าใจวิธีที่ศาลใช้ในทางปฏิบัติ

## ข้อสำคัญ — ต้องมี Thomson Reuters Subscription

Plugin นี้ **ไม่ฟรี** การจะใช้ต้องมี:

1. **CoCounsel Legal subscription** ที่ active กับ Thomson Reuters
2. **MCP connector** เปิดสำหรับบัญชีของผู้ใช้ (อาจต้องเปิดด้วย admin)
3. **OAuth login** สำเร็จกับ Thomson Reuters identity provider

หากปัญหาเรื่องสิทธิ์: ติดต่อ `cocounselsupport@tr.com`

จุดสำคัญที่ Anthropic ต้องการสื่อใน README:

> "Subscription required: CoCounsel Legal subscription with the MCP connector enabled for the user's account. Direct entitlement or access questions to cocounselsupport@tr.com."

แปลว่า — **Anthropic จะไม่ตอบคำถามเรื่อง billing, access, หรือ entitlement** ให้ติดต่อ Thomson Reuters โดยตรง

## ข้อกฎหมายและความเป็นส่วนตัว

เพราะข้อมูลที่ผ่าน MCP server ไป Thomson Reuters ไม่ใช่ข้อมูลในเครื่องของผู้ใช้อีกต่อไป — ผู้ใช้ควรอ่าน policy ของ Thomson Reuters เอง:

- **Privacy:** [thomsonreuters.com/en/privacy-statement.html](https://www.thomsonreuters.com/en/privacy-statement.html)
- **Terms of Use:** [thomsonreuters.com/en/terms-of-use.html](https://www.thomsonreuters.com/en/terms-of-use.html)
- **Accessibility:** [thomsonreuters.com/en/policies/accessibility.html](https://www.thomsonreuters.com/en/policies/accessibility.html)
- **Documentation:** [legal-mcp.thomsonreuters.com/docs/connector-guide](https://legal-mcp.thomsonreuters.com/docs/connector-guide)

## สรุป — Pattern ที่จะตามมา

`cocounsel-legal/` คือ **first-mover** ของ external plugin ใน `claude-for-legal` — เป็นตัวอย่างให้ vendor รายอื่นเห็นว่า:

1. การร่วม ecosystem ของ Claude ทำได้โดยไม่ต้องให้ Anthropic เป็นคนเขียน plugin
2. MCP เป็น standard ที่เปิดให้ vendor สร้าง server ของตัวเอง
3. OAuth flow ทำให้การ authenticate ไหลลื่นโดยไม่ต้องแชร์ credential
4. ปรัชญา "vendor-agnostic core + vendor-specific extensions" เป็นจริงในระดับ implementation

ในอีก 6-12 เดือนข้างหน้า อาจเห็น LexisNexis, Bloomberg Law, Fastcase หรือ vLex เข้ามาวาง plugin ของตัวเองใน `external_plugins/` ตามรอย CoCounsel Legal
