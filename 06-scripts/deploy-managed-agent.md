---
title: deploy-managed-agent
parent: 06 Scripts
nav_order: 1
---

# deploy-managed-agent.sh — turn manifest → live managed agent

> *"Deploy a managed-agent template to `POST /v1/agents`."*
> — header comment ของ script

`scripts/deploy-managed-agent.sh` คือ **central deploy tool** ที่อ่าน `managed-agent-cookbooks/<slug>/agent.yaml` แล้วเปลี่ยนเป็น HTTP request ที่ส่งไปยัง Anthropic Managed Agents API ได้จริง

## จุดประสงค์

YAML manifest ของ cookbook ใช้ "convenience syntax" หลายแบบที่ API ไม่เข้าใจตรง ๆ:

| Convenience | API expects |
|---|---|
| `system: {file: prompt.md}` | `system: "<inlined string>"` |
| `skills: [{path: ../some-plugin/skills/x}]` | `skills: [{type: "custom", skill_id: "skill_xxx", version: "latest"}]` |
| `callable_agents: [{manifest: subagent.yaml}]` | `callable_agents: [{type: "agent", id: "agent_xxx", version: 1}]` |

`deploy-managed-agent.sh` ทำหน้าที่ **resolver** — แปลง convenience → fully-resolved payload — โดยมีลำดับดังนี้:

1. อ่าน `agent.yaml`, expand environment variables (`${VAR}`)
2. Inline `system.file` (read content) + apply `system.append`
3. **Recursive create_agent** — สร้าง subagent (leaf) ก่อน รวบ `agent_id` ของแต่ละ leaf
4. Upload `skills/` (zip + `POST /v1/skills`) → cache เพื่อไม่ upload ซ้ำ
5. POST orchestrator (root) สุดท้าย พร้อม `callable_agents: [{type:"agent", id, version}, ...]`

## CLI

```bash
scripts/deploy-managed-agent.sh <slug> [--dry-run]
```

| Argument | ความหมาย |
|---|---|
| `<slug>` | ชื่อโฟลเดอร์ใน `managed-agent-cookbooks/` เช่น `reg-monitor`, `renewal-watcher` |
| `--dry-run` | จำลอง deploy โดยไม่ส่ง HTTP จริง — print resolved JSON body ออกมาแทน |

## Environment variables

| Var | จำเป็นเมื่อ | ทำหน้าที่ |
|---|---|---|
| `ANTHROPIC_API_KEY` | ไม่ใช่ `--dry-run` | API key สำหรับเรียก `/v1/agents` + `/v1/skills` |
| `ANTHROPIC_API_BASE` | optional | override base URL (default `https://api.anthropic.com`) |
| `SKILL_TITLE_PREFIX` | optional | prefix string เพิ่มเข้า `display_title` ของ skill ที่ upload — **strictly validated** กับ allowlist regex |
| `DEPLOY_DEBUG` | optional | print orchestrator name + callable_agents ลง stderr เพื่อ debug |

## Algorithm — pseudocode ภาษาไทย

```text
function deploy(slug):
    ตรวจ ANTHROPIC_API_KEY (ยกเว้น dry-run)
    ตรวจ SKILL_TITLE_PREFIX กับ allowlist regex (กัน injection)
    เรียก create_agent("managed-agent-cookbooks/<slug>/agent.yaml")

function create_agent(file):
    json = resolve_manifest(file)              # expand env vars + skills.from_plugin
    json = inline_system(json, base)           # อ่าน system.file มา inline
    skills_json = []
    for ทุก skill path:
        skills_json += upload_skill(path)       # zip + POST /v1/skills
    json.skills = skills_json

    sub_ids = []
    for ทุก callable_agents[].manifest:
        out = create_agent(subagent.yaml)        # recursive! ลูก deploy ก่อน
        sub_ids += { type: "agent", id, version }
    json.callable_agents = sub_ids
    del json.output_schema                      # API ไม่รู้จัก field นี้

    ถ้า dry-run → append ลง $DRY_OUT
    มิฉะนั้น → POST /v1/agents → return (id, version)
```

ผลคือ orchestrator ถูก deploy **สุดท้ายเสมอ** — เพราะมันต้องอ้าง `agent_id` ของลูกที่ deploy ไปก่อนหน้านี้แล้ว

## Security-conscious detail: SKILL_TITLE_PREFIX validation

ส่วนหนึ่งของ script ที่สำคัญมาก คือการกรอง `SKILL_TITLE_PREFIX` ก่อนใช้ใน `curl -F`:

```bash
if [[ -n "${SKILL_TITLE_PREFIX:-}" ]]; then
  if ! [[ "$SKILL_TITLE_PREFIX" =~ ^[A-Za-z0-9._/:@\ -]+$ ]]; then
    echo "refusing SKILL_TITLE_PREFIX: value contains characters outside [A-Za-z0-9._/:@ -]" >&2
    exit 1
  fi
fi
```

**ทำไมต้องกรอง**: `SKILL_TITLE_PREFIX` ถูกส่งเข้า `curl -F "display_title=${SKILL_TITLE_PREFIX}..."` — ถ้าค่ามี newline หรือ `--`, attacker สามารถ "smuggle" multipart field เพิ่มเติมเข้าไปได้ เป็น classic **header/multipart injection** ที่ทุก deploy script ต้องระวัง

YAML env-var substitution ภายใน Python helper ก็มี allowlist เดียวกัน:

```python
SAFE = re.compile(r"^[A-Za-z0-9._/:@-]*$")
def sub(m):
    name = m.group(1)
    v = os.environ.get(name)
    if v is None:
        return m.group(0)
    if not SAFE.fullmatch(v):
        sys.exit(f"refusing ${{{name}}}: value contains characters outside [A-Za-z0-9._/:@-]")
    return v
```

จุดสำคัญคือ **allowlist (whitelist) ไม่ใช่ denylist** — เพราะ denylist ของ injection char มักโดน bypass ได้เสมอ

## Skill upload + cache

`upload_skill()` ทำงานดังนี้:

1. เช็คก่อนว่าเคย upload skill นี้ใน session นี้แล้วหรือยัง (อ่านจาก `$SKILL_CACHE_FILE` — temp file)
2. ถ้าเคยแล้ว → return cached JSON `{type:"custom", skill_id:..., version:"latest"}`
3. ถ้ายัง → `zip -qr` skill folder → `POST /v1/skills` (multipart) → parse `id` → cache → return

Cache สำคัญเพราะ orchestrator + subagents มักใช้ skill ตัวเดียวกันซ้ำ ๆ (เช่น `summary-style/`, `house-style/`)

## ตัวอย่างใช้งานจริง

### Deploy reg-monitor

```bash
export ANTHROPIC_API_KEY=sk-ant-...
cd /path/to/claude-for-legal
scripts/deploy-managed-agent.sh reg-monitor
```

**Output**:

```
deployed: reg-monitor
agent id: agent_abc123def456
console:  https://console.anthropic.com/agents/agent_abc123def456
```

### Dry-run เพื่อตรวจ resolved body ก่อน deploy

```bash
scripts/deploy-managed-agent.sh reg-monitor --dry-run | jq .
```

จะได้ JSON array — body แต่ละตัว = body ที่จะ POST ไป `/v1/agents` โดยเรียง **subagents ก่อน, orchestrator สุดท้าย**

นี่คือสิ่งที่ `test-cookbooks.sh` ใช้ในการ assert ว่า cookbook ปกติ

### Deploy พร้อม debug + prefix

```bash
DEPLOY_DEBUG=1 SKILL_TITLE_PREFIX="prod/" scripts/deploy-managed-agent.sh launch-radar
```

ทุก skill ที่ upload จะมี title `prod/launch-radar-skill`, และ stderr จะ print `{name, callable_agents}` ของ orchestrator

## กรณีใช้งานจริง

| สถานการณ์ | วิธีใช้ |
|---|---|
| First-time deploy ของ cookbook ใหม่ | `... --dry-run | jq .` → review → ลบ flag → run ใหม่ |
| Re-deploy หลังแก้ system prompt | run ตรง ๆ — script จะสร้าง agent ใหม่ (ไม่ใช่ update) |
| Deploy ลง staging env | `ANTHROPIC_API_BASE=https://staging.api.anthropic.com ...` |
| CI smoke test | `... --dry-run` (ไม่ต้องใช้ API key) — เห็นได้ใน `test-cookbooks.sh` |
| Multi-environment naming | `SKILL_TITLE_PREFIX=prod-` หรือ `staging-` |

## ข้อสังเกตจากการอ่าน source

1. **Output schemas leak protection** — บรรทัด `jq ... | del(.output_schema)` ในฟังก์ชัน `create_agent` คือการลบ field `output_schema` ก่อน POST เพราะ Managed Agents API ปัจจุบันยังไม่รองรับ structured output — schema เก็บไว้ใน YAML เพื่อให้ `validate.py` ใช้ runtime แทน

2. **Recursion depth = 1** — code อนุญาตให้ recurse `create_agent` ได้ลึกกว่า 1 ระดับ แต่ `test-cookbooks.sh` มี assertion `i<len(b)-1 and x.get('callable_agents')` ที่ reject ทันที = **policy บังคับ depth-1** (root + leaves, ไม่มี grandchildren)

3. **Trap-based cleanup** — `trap 'rm -f "$SKILL_CACHE_FILE"' EXIT` ลบ temp file ทุกกรณี (success/failure/Ctrl+C)

4. **`set -euo pipefail`** บรรทัดแรก — fail-fast บน undefined var, error, broken pipe ตามมาตรฐาน Bash strict mode

## Dependencies

- `bash` (4+)
- `curl`
- `jq`
- `python3` + `pyyaml`
- `zip`

ถ้า `jq` หรือ `pyyaml` ไม่มี — script จะตายตั้งแต่บรรทัดที่ 47-48 พร้อมข้อความ `requires jq` / `requires python3 + pyyaml`
