---
title: lint-tool-scope
parent: 06 Scripts
nav_order: 2
---

# lint-tool-scope.py — security linter กัน privilege escalation

> *"Assert orchestrator `agent.yaml` files ship with scoped tool configs."*
> — docstring บรรทัดแรก

`scripts/lint-tool-scope.py` คือ **security gate** ที่สำคัญที่สุดในชุด scripts ทั้ง 5 ตัว เพราะมันคือชั้นที่กัน *privilege escalation* ของ orchestrator ก่อน deploy จริง

หมายเหตุ: ไฟล์นี้แม้จะสั้น (107 บรรทัด) แต่ encode policy ที่ Anthropic ใช้กับทุก cookbook ของ Managed Agents ออกแบบมาเฉพาะ — **อ่านเข้าใจไฟล์นี้ = เข้าใจ threat model ของระบบ managed-agent ทั้งหมด**

## จุดประสงค์

`lint-tool-scope.py` วนทุก `managed-agent-cookbooks/*/agent.yaml` (orchestrator manifest) และตรวจ block `tools:` ของ orchestrator ว่ามี **3 ข้อห้าม** ดังนี้:

1. **ห้าม `mcp_toolset`** บน orchestrator — MCP client (เช่น Slack, Google Drive, Notion) ต้องอยู่บน **subagent leaf** ไม่ใช่ parent
2. **ห้าม `write` enabled** ใน `agent_toolset*` — มีแค่ **writer leaf** ตัวเดียวเท่านั้นที่ถือ Write tool
3. **ห้าม `slack_send_message`** หรือ `slack_*` ใด ๆ — orchestrator ต้อง emit `handoff_request` แทน

**Exit code**:
- `0` = ทุก cookbook ผ่าน + print one-line summary ต่อ cookbook
- `1` = มี violation + รายชื่อไฟล์ + tool ที่ผิดบน stderr
- `2` = ไม่มีโฟลเดอร์ `managed-agent-cookbooks/`

## ทำไม 3 ข้อห้ามนี้

หลักการเบื้องหลังคือ **least-privilege orchestrator pattern**:

```text
       ┌─────────────────────────────┐
       │      Orchestrator (root)    │
       │  - อ่าน document            │
       │  - สรุปข้อมูล                │
       │  - emit handoff_request     │
       │  - ❌ ไม่มี write/MCP/Slack  │
       └──────────────┬──────────────┘
                      │ handoff via closed-schema intent
                      ▼
       ┌─────────────────────────────┐
       │  Subagent leaves            │
       │  - writer-leaf: ✅ Write     │
       │  - slack-leaf: ✅ Slack MCP  │
       │  - drive-leaf: ✅ Drive MCP  │
       └─────────────────────────────┘
```

**ถ้า orchestrator มี Slack tool** = document ที่มี prompt injection สามารถสั่ง orchestrator ส่ง Slack ตรง ๆ ได้ (single-hop attack)

**ถ้า orchestrator ไม่มี Slack** = attacker ต้องผ่าน closed-schema handoff (`orchestrate.py`) ก่อนถึงจะใช้ Slack ได้ ซึ่งโดน intent allowlist + schema validation กั้น

## Algorithm — pseudocode ภาษาไทย

```text
for each agent.yaml ใน managed-agent-cookbooks/*/:
    doc = yaml.safe_load(agent.yaml)
    tools = doc.get("tools") or []

    for each entry in tools:
        ttype = entry.get("type", "")

        # rule 1: ห้าม mcp_toolset
        if ttype == "mcp_toolset":
            error: "orchestrator must not carry mcp_toolset; move to leaf"
            continue

        if not ttype.startswith("agent_toolset"):
            continue

        # rule 2 + 3: inspect per-tool configs
        default_enabled = entry.default_config.enabled (default False)
        for each cfg in entry.configs:
            name = cfg.name
            enabled = cfg.enabled or default_enabled
            if enabled and name == "write":
                error: "orchestrator must not enable 'write'"
            if enabled and name starts with "slack":
                error: "orchestrator must not enable Slack tool"

        # rule 4 (implicit): ห้าม default_config.enabled = true
        if default_enabled:
            error: "orchestrator agent_toolset must have default_config.enabled=false"
```

จุดที่ subtle: **`default_config.enabled=true` โดน reject ทันที** เพราะถ้า default เป็น on, ทุก tool ใน toolset จะ enabled — รวม write + Slack — โดย script ไม่สามารถ enumerate toolset ทั้งหมด (รายการอยู่ฝั่ง server) ได้ ดังนั้น **fail closed** เป็น policy

## Code snippet สำคัญ

```python
ttype = entry.get("type", "")
if ttype == "mcp_toolset":
    name = entry.get("mcp_server_name", "<unnamed>")
    errs.append(
        f"{path}: orchestrator must not carry mcp_toolset "
        f"(mcp_server_name={name}); move to the subagent leaf"
    )
    continue
if not ttype.startswith("agent_toolset"):
    continue
# Inspect per-tool configs.
default_cfg = entry.get("default_config") or {}
default_enabled = bool(default_cfg.get("enabled", False))
configs = entry.get("configs") or []
for cfg in configs:
    if not isinstance(cfg, dict):
        continue
    name = cfg.get("name")
    enabled = bool(cfg.get("enabled", default_enabled))
    if enabled and name == "write":
        errs.append(
            f"{path}: orchestrator must not enable 'write'; "
            f"only the writer leaf holds Write"
        )
    if enabled and isinstance(name, str) and name.startswith("slack"):
        errs.append(
            f"{path}: orchestrator must not enable Slack tool '{name}'; "
            f"emit a handoff_request instead"
        )
```

สังเกตว่า:

- การตรวจ `name.startswith("slack")` ใช้ **prefix match** — ดักทั้ง `slack_send_message`, `slack_upload_file`, `slack_*` ที่อาจจะออกใหม่ในอนาคต
- การตรวจ `name == "write"` ใช้ **exact match** — เพราะ `write` คือ tool standard ที่ Claude API ใช้ ไม่มี variant
- ข้อความ error อธิบาย **"ทำไม"** ทุกครั้ง — เช่น `"only the writer leaf holds Write"`, `"emit a handoff_request instead"` — ทำให้ developer แก้ได้เร็ว

## ตัวอย่างใช้งาน

### Lint manual

```bash
cd /path/to/claude-for-legal
python3 scripts/lint-tool-scope.py
```

**Output (clean)**:

```
  ✓ diligence-grid           orchestrator tool scope clean
  ✓ docket-watcher           orchestrator tool scope clean
  ✓ launch-radar             orchestrator tool scope clean
  ✓ reg-monitor              orchestrator tool scope clean
  ✓ renewal-watcher          orchestrator tool scope clean
```

### Output ตอน fail

สมมติว่าเขียน orchestrator ใหม่และเผลอใส่ Slack tool:

```yaml
# managed-agent-cookbooks/my-new-cookbook/agent.yaml
tools:
  - type: agent_toolset_slack
    configs:
      - name: slack_send_message
        enabled: true
```

linter จะ print:

```
tool-scope lint FAILED:
  managed-agent-cookbooks/my-new-cookbook/agent.yaml: orchestrator must not enable Slack tool 'slack_send_message'; emit a handoff_request instead
```

และ exit 1

## กรณีใช้งานจริง

| สถานการณ์ | วิธีใช้ |
|---|---|
| **Pre-commit hook** | `python3 scripts/lint-tool-scope.py` ใน `.git/hooks/pre-commit` — กัน commit cookbook ที่ผิด policy |
| **CI workflow** | step ใน GitHub Actions ก่อน deploy job — fail PR ทันที |
| **Author cookbook ใหม่** | run หลังเขียน `agent.yaml` ก่อน push |
| **Audit periodic** | run รายเดือนเพื่อตรวจว่ามีใครแก้ orchestrator แล้ว push ผ่าน gate ได้ไหม |
| **Migration** | ก่อนเปลี่ยน leaf เป็น orchestrator — ตรวจว่าตัด tool ที่ไม่ควรมีออกหมดแล้ว |

`scripts/test-cookbooks.sh` (ดู [test-cookbooks](test-cookbooks.html)) call `lint-tool-scope.py` เป็น step แรก = ถ้า lint fail, dry-run ก็ไม่ run ต่อ = **lint คือ gate แรกใน pipeline**

## หลักการที่ encode ในไฟล์นี้

1. **Privilege segregation** — write/MCP/external-channel access ต้องอยู่บน leaf ไม่ใช่ root
2. **Closed-schema handoff** — orchestrator สื่อสารกับ subagent ผ่าน `handoff_request` schema เท่านั้น ไม่ใช่ direct tool call
3. **Default deny** — ห้าม `default_config.enabled=true` เพราะ script enumerate toolset ทั้งหมดไม่ได้
4. **Fail fast, fail loud** — exit code 1 + ข้อความระบุ path + tool + reason

ถ้าเข้าใจไฟล์นี้แล้ว ให้อ่านต่อ [orchestrate.py](orchestrate.html) — มันคือ **runtime counterpart** ของ lint ตัวนี้ (deploy-time check vs runtime enforcement)

## Dependencies

- `python3` + `pyyaml`

ไม่ต้องมี API key ไม่ต้องต่อ network — linter เป็น **pure static analysis** ทั้งหมด
