---
title: ip-legal — ทรัพย์สินทางปัญญา
parent: 03 Plugins
nav_order: 9
---

# ip-legal — ทรัพย์สินทางปัญญา

## บทนำ

`ip-legal` คือ plugin สำหรับงานทรัพย์สินทางปัญญา (intellectual property) ครอบคลุม 5 สาขา: เครื่องหมายการค้า (trademark), ลิขสิทธิ์ (copyright), สิทธิบัตร (patent), ความลับทางการค้า (trade secret), และ open source

หัวใจของ plugin นี้คือ **"triage ไม่ใช่ opinion"** — ทุก skill ที่เกี่ยวกับการประเมินความเสี่ยง (clearance, FTO, invention intake, infringement triage) จะ **ปฏิเสธสรุปว่า "clear"** เสมอ — ทำหน้าที่ "first-pass screen" ที่ช่วยทนายสิทธิบัตร/เครื่องหมายการค้าตัดสินใจง่ายขึ้น แต่ไม่ทดแทนการให้ความเห็นทางกฎหมายของผู้เชี่ยวชาญที่จดทะเบียน

จุดที่ต่างจาก `litigation-legal` ชัดเจน: plugin นี้ **ไม่ร่าง patent claim** — งาน patent prosecution ที่ต้องเขียน claim เป็นทักษะเฉพาะของ patent agent / patent attorney ที่จดทะเบียน — plugin นี้เน้น FTO, IP clause review, portfolio tracking, infringement triage แทน

12 skills + 1 scheduled agent (renewal watcher) + empty hooks + 5 MCP server

## `.claude-plugin/plugin.json`

```json
{
  "name": "ip-legal",
  "version": "1.0.2",
  "description": "Runs first-pass trademark clearance and freedom-to-operate triage, screens invention disclosures for initial patentability, drafts and triages cease-and-desist letters and DMCA takedowns (send and respond), checks open source compliance, reviews IP clauses, and tracks registrations and renewal deadlines.",
  "author": {
    "name": "Anthropic"
  }
}
```

แปล: "ทำ trademark clearance ขั้นแรกและ freedom-to-operate triage, คัดกรอง invention disclosure สำหรับ patentability เบื้องต้น, ร่างและคัดกรอง cease-and-desist + DMCA takedown (ส่งและตอบ), ตรวจ open source compliance, ทบทวน IP clause, ติดตามทะเบียนและ deadline ต่ออายุ"

## โครงสร้างโฟลเดอร์

```
ip-legal/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json                        # 5 MCP server: Solve Intelligence, CourtListener, Descrybe, Slack, GDrive
├── .gitignore
├── CLAUDE.md                        # 41 KB — practice profile template
├── README.md                        # 11 KB — คู่มือผู้ใช้
├── agents/
│   └── ip-renewal-watcher.md        # scheduled agent ตรวจ deadline ต่ออายุ
├── hooks/
│   └── hooks.json                   # ว่าง — {"hooks": {}}
├── logs/
│   └── .gitkeep                     # โฟลเดอร์เก็บ log (สร้างไว้รอ)
└── skills/                          # 12 skill
    ├── cease-desist/
    ├── clearance/
    ├── cold-start-interview/
    ├── customize/
    ├── fto-triage/
    ├── infringement-triage/
    ├── invention-intake/
    ├── ip-clause-review/
    ├── matter-workspace/
    ├── oss-review/
    ├── portfolio/
    └── takedown/
```

ความต่างจาก `litigation-legal` ที่ชัดเจน:
- **ไม่มี workspace dir** ขนาดใหญ่แบบ matters/inbound/demand-letters/oc-status (เพราะข้อมูลเก็บใน `portfolio.yaml` ใน config dir แทน)
- **มี `hooks/` folder ที่ว่าง** (`{"hooks": {}}`) — เตรียมไว้สำหรับ future use
- **มี `logs/` folder** สำหรับ output ของ agent

## Skills ทั้งหมด (12 skill)

| Skill | argument-hint | หน้าที่ (ไทย) |
|-------|---------------|---------------|
| `cold-start-interview` | `[--redo \| --check-integrations]` | สัมภาษณ์ตอนติดตั้งครั้งแรก — เรียนรู้ practice area mix (TM/patent/copyright/trade secret/OSS), jurisdiction, enforcement posture, approval matrix |
| `customize` | `[section]` | แก้ profile ทีละส่วน — risk posture, escalation contact, portfolio scope, brand protection, enforcement, clearance threshold, OSS review rule |
| `clearance` | `[mark + goods + jurisdiction]` | Trademark clearance ขั้นแรก — knockout + similar-mark check → flag list **(ไม่ใช่ clearance opinion)** |
| `fto-triage` | `[product/process/feature + jurisdiction]` | Freedom-to-operate triage — first look หา blocking patent **(ไม่ใช่ FTO opinion)** |
| `invention-intake` | `[invention disclosure]` | คัดกรอง invention ขั้นแรก — novelty, obviousness, §101 eligibility, bar date, detectability, strategic value |
| `infringement-triage` | `[facts + which right]` | คัดกรองการละเมิด TM/copyright/patent/trade secret — factor-by-factor flag list **(ไม่ใช่ finding)** |
| `cease-desist` | `<--send \| --receive> [context/path]` | ร่าง C&D (send mode) หรือ คัดกรอง C&D ที่รับเข้ามา (receive mode) พร้อม loud gate ก่อนส่ง |
| `takedown` | `<--send \| --respond \| --counter> [context/path]` | DMCA — ส่งหมายเรียกถอน (§512(c)(3)), ตอบหมายเรียกที่รับเข้ามา, หรือร่าง counter-notice (§512(g)(3)) |
| `ip-clause-review` | `[file/Drive link/text]` | ทบทวน IP clause ในสัญญา — assignment, ownership, license grant, warranty, indemnity |
| `oss-review` | `[manifest/SBOM/repo path]` | ตรวจ open source license compliance — copyleft obligation, license compatibility, ban list |
| `portfolio` | `[--report [--days N] \| --add \| --update \| --audit]` | ติดตาม IP portfolio — registration, renewal, maintenance fee, use declaration |
| `matter-workspace` | `<new \| list \| switch \| close \| none> [slug]` | จัดการ workspace สำหรับ multi-client private practice |

### กฎ "ไม่สรุปว่า clear" — สำคัญมาก

Skill 4 ตัวที่เป็น triage มี disclaimer ฮาร์ดโค้ดใน prompt:

**`clearance`** (เครื่องหมายการค้า):
> "This is a triage, not a clearance opinion. ... A 'no obvious conflicts' result means the triage didn't find anything — it does not mean the mark is clear. Clients have been sued over marks that passed a knockout search."

**`fto-triage`** (อิสรภาพในการดำเนินการ):
> "This is not a freedom-to-operate opinion. ... Patent infringement is strict liability; willful infringement triples damages. A 'no obvious blocking patents' result from this skill means the triage didn't find one — it does not mean the product is clear."

**`invention-intake`** (สิทธิบัตร):
> "This is a first-pass screen by a non-specialist, not a patentability opinion. The screen never concludes that an invention is patentable — it concludes that it passes the initial screen and warrants a prior-art search and registered-practitioner review."

**`infringement-triage`** (การละเมิด):
> "This is a triage, not a finding of infringement or non-infringement. ... Acting on a triage — sending a cease-and-desist, refusing to stop, filing suit, or deciding not to — without attorney review is how companies end up on the wrong side of fee awards, Rule 11 sanctions, declaratory-judgment actions, and (for patents) treble damages."

นี่คือ pattern ที่ Anthropic ใช้ — **disclaimer ไม่ใช่ footnote — มันคือ identity ของ skill** — ปรากฏที่บรรทัดแรกของ prompt body

## Agents

`ip-legal` มี 1 agent — `ip-renewal-watcher`

```yaml
---
name: ip-renewal-watcher
description: Scheduled agent that reads the IP portfolio register, computes what's due, and posts a ranked deadline report. Runs weekly by default. Posts to the channel named in CLAUDE.md → Renewal alerts.
model: sonnet
tools: ["Read", "Write", "mcp__anaqua__*", "mcp__cpa__*", "mcp__altlegal__*", "mcp__*__slack_send_message"]
---
```

**หลักการทำงาน**:

1. อ่าน `CLAUDE.md` หา alert destination (Slack channel / email list / inline)
2. Load `portfolio` skill, refresh computed deadline ทุก asset (ไม่เชื่อ date ที่เก็บไว้อย่างเดียว) แล้วรัน Mode 2 window 90 วัน
3. **Immediate-escalation check** — ถ้ามี deadline อยู่ใน `grace` หรือ `lapsed` โพสต์ทันทีไม่รอ schedule
   - US §8 declaration: grace 6 เดือน + surcharge
   - US patent maintenance fee: grace 6 เดือน + surcharge
   - ทั้งคู่ — พลาด = เสีย asset
4. **IP management system cross-reference** — ถ้า Anaqua / CPA Global / Alt Legal connected และ register ไม่ sync >30 วัน → sync ก่อน, reconcile (system of record ชนะตอน conflict)
5. โพสต์ report

**Schedule**: รายสัปดาห์ (จันทร์เช้า) — ปรับได้ (portfolio ขนาดใหญ่รายวัน, portfolio เล็กรายเดือน)

**Output format** (ตัด snippet):

```
📅 IP Portfolio — week of [date]

🔴 IN GRACE / LAPSED ([N])
• [Asset ID] / [Jurisdiction] / [Mark or title]
  [Action] — original due [date], grace ends [date]
  Owner: [business owner] | Counsel: [firm or docket ID]

⏰ DUE WITHIN 30 DAYS ([N])
🟠 DUE 30-60 DAYS ([N])
🟡 DUE 60-90 DAYS ([N])
🌐 AGENT-MANAGED ([N])         # managed by local agent — confirm directly
❓ UNKNOWN ([N])               # missing data; cannot compute
```

**Guardrail** — ทุก run repeat caveat ว่า:
- IP deadline เป็น jurisdiction-specific
- บางอย่างมี grace + surcharge บางอย่างไม่มี
- docketed-but-wrong deadline แย่กว่า undocketed deadline เพราะสร้าง false confidence
- agent เป็นแค่ surfacing tool — ไม่ใช่ system of record

**ที่ agent นี้ "ไม่ทำ"**:
- ไม่ file อะไร — ทนายหรือ foreign associate execute
- ไม่จ่าย maintenance fee / annuity — CPA Global ทำหน้าที่นั้น
- ไม่ตัดสินใจว่าจะต่ออายุหรือไม่ — เป็น business + legal call
- ไม่แก้ register — เพิ่ม/แก้ผ่าน `/ip-legal:portfolio --add` / `--update` / sync จาก IP management system
- ไม่ ping business owner โดยตรง — channel post tag แทน

## Hooks

```json
{
  "hooks": {}
}
```

ว่างเปล่า — เตรียมไว้สำหรับ future use

## Workspace dirs

`ip-legal` ไม่มี workspace dir ขนาดใหญ่แบบ `litigation-legal` — ข้อมูล portfolio ทั้งหมดเก็บอยู่ใน:

```
~/.claude/plugins/config/claude-for-legal/ip-legal/
├── CLAUDE.md                  # practice profile
└── portfolio.yaml             # IP asset register (เขียนโดย /portfolio --add/--update)
```

**Matter workspace (optional)** — ถ้าเป็น private practice ที่มีหลายลูกความ จะใช้ `/matter-workspace new <slug>` สร้าง `matters/[slug]/matter.md` แบบเดียวกับ litigation แต่ structure เบากว่ามาก

## References / playbooks

ทุก skill load `CLAUDE.md` ก่อนทำงาน — ถ้า `CLAUDE.md` มี `[PLACEHOLDER]` หรือ `[Your Company Name]` อยู่ skill จะ **ปฏิเสธทำงาน** และบอกให้รัน `/cold-start-interview` ก่อน

นี่คือ pattern ที่ดีของ Anthropic — **"refuse to act without context"** — สำคัญมากใน IP เพราะ enforcement posture ของแต่ละองค์กรต่างกันมาก (aggressive / measured / conservative)

## Connectors (MCP)

5 MCP server ใน `.mcp.json`:

| Server | บทบาท |
|--------|-------|
| **Solve Intelligence** | งาน patent — ค้น patent + non-patent literature, legal text, SEP technical standard, prior art, claim analysis |
| **CourtListener** | Free Law Project — opinion ศาลสหรัฐ, PACER docket, profile ผู้พิพากษา, oral argument, citation verification |
| **Descrybe** | Primary law research — ค้น case ตาม concept หรือถ้อยคำ, หา case จาก citation, extract authority, check treatment, verify quoted language |
| **Slack** | ค้น message, อ่าน channel, หา discussion |
| **Google Drive** | ค้น/อ่าน/ดึงเอกสาร |

`recommendedCategories`: `ip-management, legal-research, case-law, documents, chat, email`

**Optional MCP** (ที่ agent อ้างถึงแต่ไม่อยู่ใน `.mcp.json` default) — `Anaqua`, `CPA Global`, `Alt Legal` — เป็น IP management system ที่ผู้ใช้ติดตั้งเอง

**ข้อสังเกต**: USPTO ไม่อยู่ใน MCP list — เพราะ USPTO Trademark Status & Document Retrieval (TSDR) และ Madrid Monitor ของ WIPO ไม่ใช่ MCP — agent บอกให้ "verify against USPTO TSDR / WIPO Madrid Monitor / the relevant registry before filing or paying"

## Workflow ตัวอย่าง

### Case 1: ตรวจ trademark clearance ก่อนเปิดตัว brand ใหม่

1. **เริ่มที่ clearance** — `/ip-legal:clearance "BluePeak for cloud storage services in US/EU"`
2. **Skill รัน**:
   - Knockout search หา exact match
   - Similar-mark check ตาม likelihood-of-confusion factor (DuPont factors สำหรับ US, EU equivalent สำหรับ EU)
   - ดู goods/services overlap
3. **Output**: flag list — "found X marks worth attorney review" (**ไม่บอกว่า clear**)
4. **ถ้าผ่าน flag เบื้องต้น** → ส่งให้ trademark counsel ทำ full professional search
5. **ถ้าพบ blocker** → ตัดสินใจ rebrand / coexistence agreement / consent letter
6. **ถ้าตัดสินใจส่ง** → handoff `/portfolio --add` ลงทะเบียนใน register

### Case 2: รับ C&D เครื่องหมายการค้า → ตัดสินใจตอบ

1. **รับจดหมาย** — `/ip-legal:cease-desist --receive ~/Downloads/cd-from-acme.pdf`
2. **Skill รัน**:
   - Extract: ฝ่ายส่ง, mark/work อ้างสิทธิ์, ฐานกฎหมาย (TM/copyright/patent), ระยะเวลา, threat
   - Cross-check `portfolio.yaml` หา conflict กับ asset ของเราเอง
   - Analyze merit: ความน่าเชื่อถือของข้ออ้าง, defense ที่มี (fair use, prior use, descriptive, abandonment ฯลฯ)
3. **Output**: options memo
   - **Comply** — หยุดใช้ + ลบ + รับรอง
   - **Comply partial** — เปลี่ยน mark / scope
   - **Engage** — เจรจา coexistence
   - **Counter** — ปฏิเสธพร้อมแสดง defense
   - **Ignore** — strategic call (ถ้าฝ่ายตรงข้ามไม่มีฐาน)
4. **ทนายตัดสินใจ** — ถ้า counter → drafts response letter (ผ่าน gate เดียวกับ send mode)

### Case 3: ส่ง DMCA takedown notice — gate Lenz + perjury

1. **ระบุการละเมิด** — เห็น content ของเราโดน copy ใน platform third party (YouTube, GitHub, marketplace)
2. **เริ่ม takedown** — `/ip-legal:takedown --send`
3. **Skill รัน gate**:
   - **Fair-use gate (Lenz v. Universal)** — ตรวจว่าผู้ใช้มี fair use argument ที่ชัดเจนหรือไม่ (Lenz บังคับให้ rights holder พิจารณา fair use ก่อนส่ง takedown) — ถ้าใช่ ปฏิเสธหรือต้องระบุเหตุผลชัดเจน
   - **Loud perjury / §512(f) gate** — ผู้ส่งต้องสาบานภายใต้บทลงโทษฐานเบิกความเท็จ; §512(f) ให้สิทธิ defendant ฟ้องคืน "knowing material misrepresentation" — gate บังคับให้ผู้ใช้รับทราบและ confirm
4. **Draft §512(c)(3) notice** — มี element ครบ: identification ของ work, identification ของ infringing material + location, contact info, good-faith statement, accuracy statement (under perjury), signature
5. **Send** — ผ่าน platform's DMCA agent

### Case 4: รับ DMCA takedown — draft counter-notice

1. **รับ takedown** จาก platform แจ้งว่า content ของเราถูก takedown
2. **`/ip-legal:takedown --respond ~/Downloads/dmca-notice.pdf`**
3. **Skill วิเคราะห์**: ตัวเลือก comply / counter / engage / ignore
4. **ถ้าเลือก counter** → `/ip-legal:takedown --counter`
5. **Loud gate ที่ §512(g)(3) บังคับ**:
   - **Consent to federal jurisdiction** — counter-notice ทำให้ผู้ส่งยอมรับเขตอำนาจศาลกลางใน district ที่ address อยู่ — ถ้าฝ่ายส่ง takedown เดิมเป็นต่างประเทศ จะอยู่ใน district ที่ platform ตั้ง — gate บังคับให้ user รับทราบ
   - **Perjury statement** — sworn under penalty of perjury ว่าเชื่อโดยสุจริตว่า content ถูก takedown ผิด
6. **ถ้าผ่าน gate** → draft counter-notice ครบ element

### Case 5: ตรวจ open source compliance ก่อน ship

1. **เริ่ม oss-review** — `/ip-legal:oss-review ~/project/package.json`
2. **Skill รัน**:
   - Classify ทุก dependency ตาม license family (permissive: MIT/BSD/Apache; weak copyleft: LGPL/MPL; strong copyleft: GPL/AGPL; non-OSI: Commons Clause, BSL, SSPL)
   - Map obligation ตาม deployment model (SaaS — AGPL ติด, on-prem — GPL ติด, library — LGPL ต่าง)
   - Flag license-unknown packages + non-OSI ที่ปลอมเป็น OSS
   - Cross-check `CLAUDE.md` ban list / accept list ของบริษัท
3. **Output**: per-dependency recommendation — comply (add attribution) / replace / remove / seek legal review / seek commercial license

### Case 6: ติดตาม renewal deadline

1. **Weekly run** — agent `ip-renewal-watcher` อ่าน `portfolio.yaml`
2. **คำนวณ deadline 90 วัน** — รายสัปดาห์
3. **Escalate immediate** — ถ้าเจอ grace/lapsed → โพสต์ทันที
4. **Sync IP management system** ถ้า connected (Anaqua/CPA/Alt Legal)
5. **โพสต์ Slack** — bucket ตาม urgency (🔴 grace, ⏰ <30d, 🟠 30-60d, 🟡 60-90d, 🌐 agent-managed, ❓ unknown)
6. **Business owner / outside counsel** ตัดสินใจว่าต่ออายุหรือไม่
7. **Filing** ทำผ่าน CPA Global หรือ foreign associate — agent ไม่ file เอง

## ข้อควรระวัง

### Triage ≠ Opinion

- ทุก triage skill (clearance, FTO, invention-intake, infringement-triage) **ไม่สรุปว่า "clear"** — เป็น disclaimer ที่ฮาร์ดโค้ดและ user override ไม่ได้
- "No obvious conflict" ≠ "clear" — มี client โดนฟ้องเรื่อง mark ที่ผ่าน knockout search
- Patent infringement = **strict liability** — willful infringement = treble damage; ทำตาม triage โดยไม่ให้ patent counsel ตรวจคือเสี่ยงสูงสุด

### Fair use (Lenz)

- DMCA takedown ในสหรัฐ — *Lenz v. Universal Music Corp.* บังคับให้ rights holder พิจารณา fair use ก่อนส่ง — `takedown --send` มี gate บังคับ
- §512(f) — knowing material misrepresentation ใน takedown notice → liable ต่อ defendant — gate perjury บังคับ

### §512(g)(3) counter-notice — consent to federal jurisdiction

- การส่ง counter-notice = ยอมรับเขตอำนาจศาลกลางสหรัฐใน district ที่ address อยู่
- ถ้า user อยู่ต่างประเทศ → consent ต่อ district ที่ platform ตั้ง (เช่น N.D. Cal. สำหรับ Google/YouTube)
- Gate บังคับให้ user รับทราบและ confirm explicit

### Bar date / statutory deadline

- **§102 bar dates** — invention-intake ต้องตรวจว่ามี public disclosure / sale / use ที่ทำให้ patent ใช้ไม่ได้
- **Madrid Protocol** — มี deadline 6 เดือนสำหรับ international application priority
- **Maintenance fee window** — 3.5/7.5/11.5 ปีหลัง grant สำหรับ US utility patent — มี grace 6 เดือนพร้อม surcharge — พลาด = patent expired

### Privilege ใน C&D

- C&D ที่ส่งออก = **ไม่ privileged** (ส่งให้บุคคลที่สามแล้ว) — ใช้เป็น admission ได้
- Internal memo ก่อนส่ง C&D = privileged (attorney-client) — แต่ถ้า share กับ third party (โฆษณา, agency) → waive
- `cease-desist` ใช้ work-product header เป็น default แต่ user ต้องระวังขั้นตอน clear/approve ภายใน

### Declaratory judgment risk

- ส่ง C&D ที่หนักเกินไป — counterparty มีสิทธิยื่น **declaratory judgment (DJ) action** ในศาลที่เลือกเอง → forum shopping → home court ของเราเสียไป
- Enforcement posture ใน `CLAUDE.md` กำหนด threshold — aggressive / measured / conservative → ปรับ tone ของ C&D ให้สอดคล้อง

### Patent claim drafting — out of scope

- `ip-legal` **ไม่ร่าง patent claim** — ระบุชัดใน README ว่า "Patent prosecution with claim strategy is a specialist craft that needs a patent agent or patent attorney"
- งาน patent ใน plugin นี้: FTO triage, IP clause review, portfolio tracking, infringement triage
- ใช้ patent agent / patent attorney จดทะเบียน USPTO สำหรับงาน prosecution

### OSS — copyleft virality

- **AGPL** — copyleft virality ครอบคลุม network use (SaaS = AGPL trigger) — ต่างจาก GPL ที่ trigger เฉพาะ distribution
- **GPL** — link ↔ aggregate distinction สำคัญ — link เป็น "derivative work"
- **Non-OSI licenses ที่ปลอมเป็น OSS** — Commons Clause, BSL, SSPL — ไม่ใช่ open source ตามนิยาม OSI แต่ marketing บอกว่าใช่ — `oss-review` flag

### Refusal pattern — ไม่ทำงานก่อน setup

- ถ้า `~/.claude/plugins/config/claude-for-legal/ip-legal/CLAUDE.md` มี `[PLACEHOLDER]` หรือ `[Your Company Name]` — ทุก skill **ปฏิเสธทำงาน** บอกให้รัน `/cold-start-interview` ก่อน
- เพราะ enforcement posture / approval matrix / practice area mix ต่างกันมาก — generic answer = wrong answer

---

## สรุปจุดที่ทำให้ `ip-legal` พิเศษ

1. **"Triage, not opinion" เป็น identity** — disclaimer ฮาร์ดโค้ดในทุก skill ที่เกี่ยวกับการประเมินความเสี่ยง — เป็นแบบอย่างที่ดีของ AI safety pattern
2. **Loud gates** — `cease-desist --send` มี gate enforcement posture; `takedown --send` มี Lenz + perjury gate; `takedown --counter` มี consent-to-federal-jurisdiction gate — ทุก gate บังคับ user confirm explicit
3. **Refuse without context** — `CLAUDE.md` เต็มทุก placeholder เป็นเงื่อนไขขั้นต่ำ; ไม่งั้น skill ปฏิเสธ
4. **Patent prosecution อยู่นอก scope โดยตั้งใจ** — plugin ที่กล้าบอก "เราไม่ทำงานนี้ ไปหา patent agent" — เป็น honesty ที่หา ได้ยาก
5. **`ip-renewal-watcher` agent** — proactive แต่ไม่เกินเลย — surface deadline แต่ไม่ file/ไม่จ่าย/ไม่ตัดสินใจ
6. **5 MCP server** ที่ครอบคลุม patent research (Solve Intelligence), case law (CourtListener, Descrybe), และ documents/chat (Drive, Slack)
7. **IP management system optional** — Anaqua / CPA Global / Alt Legal เป็น optional connector ไม่ bundle เพราะองค์กรเลือกใช้ต่างกัน

`ip-legal` คือตัวอย่างที่ดีของ **"plugin ที่รู้ว่าตัวเองรู้อะไรและไม่รู้อะไร"** — ในงาน IP ที่ผิดพลาดเล็กน้อยอาจหมายถึง treble damage, fee award, หรือ Rule 11 sanction — การ "ไม่กล้าสรุป" คือ feature ไม่ใช่ bug
