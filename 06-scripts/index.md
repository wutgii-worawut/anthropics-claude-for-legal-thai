---
title: 06 Scripts
nav_order: 7
has_children: true
permalink: /06-scripts/
---

# 06 Scripts — เครื่องมือ deploy, lint, และ validate

> *"Validate before you deploy. Lint before you ship. Test before you trust."*

หมวดนี้คือคำอธิบายของ **5 scripts** ใน `scripts/` ระดับ root ของ repository `anthropics/claude-for-legal` — เครื่องมือทั้งหมดที่ทำให้ **managed-agent cookbook** เปลี่ยนจาก YAML manifest ในไฟล์ → กลายเป็น agent ที่ deploy จริงบน Anthropic API ได้อย่างปลอดภัย

ถ้าหมวด `05-cookbooks` คือคำอธิบาย *"cookbook มีรูปทรงอย่างไร"* หมวดนี้คือคำอธิบาย *"แล้วเอามันไปใช้งานจริงอย่างไร และจะมั่นใจได้ยังไงว่าไม่มี security hole"*

## ทำไม scripts ถึงสำคัญในระดับ repo

Anthropic เลือกแยก **deploy logic** ออกจาก agent manifest อย่างชัดเจน:

- **`agent.yaml`** คือ **declarative manifest** — บอกว่า agent ต้องการอะไร (system prompt, tools, sub-agents, output schema)
- **`scripts/deploy-managed-agent.sh`** คือ **imperative resolver** — ทำให้ manifest กลายเป็น HTTP request ที่ส่งไปยัง `/v1/agents` ได้

การแยกแบบนี้ทำให้:

1. **มนุษย์อ่าน manifest ได้รู้เรื่อง** — ไม่ต้องเห็น `curl` หรือ `jq` ปนกับ business logic
2. **ทดสอบได้ดีโดยไม่ deploy จริง** — `--dry-run` ของ `deploy-managed-agent.sh` คือ pre-flight check
3. **CI/CD เข้ามาแทรกได้** — `test-cookbooks.sh` + `lint-tool-scope.py` คือชั้น verification ก่อนเข้า production

## scripts ทั้ง 5 ไฟล์

| ลำดับ | Script | หน้าที่หลัก | run โดย |
|---|---|---|---|
| 1 | [deploy-managed-agent](deploy-managed-agent.html) | deploy cookbook → `POST /v1/agents` พร้อม resolve `system.file`, `skills.path`, `callable_agents.manifest` | มนุษย์ / CI |
| 2 | [lint-tool-scope](lint-tool-scope.html) | **security linter** — กัน privilege escalation บน orchestrator (no MCP, no write, no Slack) | CI / pre-commit |
| 3 | [orchestrate](orchestrate.html) | **reference event loop** — closed-schema handoff orchestrator ระหว่าง managed agents | ตัวอย่าง (ไม่ใช่ production) |
| 4 | [test-cookbooks](test-cookbooks.html) | **dry-run validator** — assert ว่า resolved `POST` body ถูกต้อง (valid JSON, depth ≤ 1, มี system, ไม่มี `output_schema` leak) | CI |
| 5 | [validate](validate.html) | **JSON Schema validator** — เอาไว้ตรวจ output ของ reader subagent ก่อนส่งให้ orchestrator | runtime harness ระหว่าง subagent → orchestrator |

## ความสัมพันธ์ระหว่าง scripts

```text
                     ┌─────────────────────────┐
                     │  managed-agent-cookbooks│
                     │  / <slug> / agent.yaml  │
                     └────────────┬────────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
   ┌──────────────────┐  ┌─────────────────┐  ┌───────────────────┐
   │ lint-tool-scope  │  │  test-cookbooks │  │ deploy-managed-   │
   │  (security)      │  │  (--dry-run +   │  │  agent.sh         │
   │                  │  │   shape check)  │  │  (real deploy)    │
   └──────────────────┘  └────────┬────────┘  └─────────┬─────────┘
                                  │                     │
                                  ▼                     ▼
                          (PASS/FAIL ใน CI)    POST /v1/agents → API

   (runtime)
   ┌─────────────────────────────────────────┐
   │ orchestrate.py — ตัวอย่าง event loop      │
   │ ที่อ่าน handoff_request จาก orchestrator  │
   │ แล้ว steer subagent ผ่าน intent allowlist │
   └─────────────────────────────────────────┘

   (per subagent reader)
   ┌─────────────────────────────────────────┐
   │ validate.py — schema-check output.json  │
   │ ของ subagent ก่อน orchestrator consume   │
   └─────────────────────────────────────────┘
```

## วิธีอ่านที่แนะนำ

- ถ้าเป็น **deploy engineer / DevOps** → [deploy-managed-agent](deploy-managed-agent.html) → [test-cookbooks](test-cookbooks.html)
- ถ้าเป็น **security reviewer** → [lint-tool-scope](lint-tool-scope.html) → [orchestrate](orchestrate.html) (อ่าน security note ใน docstring)
- ถ้าเป็น **agent author / cookbook writer** → [validate](validate.html) → [test-cookbooks](test-cookbooks.html)
- ถ้าจะเข้าใจ **handoff pattern** ระหว่าง agents → [orchestrate](orchestrate.html) เป็น must-read

## หลักการที่อยู่เบื้องหลัง scripts ชุดนี้

จากการอ่าน script ทั้ง 5 ตัว เราพบหลักการสำคัญ 3 ข้อ:

1. **Closed schema beats free text.** `orchestrate.py` ไม่เคยให้ free-text ของ document ผ่าน LLM กลายเป็น steering prompt — ทุก handoff ต้องผ่าน enum ของ `intent` + JSON Schema ของ `params` ก่อน

2. **Defense in depth, but trust the gate.** `orchestrate.py` มี 5 ชั้นป้องกัน prompt injection แต่ docstring บอกตรง ๆ ว่าชั้นไหน "PRIMARY" (intent allowlist + target allowlist) และชั้นไหน "low assurance" (denylist regex) — เพื่อไม่ให้ reviewer หลงเชื่อว่า denylist กันได้จริง

3. **Lint at the YAML, not at the API.** `lint-tool-scope.py` ทำงานบน `agent.yaml` ไฟล์ — ก่อน HTTP request ออกจากเครื่อง เพราะ **policy ที่ deploy ไปแล้ว แก้ยากกว่า policy ที่ยังไม่ deploy**

หลักการชุดนี้คือสิ่งที่ทำให้ cookbooks ทั้ง 5 ตัว (reg-monitor, renewal-watcher, diligence-grid, launch-radar, docket-watcher) ปลอดภัยพอที่จะใช้กับ document จากภายนอก (regulatory filing, opposing-counsel motion, third-party diligence material) ที่อาจมี prompt-injection payload ฝังอยู่
