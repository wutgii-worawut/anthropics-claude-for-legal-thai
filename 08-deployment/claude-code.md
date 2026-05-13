---
title: Claude Code
parent: 08 Deployment
nav_order: 2
---

# ติดตั้ง Claude for Legal ใน Claude Code

> "If you have a terminal window open with Claude in it, that's Claude Code."
> — `QUICKSTART.md`

**Claude Code** คือ CLI ของ Anthropic ที่รันใน terminal — เหมาะกับ **นักพัฒนา, power user, legal-ops engineer** ที่ต้องการ:

- **Batch operation** — review เอกสารหลายชิ้นพร้อมกัน
- **File-based workflow** — input/output เป็นไฟล์ในโฟลเดอร์
- **Reproducible setup** — ใช้ git tracking practice profile
- **Pipe กับ shell** — เชื่อมต่อกับ script อื่น ๆ ใน workflow

## ความต่างระหว่าง Cowork กับ Claude Code

| มิติ | Cowork | Claude Code |
|------|--------|-------------|
| Trigger | คลิกหรือพิมพ์ใน sidebar | พิมพ์ใน terminal |
| Input | Word/Excel/Drive ผ่าน sidebar | path ของไฟล์ในเครื่อง |
| Output | แสดงในแชท + sidebar | เขียนไฟล์ในโฟลเดอร์ปัจจุบัน |
| Plugin install | คลิก install จาก marketplace | `/plugin install <name>@claude-for-legal` |
| Practice profile location | `~/.claude/plugins/config/.../<plugin>/CLAUDE.md` | `~/.claude/plugins/config/.../<plugin>/CLAUDE.md` (ที่เดียวกัน) |
| Connector | OAuth pop-up ใน UI | OAuth ครั้งแรกใน browser + token เก็บใน `.mcp.json` |
| ผู้ใช้หลัก | ทนายความ | นักพัฒนา / legal-ops |

ข้อสำคัญ — **practice profile เก็บที่เดียวกัน** ดังนั้น user คนเดียวที่ใช้ทั้ง Cowork และ Claude Code จะใช้ profile ร่วมกัน — กรอกครั้งเดียวใช้ได้ทั้งสอง surface

## ขั้นตอนติดตั้ง

### ขั้นที่ 1: เตรียม Claude Code

ติดตั้ง Claude Code ผ่าน CLI ที่ตรงกับ OS — ดูคู่มือที่ [claude.com/product/claude-code](https://claude.com/product/claude-code)

### ขั้นที่ 2: เพิ่ม marketplace

มี 2 วิธี:

**วิธี A — ใช้ path ของ repo ที่ clone มา:**

```bash
# ใน Claude Code shell
/plugin marketplace add /Users/you/Desktop/claude-for-legal
```

หรือใช้วิธีลากโฟลเดอร์ลงใน terminal (drag-and-drop):

```
/plugin marketplace add <space>
```

แล้วลากโฟลเดอร์ `claude-for-legal` ที่ unzip แล้วลงใน terminal — path จะถูกเติมให้อัตโนมัติ

**วิธี B — ใช้ GitHub URL โดยตรง:**

```bash
/plugin marketplace add https://github.com/anthropics/claude-for-legal
```

### ขั้นที่ 3: ติดตั้ง plugin

เลือก plugin ที่ตรงกับงาน — สามารถติดตั้งหลายตัวพร้อมกันได้:

```bash
/plugin install commercial-legal@claude-for-legal
/plugin install privacy-legal@claude-for-legal
/plugin install corporate-legal@claude-for-legal
```

### ขั้นที่ 4: **Restart Claude Code (จำเป็น)**

> *"Restart Claude Code. Close and reopen. This step is not optional — the plugin isn't live until you restart."* — `QUICKSTART.md`

ปิด terminal แล้วเปิดใหม่ — plugin จะ active หลัง restart เท่านั้น

### ขั้นที่ 5: เลือก user scope (สำคัญ)

ตอนติดตั้งจะมีคำถามว่าจะ install สำหรับ project นี้เท่านั้น หรือทุก project (user scope) — **เลือก user scope**

> *"It's counterintuitive: project scope feels safer. But project scope blocks the plugin from reading files outside the project folder."* — `QUICKSTART.md`

เหตุผล: skill ส่วนใหญ่ต้องการอ่านไฟล์ผู้ใช้ — เช่น contract ใน `Documents/`, playbook ใน `Downloads/`, matter file ใน `Dropbox/` — ถ้าเลือก project scope จะอ่านไม่ได้

> **User scope ไม่ได้ให้สิทธิ์ extra** — plugin อ่านไฟล์ได้แค่ไฟล์ที่ user ชี้ให้อ่าน หรือไฟล์ใน current directory เท่านั้น แค่ทำให้ plugin ใช้งานได้จากทุก folder แทนที่จะใช้ได้แค่ folder เดียว

หากลืมเลือก user scope:

```bash
/plugin uninstall <plugin>
cd ~  # กลับไป home directory
/plugin install <plugin>@claude-for-legal
```

### ขั้นที่ 6: รัน cold-start interview

```bash
/commercial-legal:cold-start-interview
/privacy-legal:cold-start-interview
/corporate-legal:cold-start-interview
```

> *"Run the cold-start interview first. Every other skill in a plugin reads from the practice profile it writes. Skipping setup is the single most common reason a skill produces generic output."* — `README.md`

ในแต่ละ plugin ต้องรัน interview แยกกัน เพราะ practice profile แยกตาม plugin (sales-side สำหรับ commercial vs. privacy posture สำหรับ privacy)

### ขั้นที่ 7: เชื่อม research connector

> *"Start by connecting a research tool. Everything else is better with one, and citations are unverified without one."* — `README.md`

Plugin แต่ละตัวมี `.mcp.json` ที่กำหนด research connector มาแล้ว — Claude Code จะถาม authorize ครั้งแรกที่ skill เรียกใช้ connector citation guardrail จะรู้จัก:

- **CourtListener** — federal dockets และ opinions (ฟรี)
- **Trellis** — state court dockets
- **Descrybe** — case law research
- **Solve Intelligence** — patent research
- **CoCounsel Legal** (Westlaw, ผ่าน external plugin)

## Practice profile ใน Claude Code

Path มาตรฐาน:

```
~/.claude/plugins/config/claude-for-legal/<plugin>/CLAUDE.md
```

**ข้อดี**: เป็นไฟล์ markdown ธรรมดา — ใช้ git track ได้, diff ได้, share ทีมได้

ตัวอย่าง workflow ทีม:

```bash
# คนแรกกรอก profile
/commercial-legal:cold-start-interview

# Commit profile ลง shared repo
cp ~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md \
   ~/team-legal-config/commercial-legal.md
cd ~/team-legal-config && git add . && git commit -m "Update commercial-legal profile"

# เพื่อนร่วมงาน pull ไปใช้
git pull
cp ~/team-legal-config/commercial-legal.md \
   ~/.claude/plugins/config/claude-for-legal/commercial-legal/CLAUDE.md
```

## ตัวอย่างการใช้งาน

### กรณี 1: Batch review vendor agreements

```bash
cd ~/Documents/vendor-contracts-2026Q2
ls *.docx > files.txt

# ใน Claude Code
for f in $(cat files.txt); do
  /commercial-legal:review "$f"
done
```

ผลลัพธ์: review memo + tracked changes สำหรับทุกไฟล์

### กรณี 2: รัน chronology builder จาก declarations

```bash
cd ~/matters/acme-v-us-2026
/litigation-legal:chronology declarations/
```

Claude อ่าน declarations ใน folder, สกัด timeline, เขียน `chronology.md`

### กรณี 3: ตรวจ legal hold

```bash
/litigation-legal:legal-hold --action=refresh --matter=acme-v-us-2026
```

## อัปเดต plugin

```bash
/plugin update
```

จะตรวจ marketplace ว่ามี version ใหม่ของ plugin ที่ติดตั้งไหม — Practice profile **ไม่ถูกแทนที่** เมื่อ update

## ความเข้าใจผิดที่พบบ่อย

| ปัญหา | สาเหตุ | แก้ไข |
|------|--------|------|
| "Command not found" หลัง install | ลืม restart Claude Code | ปิด-เปิด terminal ใหม่ |
| "Run setup first" | ยังไม่ได้รัน cold-start | `/<plugin>:cold-start-interview` |
| Citation ขึ้น `[verify]` ทุกที่ | ยังไม่ auth research connector | รัน skill ที่ต้องใช้ connector — จะมี OAuth prompt |
| Plugin หาไฟล์ไม่เจอ (ใน Documents) | Plugin scope เป็น project-scoped | uninstall แล้ว reinstall จาก home directory |
| Plugin ไม่ตอบคำถามที่ต้องการ | Plugin ผิดตัว / skill ไม่ตรง | `/legal-builder-hub:related-skills-surfacer` |

## สรุป Claude Code

Claude Code = surface สำหรับ **นักพัฒนา / legal-ops** — install ทาง CLI, file-based input/output, สามารถ batch + scripting ได้

ขั้นต่อไป — ถ้าต้องการ scheduled agent หรือ headless deployment → อ่าน [managed-agents](managed-agents.html)
