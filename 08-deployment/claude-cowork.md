---
title: Claude Cowork
parent: 08 Deployment
nav_order: 1
---

# ติดตั้ง Claude for Legal ใน Claude Cowork

> "After install, skills fire automatically when relevant, slash commands are available via `/`, and the scheduled agents run on the cadence set in their frontmatter."
> — `README.md`

**Claude Cowork** คือ desktop app ของ Anthropic ที่ทนายความส่วนใหญ่จะใช้เป็น surface หลัก เพราะมี sidebar รวมกับ Word, Excel, Outlook, Slack ของ Microsoft 365 ได้โดยตรง — ทำให้ทนายไม่ต้องออกจาก document ที่กำลัง review เพื่อเปิด terminal

## ใครควรใช้ Cowork

| ลักษณะงาน | เหมาะกับ Cowork? |
|------|----------|
| Review NDA / vendor agreement ใน Word ทุกวัน | ใช่ — ใช้ tracked changes ใน sidebar |
| ตรวจ marketing claim ก่อน launch | ใช่ — สนทนาแบบ chat |
| สรุปสำหรับ business stakeholder | ใช่ — copy-paste ตรงจาก sidebar |
| Batch review 50 vendor agreement พร้อมกัน | ไม่ — ใช้ Claude Code |
| Scheduled renewal watcher รันทุกจันทร์ | ไม่ — ใช้ Managed Agents |

## ขั้นตอนติดตั้ง (60 วินาที)

### ขั้นที่ 1: ติดตั้ง Claude Desktop

ดาวน์โหลดจาก [claude.com/download](https://claude.com/download) และเปิด account ของ Anthropic

### ขั้นที่ 2: เปิด Cowork tab

- เปิดแอป Claude Desktop
- คลิก **Cowork** tab ที่แถบด้านบน
- คลิก **Customize** ที่แถบซ้ายมือ

### ขั้นที่ 3: ติดตั้ง plugin จาก marketplace

- คลิก **Browse plugins** — จะเห็นรายการ 12 plugin ของ Claude for Legal
- เลือก plugin ที่เหมาะกับงาน — เช่น `commercial-legal` สำหรับทนาย contracts, `privacy-legal` สำหรับ DPO
- คลิก **Install**

หรือถ้าต้องการอัปโหลด custom plugin (เช่น plugin ที่ fork มาแก้ไข):

- ซิป plugin directory ทั้งหมด (เช่น `commercial-legal.zip`)
- คลิก **Upload custom plugin file**
- เลือกไฟล์ zip

> *"any plugin directory zipped up"* — `README.md`

## ขั้นตอนกรอก Practice Profile

หลังติดตั้งแล้ว **อย่ารีบใช้ skill** — ต้องรัน **cold-start interview** ก่อน ไม่งั้น skill จะให้ผลลัพธ์ generic

### Cold-start interview คืออะไร

เป็น skill พิเศษที่ Anthropic ออกแบบให้เป็น **wizard** สำหรับสัมภาษณ์ผู้ใช้ — จะถามว่า

- ทำงาน sales-side หรือ purchasing-side?
- สำนักงานมีกี่คน?
- ใช้ CLM ตัวไหน (Ironclad, DocuSign, iManage)?
- threshold การ escalate เท่าไหร่ ($50K? $500K?)?
- รัฐ/ประเทศที่ทำธุรกิจหลัก?
- มี playbook หรือ template ที่ใช้อยู่ไหม?

ผลลัพธ์ของการสัมภาษณ์ → เขียนลงไฟล์ `CLAUDE.md` ที่เรียกว่า **practice profile** ซึ่ง skill ทุกตัวใน plugin จะอ่านไฟล์นี้ก่อนทำงานเสมอ

### เริ่มสัมภาษณ์

ในแชท Cowork พิมพ์:

```
/commercial-legal:cold-start-interview
```

Claude จะเริ่มถามทีละข้อ — มีให้เลือก **quick start** (2 นาที สำหรับใช้งานทันที) หรือ **full** (10–20 นาที สำหรับผลผลิตคุณภาพสูงสุด)

> *"More seed material is better; a quick start option is available if you want to be productive in 2 minutes and refine later."* — `README.md`

หากมี **seed document** เช่น MSA ที่เซ็นแล้ว, playbook ภายใน, memo ตรวจสอบเก่า — ชี้ให้ Claude อ่านในระหว่างสัมภาษณ์ จะทำให้ profile แม่นยำขึ้นมาก

### แก้ไข profile หลังสัมภาษณ์

Profile บันทึกที่ — บน macOS / Linux:

```
~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md
```

แก้ไขโดยตรงได้สำหรับการเปลี่ยนเล็ก ๆ (threshold ใหม่, contact ใหม่) ถ้าเปลี่ยนใหญ่ (เข้า jurisdiction ใหม่, เปลี่ยน CLM) — รัน `cold-start-interview` ใหม่

## ตัวอย่างการใช้งานจริง

### กรณี 1: Review NDA ใน Word

1. เปิดเอกสาร NDA ใน Microsoft Word
2. คลิก **Claude sidebar** ในแถบ ribbon
3. พิมพ์:

```
/commercial-legal:review
```

4. Claude จะ:
   - อ่าน NDA จาก Word ปัจจุบัน
   - เปรียบเทียบกับ playbook ใน practice profile
   - เสนอ tracked changes (insert/delete) ลงใน Word
   - เพิ่ม memo สรุปประเด็นที่ต้องเจรจา
5. ทนาย accept/reject ทีละจุดเหมือน review markup จากเพื่อนร่วมงาน

### กรณี 2: ตอบ DSAR (Data Subject Access Request)

```
/privacy-legal:dsar-response DSAR-2026-0142
```

Claude จะ:
- ตรวจ identity ของผู้ร้อง
- ระบุ data sources ใน Google Drive / CRM
- ประเมินว่าคำขอครอบคลุมข้อมูลใดบ้าง
- ร่าง response email พร้อม attached data summary
- แสดง deadline ตาม jurisdiction (CCPA 45 วัน, GDPR 30 วัน)

### กรณี 3: ตรวจ marketing claim ก่อน launch

ในแชท Cowork:

```
/product-legal:marketing-claims-review
```

จากนั้น paste ข้อความ marketing ลงไป — Claude จะ flag ข้อความที่:
- ต้องมี substantiation (เช่น "fastest", "most accurate")
- ต้องปรับ wording (เช่น claim ที่ทำให้เข้าใจผิด)
- ต้องตัดออก (false claim)

## Scheduled agent ใน Cowork

Cowork รัน scheduled agent ตาม cron schedule ที่กำหนดใน frontmatter ของ agent — เช่น `renewal-watcher` รันทุกเช้าวันจันทร์ 9:00 น. และโพสต์รายงานเข้า Slack channel ที่กำหนด

ดูรายการ scheduled agent ทั้งหมดได้ที่ [03-plugins](../03-plugins/index.html)

## ความเข้าใจผิดที่พบบ่อย

| สถานการณ์ | สาเหตุที่แท้จริง | แก้ไข |
|----------|------------------|------|
| Skill ให้ผลลัพธ์ generic ไม่ตรงสไตล์สำนักงาน | ยังไม่ได้รัน cold-start-interview | รัน `/<plugin>:cold-start-interview` |
| Citation ขึ้น `[verify]` ทุกที่ | ยังไม่ได้ auth research connector | Settings → Connectors → CourtListener / Westlaw |
| Plugin หาไฟล์ในเครื่องไม่เจอ | Plugin scope เป็น project-scoped | uninstall แล้ว reinstall เป็น user-scoped |
| Tracked changes ไม่ขึ้นใน Word | ยังไม่ติดตั้ง Claude for Microsoft 365 | ติดตั้งจาก Microsoft AppSource |

## สรุป Cowork

Cowork = surface สำหรับ **ทนายความ end-user** — ติดตั้งเสร็จใน 60 วินาที, สนทนาเหมือน chat กับเพื่อนร่วมงาน, สามารถพิมพ์ slash command เพื่อ trigger skill ใด ๆ ก็ได้ใน plugin ที่ติดตั้ง

ขั้นต่อไป — ถ้าต้องการ batch automation หรือทำงานใน terminal → อ่าน [claude-code](claude-code.html)
