---
title: Validation
parent: 09 Development
nav_order: 2
---

# Validation — `validate.py` และ `test-cookbooks.sh`

> "Run the validators. `scripts/validate.py` and `scripts/lint-tool-scope.py` check the structural invariants the plugin loader depends on."
> — `CONTRIBUTING.md`

Claude for Legal **ไม่ใช้ unit test** สำหรับเนื้อหา skill (เพราะ output เป็น judgment ของทนายที่ test ไม่ได้) แต่ใช้ **structural validation** เพื่อตรวจว่า:

- Plugin loader อ่าน plugin ได้
- Skill register เป็น slash command ได้ถูกต้อง
- Cookbook deploy ได้ไม่มี error
- Tool scope ใน Managed Agents ไม่ over-permissioned

หมวดนี้สอนวิธีใช้, JSON schema, และวิธี debug

## Scripts ที่ Anthropic provide

| Script | จุดประสงค์ |
|--------|-------------|
| `scripts/validate.py` | ตรวจ structural invariant ของ plugin, skill, frontmatter |
| `scripts/lint-tool-scope.py` | ตรวจ tool permission ใน Managed Agent — ไม่ให้ over-scoped |
| `scripts/test-cookbooks.sh` | Dry-run ทุก cookbook + lint tool scope |
| `scripts/deploy-managed-agent.sh` | Deploy cookbook (production) |
| `scripts/orchestrate.py` | Reference event loop สำหรับ steering agents |

## `validate.py` — ตรวจ structural invariant

### รันสำหรับ plugin ทั้งหมด

```bash
python scripts/validate.py
```

จะวน loop ผ่านทุก plugin ใน repo แล้วตรวจ:

1. **`<plugin>/.claude-plugin/plugin.json`** — มีอยู่และ format ถูก
2. **`<plugin>/CLAUDE.md`** — มีอยู่ (practice profile template)
3. **`<plugin>/README.md`** — มีอยู่
4. **`<plugin>/skills/*/SKILL.md`** — แต่ละ skill มี SKILL.md
5. **Frontmatter format** — `name`, `description`, optional `argument-hint`, `user-invocable`
6. **Marketplace registration** — plugin ต้องอยู่ใน `.claude-plugin/marketplace.json`

### รันสำหรับ plugin เดียว

```bash
python scripts/validate.py commercial-legal
```

### JSON Schema ที่ใช้ตรวจ

ทุก plugin มี `plugin.json` ตาม schema นี้:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "version", "description", "author"],
  "properties": {
    "name": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9-]*$"
    },
    "version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$"
    },
    "description": {
      "type": "string",
      "maxLength": 500
    },
    "author": {
      "type": "object",
      "required": ["name"],
      "properties": {
        "name": { "type": "string" }
      }
    }
  }
}
```

Skill frontmatter schema:

```yaml
required: [name, description]
properties:
  name:
    type: string
    pattern: "^[a-z][a-z0-9-]*$"
  description:
    type: string
    maxLength: 1024          # important — trigger signal
  argument-hint:
    type: string
  user-invocable:
    type: boolean
    default: true
```

### Error ที่พบบ่อย

| Error | สาเหตุ | วิธีแก้ |
|-------|--------|---------|
| `plugin.json missing` | ลืมสร้างไฟล์ | สร้าง `<plugin>/.claude-plugin/plugin.json` |
| `name does not match folder` | ตั้งชื่อ folder ไม่ตรง `name:` | rename folder หรือ frontmatter ให้ตรง |
| `description too long (>1024 chars)` | trigger signal ยาวเกิน | ตัดประโยคให้กระชับ |
| `version not semver` | ใส่ version เป็น "v1" หรือ "1.0" | ใช้ `1.0.0` format |
| `skill not in plugin folder` | สร้าง skill ผิด path | ย้ายไปที่ `<plugin>/skills/<name>/` |

## `lint-tool-scope.py` — ตรวจ Managed Agent tool scope

### หน้าที่

ป้องกัน **privilege escalation** ใน Managed Agents โดยตรวจว่า:

1. Reader agent มี `tools:` แค่ `Read`, `Grep` (ไม่มี Write, ไม่มี MCP write tool)
2. Analyzer agent มี MCP read tool แต่ **ไม่มี** Write
3. Writer agent มี Write แต่ **ไม่ได้** สัมผัส raw documents
4. ไม่มี wildcard `mcp__*` (ต้อง scope ชัดเจน)

### รัน

```bash
python scripts/lint-tool-scope.py managed-agent-cookbooks/renewal-watcher/agent.yaml
```

หรือทุก cookbook:

```bash
python scripts/lint-tool-scope.py managed-agent-cookbooks/
```

### ตัวอย่าง output

```
[PASS] managed-agent-cookbooks/renewal-watcher/agent.yaml
  - tier: orchestrator
  - tools: 4 (Read, Write, mcp__ironclad__*, mcp__slack__send_message)
  - subagents: 1 (alert-writer.yaml)

[FAIL] managed-agent-cookbooks/renewal-watcher/subagents/repo-reader.yaml
  - tier: reader
  - violation: reader tier cannot have Write tool
  - violation: reader tier should not have mcp__slack__send_message
```

### Tool scope rule

| Tier | Allowed tools | Forbidden |
|------|---------------|-----------|
| **Reader** | `Read`, `Grep`, `mcp__<connector>__search`, `mcp__<connector>__fetch` | `Write`, `Edit`, `mcp__*__write`, `mcp__*__send` |
| **Analyzer** | Tools ของ reader + ทำ computation ใน head | `Write`, `Edit` |
| **Writer** | `Write`, `Edit`, `mcp__slack__send_message`, `mcp__email__send` | direct file read จาก raw source (ต้องรับ data ผ่าน structured handoff) |

> เหตุผล: ถ้า writer ได้รับ raw document → prompt injection ใน document สามารถ trigger writer ไปทำ action เกิน scope

## `test-cookbooks.sh` — Dry-run ทุก cookbook

### รัน

```bash
bash scripts/test-cookbooks.sh
```

### ทำอะไรบ้าง

1. **Iterate** ทุก folder ใน `managed-agent-cookbooks/`
2. **Validate** `agent.yaml` schema (ใช้ `validate.py`)
3. **Lint tool scope** (`lint-tool-scope.py`)
4. **Resolve references** — ตรวจว่า `system.file:` และ `skills.path:` ชี้ไปยังไฟล์ที่มีจริง
5. **Dry-run deploy** — ทำทุกอย่างยกเว้น `POST /v1/agents`
6. **Print summary** — pass/fail count

### Output ตัวอย่าง

```
[1/5] managed-agent-cookbooks/diligence-grid
  - agent.yaml: valid
  - tool scope: pass
  - references: 3/3 resolved
  - dry-run: success

[2/5] managed-agent-cookbooks/docket-watcher
  - agent.yaml: valid
  - tool scope: pass
  - references: 2/2 resolved
  - dry-run: success

[3/5] managed-agent-cookbooks/launch-radar
  - agent.yaml: valid
  - tool scope: FAIL — writer subagent has Read tool
  - SKIPPED

...

Total: 4/5 pass
```

### ใช้เมื่อไร

- ก่อน push PR ที่แก้ cookbook ใด ๆ
- ก่อน deploy production
- หลัง update SKILL.md ที่ cookbook reference ถึง

## วิธี debug

### Error: `plugin.json schema validation failed`

```bash
# ดูรายละเอียด error
python scripts/validate.py commercial-legal --verbose

# ตรวจ JSON syntax
python -m json.tool commercial-legal/.claude-plugin/plugin.json
```

### Error: `SKILL.md frontmatter not valid YAML`

```bash
# ใช้ yaml parser ตรวจ
python -c "import yaml; print(yaml.safe_load(open('<path>').read().split('---')[1]))"
```

### Error: `reference file not found`

```bash
# ตรวจว่า reference path ใน agent.yaml ชี้ถูก
yq '.system.file' managed-agent-cookbooks/<slug>/agent.yaml
ls -l managed-agent-cookbooks/<slug>/../../$(yq -r '.system.file' managed-agent-cookbooks/<slug>/agent.yaml)
```

### Error: `tool scope violation`

อ่าน output ของ `lint-tool-scope.py` — จะบอกว่า tier ใด มี tool ใดที่ไม่อนุญาต — แก้ไข `tools:` ใน yaml ของ subagent นั้น

## Continuous validation

แนะนำให้ใส่ใน git hook หรือ CI:

```bash
# .git/hooks/pre-commit
#!/usr/bin/env bash
set -e
python scripts/validate.py
python scripts/lint-tool-scope.py managed-agent-cookbooks/
```

หรือใน GitHub Actions:

```yaml
name: Validate
on: [push, pull_request]
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: python scripts/validate.py
      - run: python scripts/lint-tool-scope.py managed-agent-cookbooks/
      - run: bash scripts/test-cookbooks.sh
```

## ไม่มีอะไรที่ validation ตรวจ

สำคัญที่ต้องเข้าใจ — `validate.py` **ไม่** ตรวจ:

| สิ่งที่ไม่ตรวจ | ทำไมไม่ตรวจ |
|---------------|------------|
| Skill ทำงานถูกต้องตามที่บอกใน description | ต้องใช้ทนายทดสอบ — ไม่ใช่งานของ validator |
| Citation มาจาก source ที่ถูกต้อง | research connector ทำหน้าที่นี้ |
| Output format ตรงกับ spec | ผ่าน `/legal-builder-hub:skills-qa` |
| Practice profile กรอกครบ | runtime check ของ skill เอง |
| Doctrine ใน skill ถูกต้อง | peer review จากทนาย |

> **Validation = ตรวจ structure ไม่ใช่ semantics**

สำหรับ semantics — ใช้ **community skill QA framework** (`/legal-builder-hub:skills-qa`) ที่ตรวจ 9 design parameter

## Skill QA Framework (legal-builder-hub)

แม้จะไม่ใช่ technical validator — `/legal-builder-hub:skills-qa` คือเครื่องมือ **design review** สำหรับ skill ใหม่ ตรวจ:

1. Clarity of purpose
2. Scope appropriateness
3. Safety & guardrails
4. Citation discipline
5. Destination awareness
6. Privilege handling
7. MCP scope (อะไร skill touch ได้)
8. File-write targets
9. Injection resistance

รันก่อนส่ง PR:

```bash
/legal-builder-hub:skills-qa <plugin>/skills/<my-skill>/
```

หาก fail — แก้ก่อน push

## สรุป Validation

| Tool | ตรวจอะไร | รันเมื่อไร |
|------|---------|------------|
| `validate.py` | structural invariant ของ plugin/skill | ก่อนทุก commit |
| `lint-tool-scope.py` | tool permission ใน Managed Agent | ก่อนแก้ cookbook |
| `test-cookbooks.sh` | dry-run cookbook ทั้งหมด | ก่อน deploy production |
| `/legal-builder-hub:skills-qa` | design quality ของ skill | ก่อนส่ง PR |

ขั้นต่อไป — เข้าใจ security model ที่อยู่เบื้องหลัง: [security-model](security-model.html)
