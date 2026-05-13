---
title: marketplace.json — ทะเบียน plugin
parent: 02 โครงสร้างโฟลเดอร์
nav_order: 2
---

# `.claude-plugin/marketplace.json` — ทะเบียน plugin ระดับ repo

> *"`/plugin marketplace add ...` → drag the unzipped folder onto the terminal window…"*
> — `QUICKSTART.md`

ไฟล์ `.claude-plugin/marketplace.json` คือ **ทะเบียน plugin ระดับ repository** ที่ Claude Code อ่านเพื่อรู้ว่า repo นี้มี plugin อะไรบ้าง แต่ละตัวอยู่ที่ไหน ใครเขียน และทำอะไร

ถ้าไม่มีไฟล์นี้ Claude Code จะไม่รู้จัก repo เลย — มันไม่ใช่ "เผลอใส่ไว้" แต่เป็น **API contract** ระหว่าง repo กับ Claude Code runtime

## ตำแหน่งของไฟล์

```text
claude-for-legal/
└── .claude-plugin/
    └── marketplace.json    ← ไฟล์นี้
```

โฟลเดอร์ `.claude-plugin/` ขึ้นต้นด้วย dot — โดย convention ของ Claude Code, **dotted folder = metadata directory** ที่ runtime ใช้

## JSON schema เบื้องต้น

โครงไฟล์เป็นแบบนี้:

```json
{
  "name": "claude-for-legal",
  "owner": {
    "name": "Anthropic"
  },
  "plugins": [
    {
      "name": "commercial-legal",
      "source": "./commercial-legal",
      "description": "Reviews vendor agreements, NDAs...",
      "author": {
        "name": "Anthropic"
      }
    },
    {
      "name": "cocounsel-legal",
      "source": "./external_plugins/cocounsel-legal",
      "description": "CoCounsel Legal delivers comprehensive Westlaw Deep Research reports...",
      "author": {
        "name": "Thomson Reuters"
      }
    }
    // ... รวม 13 plugin
  ]
}
```

## คำอธิบายแต่ละฟิลด์

### Top-level

| ฟิลด์ | type | required | คำอธิบาย |
|---|---|---|---|
| `name` | string | yes | ชื่อ marketplace — ผู้ใช้จะเห็นเวลา `/plugin install <plugin>@claude-for-legal` |
| `owner.name` | string | yes | เจ้าของ marketplace (ไม่ใช่ของ plugin แต่ละตัว) |
| `plugins` | array | yes | array ของ plugin entry |

### Plugin entry

| ฟิลด์ | type | required | คำอธิบาย |
|---|---|---|---|
| `name` | string | yes | ชื่อ plugin — ใช้เป็น namespace ของ skill (เช่น `/commercial-legal:review`) |
| `source` | string (path) | yes | relative path ไปยังโฟลเดอร์ plugin — ที่นั่นต้องมี `.claude-plugin/plugin.json` |
| `description` | string | yes | คำอธิบายหนึ่งย่อหน้า — แสดงเวลา `/plugin marketplace list` |
| `author.name` | string | yes | ชื่อผู้เขียน plugin (ต่างจาก `owner` ของ marketplace) |

## ทำไมต้องมี marketplace registry?

### ปัญหาที่แก้

ถ้าไม่มี registry ที่รวมศูนย์ Claude Code ก็ต้อง:

- เดาว่า plugin ใดอยู่ในโฟลเดอร์ใด → ใช้ heuristic เช่น "หา `.claude-plugin/plugin.json` ทุกระดับ"
- ไม่รู้ว่า plugin ใดเป็น "officially recommended" จาก repo owner กับตัวที่เป็น "external/experimental"
- เปิดช่องให้ใส่ malicious plugin ลึกในโฟลเดอร์โดยที่ user ไม่รู้

### ประโยชน์ที่ได้

| ประโยชน์ | คำอธิบาย |
|---|---|
| **Discovery** | `/plugin marketplace list` แสดงทุก plugin ที่ available — ไม่ต้องเดา |
| **Authorship clarity** | ฟิลด์ `author.name` แยก plugin ของ Anthropic vs ของ vendor (Thomson Reuters) ชัดเจน |
| **Path normalization** | `source: "./commercial-legal"` ทำให้ refactor / ย้ายโฟลเดอร์ได้ในที่เดียว |
| **Security boundary** | Claude Code ติดตั้งเฉพาะที่ register ไว้ — ไม่ใช่สแกนทั้ง disk |

## In-tree plugins vs external_plugins

จุดที่น่าสนใจของ `marketplace.json` ของ `claude-for-legal` คือมี **2 ประเภท** ของ plugin source:

### 1. In-tree plugins (12 ตัว)

```text
"source": "./commercial-legal"
"source": "./privacy-legal"
"source": "./litigation-legal"
// ... รวม 12 ตัว
```

| คุณสมบัติ | ค่า |
|---|---|
| ตำแหน่ง | โฟลเดอร์ระดับเดียวกับ root |
| Author | Anthropic |
| Maintained by | Anthropic team |
| Update cadence | ตามที่ Anthropic release |

### 2. External plugins (1 ตัว)

```text
"source": "./external_plugins/cocounsel-legal"
```

| คุณสมบัติ | ค่า |
|---|---|
| ตำแหน่ง | ใต้ `external_plugins/` |
| Author | Thomson Reuters |
| Maintained by | Vendor (Anthropic merge แต่ไม่ดูแล code) |
| ที่มา | PR จาก vendor; ผ่าน security review ของ Anthropic |

### ทำไมต้องแยก?

```text
claude-for-legal/
├── commercial-legal/          ← Anthropic เขียน, dogfood, รับผิดชอบเอง
├── privacy-legal/             ← เหมือนกัน
├── ...
└── external_plugins/
    └── cocounsel-legal/       ← Thomson Reuters เขียน, Anthropic แค่ host
```

**เหตุผล**:

1. **Authorship signal** — user เห็น `external_plugins/` แล้วรู้ว่าไม่ใช่ของ Anthropic ตรง ๆ
2. **Code review boundary** — PR ที่แตะ `external_plugins/` ต้องผ่าน vendor agreement ก่อน
3. **License independence** — vendor ใช้ license ของตนเองได้ถ้าจำเป็น
4. **MCP scope** — `cocounsel-legal` ต่อ Westlaw API (commercial); ไม่ใช่ free tool ของ Anthropic

ใน [README ของ `managed-agent-cookbooks/`](../managed-agent-cookbooks/) มีพูดถึงเรื่องนี้:

> *"Every agent in this repo ships two ways: as a Claude Code plugin you install today (see the vertical directories at repo root), and as a Claude Managed Agent template…"*

→ ดังนั้น `marketplace.json` ครอบคลุมเฉพาะ **plugin form** เท่านั้น; managed-agent cookbooks ไม่ถูกลง register ในไฟล์นี้

## ความสัมพันธ์ระหว่างไฟล์

```text
.claude-plugin/marketplace.json
└── plugins[i].source → <plugin-dir>/
                        └── .claude-plugin/plugin.json
                              ├── name      ← ต้องตรงกับ plugins[i].name
                              ├── version
                              ├── description
                              └── author
```

**กฎ**: `marketplace.json` → `plugins[i].name` **ต้องเท่ากับ** `<plugin-dir>/.claude-plugin/plugin.json` → `name`

ถ้าไม่ตรงกัน `scripts/validate.py` จะ fail และ CI จะ block PR

## ตัวอย่าง marketplace entry แบบเต็ม

ตัด snippet จาก `marketplace.json` ของจริง:

```json
{
  "name": "litigation-legal",
  "source": "./litigation-legal",
  "description": "Manages the litigation portfolio — matters, deadlines, holds, demands, outside counsel — and does the work: claim charts (patent and civil), chronologies, depo prep, privilege logs, brief drafting. Adapts to how you work litigation: in-house, firm, or solo.",
  "author": {
    "name": "Anthropic"
  }
}
```

สังเกตว่า `description` เป็น **ประโยคทำงาน** ไม่ใช่ noun phrase — สไตล์ของ Anthropic ใช้ verb-led description เพื่อบอก user ตรง ๆ ว่า plugin ทำอะไร ไม่ใช่ "เป็นอะไร"

## วิธีติดตั้ง plugin จาก marketplace นี้

ผู้ใช้ run:

```bash
# 1. Add marketplace (one-time, จะลงทะเบียน path ของ repo)
/plugin marketplace add /path/to/claude-for-legal

# 2. Install plugin
/plugin install commercial-legal@claude-for-legal

# 3. Restart Claude Code (สำคัญ — ระบบจะไม่ load จนกว่าจะ restart)
```

ภายใต้ฮูด: Claude Code อ่าน `.claude-plugin/marketplace.json` → หา entry ชื่อ `commercial-legal` → ไปอ่าน `<source>/.claude-plugin/plugin.json` → load `skills/`, `agents/`, `hooks/`, `CLAUDE.md`

## ข้อจำกัด

- Plugin ที่ไม่อยู่ใน `marketplace.json` → user ติดตั้งไม่ได้แม้จะมีโฟลเดอร์อยู่ใน repo
- เปลี่ยน `name` ของ plugin → ผู้ใช้เดิมต้อง uninstall + reinstall (commands กลายเป็น namespace ใหม่)
- `source` path ต้องเป็น relative — ห้ามใช้ `https://` URL (ต่างจาก npm registry)

## สรุป

`marketplace.json` คือ **manifest ระดับ repo** ที่:

- บอก Claude Code ว่าใน repo นี้มี plugin อะไรบ้าง
- แยกแยะ in-tree (Anthropic) กับ external (vendor) ชัดเจน
- เป็น contract ที่ผูกกับ `plugin.json` ของแต่ละ plugin
- ทำให้ user ติดตั้งและ discover ได้ง่าย ๆ ผ่าน `/plugin install`

หน้าถัดไป → [plugin-anatomy](plugin-anatomy.html) จะลงลึกที่ `<plugin>/.claude-plugin/plugin.json` และโครงสร้างภายในของ plugin แต่ละตัว
