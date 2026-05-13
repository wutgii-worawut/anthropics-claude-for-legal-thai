---
title: MCP Connectors
parent: 08 Deployment
nav_order: 4
---

# ติดตั้งและตั้งค่า MCP Connectors

> "Connect a research tool first. Every plugin ships with legal research connectors already configured."
> — `README.md`

**MCP Connector** (Model Context Protocol) คือสะพานเชื่อมระหว่าง Claude กับระบบภายนอก — CLM, DMS, e-discovery, research database, productivity tool — ไม่มี connector = ไม่มีข้อมูลปัจจุบัน, citation ที่ตรวจสอบไม่ได้

หมวดนี้รวบรวม **connector ทั้งหมด** ที่ Claude for Legal รองรับ, plugin ที่ใช้, วิธีติดตั้ง และวิธี auth

## ทำไม connector สำคัญ

จาก `README.md`:

> *"Citations that come through a research connector are tagged with the source. Citations from model knowledge alone are flagged `[verify]`."*

ความต่าง:

| สถานการณ์ | Citation tag | ความน่าเชื่อถือ |
|----------|--------------|------------------|
| มี research connector (CourtListener, Westlaw, Descrybe) | tagged with source | ✓ ตรวจสอบได้ |
| ไม่มี research connector | `[verify]` | ✗ มาจาก training data |
| ไม่มี connector ใด ๆ | reviewer note: "sources not verified" | ✗ ทนายต้อง verify เองทุกจุด |

**กฎทอง**: เชื่อม **research connector อย่างน้อย 1 ตัว** ก่อนใช้งานจริง

## ตารางรวม Connector ทั้งหมด

### Research connectors (เน้น first)

| Connector | Auth | ใช้กับ plugin | หน้าที่ |
|-----------|------|---------------|---------|
| **CourtListener** | API key (optional, free) | `legal-clinic`, `ip-legal`, `litigation-legal`, `law-student` | federal dockets และ opinions (ฟรี, public) |
| **CoCounsel Legal** (Thomson Reuters) | OAuth | `cocounsel-legal` (external) | Westlaw Deep Research — caselaw, statutes, regulations, Practical Law, secondary sources ใน 3 jurisdictions ต่อ run |
| **Trellis** | Customer subscription | `litigation-legal` | state court dockets, motions |
| **Descrybe** | Customer subscription | `legal-clinic`, `ip-legal`, `law-student` | case law research และ summarization |
| **Solve Intelligence** | Customer subscription | `corporate-legal`, `ip-legal` | patent drafting และ prosecution |
| **Aurora** | Customer subscription | `litigation-legal` | clinic-style matter management และ calendaring |

### CLM (Contract Lifecycle Management)

| Connector | Auth | ใช้กับ plugin | หน้าที่ |
|-----------|------|---------------|---------|
| **Ironclad** | Customer subscription | `commercial-legal` | อ่าน contract register, renewal dates, clauses |
| **DocuSign / DocuSign CLM** | Customer subscription | `commercial-legal` | envelope status, executed contracts, CLM metadata |

### DMS (Document Management System)

| Connector | Auth | ใช้กับ plugin | หน้าที่ |
|-----------|------|---------------|---------|
| **iManage** | Customer subscription | `commercial-legal`, `corporate-legal` | อ่าน DMS — matter workspaces, document versions |
| **Box** | Tenant account | `corporate-legal` | อ่านไฟล์ใน VDR และ matter rooms |

### E-discovery

| Connector | Auth | ใช้กับ plugin | หน้าที่ |
|-----------|------|---------------|---------|
| **Everlaw** | Customer subscription | `litigation-legal` | e-discovery productions, tagged sets, chronologies |

### Drafting tools

| Connector | Auth | ใช้กับ plugin | หน้าที่ |
|-----------|------|---------------|---------|
| **Definely** | Customer subscription | `commercial-legal`, `corporate-legal` | in-document drafting, defined-terms checks |
| **Lawve AI** | Customer subscription | `legal-builder-hub` | contract review assist, clause libraries |

### Specialty

| Connector | Auth | ใช้กับ plugin | หน้าที่ |
|-----------|------|---------------|---------|
| **Courtroom5** | Customer subscription | `legal-clinic` | self-represented litigant workflow |
| **TopCounsel** | Customer subscription | `commercial-legal`, `corporate-legal`, `litigation-legal` | matter routing และ outside counsel panel |

### Productivity (ทุก plugin)

| Connector | Auth | ใช้กับ plugin | หน้าที่ |
|-----------|------|---------------|---------|
| **Slack** | OAuth | ทุก plugin (12) | อ่าน channels, search, ส่ง message, post canvases |
| **Google Drive** | OAuth | ทุก plugin (12) | อ่าน docs, sheets, slides; fetch by link |

### Issue tracker (product-legal)

| Connector | Auth | ใช้กับ plugin | หน้าที่ |
|-----------|------|---------------|---------|
| **Linear** | Workspace token | `product-legal` | launch tracker, issue tracking |
| **Atlassian (Jira)** | Workspace token | `product-legal` | launch tracker, issue tracking |
| **Asana** | Workspace token | `product-legal` | launch tracker, project tracking |

## วิธี install + auth (รายตัว)

### 1. ผ่าน Claude Cowork

- Settings → **Connectors**
- เลือก connector ที่ต้องการ
- คลิก **Connect** — OAuth popup จะเปิด browser
- ลงชื่อด้วย account ของระบบนั้น
- approve permission
- กลับมาที่ Cowork — connector active

> Cowork จัดการ token refresh, scope, และ revoke ให้อัตโนมัติ

### 2. ผ่าน Claude Code (`.mcp.json`)

แต่ละ plugin มี `.mcp.json` ของตัวเองที่กำหนด connector มาแล้ว — Claude Code จะ trigger OAuth ครั้งแรกที่ skill เรียกใช้

ตัวอย่าง `.mcp.json` ของ `commercial-legal`:

```json
{
  "mcpServers": {
    "Ironclad": {
      "type": "http",
      "url": "https://mcp.na1.ironcladapp.com/mcp",
      "title": "Ironclad",
      "description": "Search your contract repository..."
    },
    "DocuSign": {
      "type": "http",
      "url": "https://mcp.docusign.com/mcp",
      "title": "DocuSign",
      "description": "Envelope status, executed contracts..."
    },
    "Slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "title": "Slack"
    },
    "Google Drive": {
      "type": "http",
      "url": "https://drivemcp.googleapis.com/mcp/v1",
      "title": "Google Drive"
    }
  },
  "recommendedCategories": [
    "contract-management",
    "e-signature",
    "legal-document-management"
  ]
}
```

ใน Claude Code, เพิ่ม / แก้ไข connector ผ่าน:

```bash
claude mcp add <connector-name> <url> --type http
claude mcp list
claude mcp remove <connector-name>
```

### 3. ผ่าน Managed Agents (headless)

ใน `agent.yaml`:

```yaml
tools:
  - "mcp__ironclad__*"
  - "mcp__slack__send_message"
  - "mcp__google_drive__read"
```

Auth ผ่าน **service account** หรือ **API key** ใน environment variable:

```bash
export IRONCLAD_API_KEY=...
export SLACK_BOT_TOKEN=xoxb-...
```

> ห้าม commit secret ลง git — ใช้ secret manager (AWS Secrets Manager, GCP Secret Manager, Vault)

## Auth flow ตาม connector

| Auth method | Connector ที่ใช้ |
|-------------|-------------------|
| **OAuth** (interactive) | Slack, Google Drive, CoCounsel Legal, Box, DocuSign, Linear, Jira, Asana |
| **API key** | CourtListener (optional), Ironclad, iManage, Everlaw, Trellis, Descrybe |
| **Customer SSO** | Aurora, Solve Intelligence, TopCounsel, Definely, Lawve AI, Courtroom5 |

## ตรวจสอบว่า connector ทำงานหรือไม่

### ใน Cowork

Settings → Connectors → ดู status เป็น **Connected** สีเขียว

### ใน Claude Code

```bash
claude mcp list
```

จะแสดงรายการ connector ทั้งหมดที่ active สำหรับ user/project ปัจจุบัน

ลอง trigger skill ที่ต้องใช้ connector — ถ้า skill ทำงานปกติ + citation มี source tag → connector ทำงาน

ถ้า skill ขึ้น `connector unavailable` → ตรวจ:
- Token หมดอายุ? → re-auth
- URL ถูกต้อง? → ดู `.mcp.json`
- Subscription หมดอายุ? → ติดต่อ vendor

### ใน Managed Agents

```bash
curl -H "x-api-key: $ANTHROPIC_API_KEY" \
  https://api.anthropic.com/v1/agents/<agent_id>/connectors
```

จะคืน status ของแต่ละ connector ที่ agent มีสิทธิ์ใช้

## Connector ที่ยังไม่มี (Wanted)

จาก `CONNECTORS.md` — Anthropic เปิดรับ connector ใหม่:

- **IP management** — Anaqua, Clarivate IPfolio, AppColl, Patrix, Alt Legal, FoundationIP
- **USPTO by customer number** — full portfolio status + deadlines
- **USPTO TSDR** — trademark status
- **Thomson Reuters** — CoCounsel, Practical Law, Westlaw (research + drafting ทุก plugin)
- **SS&C Intralinks / Datasite** — VDR สำหรับ diligence
- **Relativity / Everlaw beyond read** — e-discovery workflow (write)
- **State bar CLE trackers** — สำหรับ bar prep
- **Court e-filing** — PACER write, state e-filing (พร้อม hard irreversibility gate)
- **Global AI Regulation Tracker** — multi-jurisdiction AI reg tracking
- **Regulatory primary sources** — eCFR, Federal Register, EUR-Lex, legislation.gov.uk, Federal Register of Legislation AU, Singapore Statutes Online

ถ้าทีมท่านสร้าง MCP server ที่ตอบโจทย์เหล่านี้ — ส่ง PR ไปที่ repo เลย ดู [09-development](../09-development/index.html) สำหรับ contribution guideline

## Best practice

1. **เชื่อม research connector ก่อนเสมอ** — CourtListener ฟรี, เริ่มจากตัวนี้
2. **ใช้ user scope (Claude Code)** — connector ที่ install แบบ user ใช้ได้จากทุก project
3. **อย่า commit `.mcp.json` ที่มี secret** — ใช้ `.env` หรือ secret manager
4. **เช็ค `recommendedCategories`** — ดูว่า plugin ต้องการ connector ประเภทใด
5. **OAuth ครั้งเดียว, refresh อัตโนมัติ** — ไม่ต้อง auth ทุกครั้ง
6. **ทดสอบ connector ก่อน production** — รัน skill ง่าย ๆ ดู citation tag

## ความเข้าใจผิดที่พบบ่อย

| ปัญหา | สาเหตุ | แก้ไข |
|------|--------|------|
| Connector connect แล้วแต่ไม่มี data | scope ของ token ไม่ครอบคลุม | re-auth พร้อม scope ที่กว้างขึ้น |
| Citation ยัง `[verify]` ทั้งที่เชื่อม connector แล้ว | research connector ไม่ใช่ตัวที่ guardrail รู้จัก | เชื่อม CourtListener / Trellis / Descrybe / Solve เพิ่ม |
| `mcp__ironclad__*` permission denied ใน Managed Agents | ไม่มี API key ใน env | export `IRONCLAD_API_KEY` |
| Slack ส่ง message ไม่ได้ | ไม่ได้ approve write scope | re-auth, approve `chat:write` |

## สรุป MCP Connectors

Connector = **ตัวเชื่อม** ระหว่าง Claude กับระบบจริงของลูกค้า — research, CLM, DMS, e-discovery, productivity ต้องเชื่อม **research connector ก่อนเสมอ** เพื่อให้ citation ตรวจสอบได้

จบหมวด deployment — ขั้นต่อไป ถ้าต้องการ **เพิ่ม skill หรือ plugin ของตัวเอง** → ไปที่ [09-development](../09-development/index.html)
